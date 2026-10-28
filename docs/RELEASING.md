# Publishing a release (maintainer)

Releases are built and published by GitHub Actions (`.github/workflows/`). DMGs are never committed: `.gitignore` excludes them.

## Pull requests
Every pull request that touches the app (`MusicManager/`, `Scripts/`, `project.yml`) builds the DMG. It is attached to the run page for 7 days and nothing is published.

## Versioned release
1. Set `CFBundleShortVersionString` (e.g. `2.4` or `2.4 Beta 1`) and `CFBundleVersion` in `project.yml`, and add the section to `CHANGELOG.md` (heading `## 2.4 Beta 1 — build 40`).
2. Optional: write curated notes in `docs/RELEASE-NOTES-<version>.md` (e.g. `docs/RELEASE-NOTES-2.4.md`, `docs/RELEASE-NOTES-2.4-beta1.md`); without it the `CHANGELOG.md` section is used.
3. Tag the commit whose `project.yml` has that version and push the tag: `git tag v2.4-beta1 && git push origin v2.4-beta1` (final: `v2.4`). Betas are tagged on the `beta` branch, finals on `main`; the branch must contain `.github/workflows/`, because GitHub runs the workflows of the tagged commit. Tags containing `alpha`, `beta` or `rc` become pre-releases; others become *Latest*.
4. The **Release** workflow checks the tag matches `project.yml`, runs `Scripts/build-release.sh` to build `MusicManager-<version>.dmg` and its `.sha256`, and publishes the release with the notes, checksum, bundled libmtp/libusb versions, `docs/INSTALL.md` and the changeset since the previous tag.

To rebuild an existing tag: Actions ▸ Release ▸ Run workflow ▸ tag. If the release already exists, only the DMG and `.sha256` are replaced and its notes are kept.

## Requirements
- `Scripts/` must be committed: CI builds with `Scripts/build-release.sh`, and `project.yml` runs `Scripts/bundle-libs.sh` as a build phase.
- `MMFeedbackURL` in `project.yml` points to `https://github.com/RickyDive/musicmanager/issues/new/choose`; Help ▸ Send Feedback opens that page.
- The repository must contain the source (LGPL requirement for the bundled libraries; the links to the sources are in `THIRD-PARTY-LICENSES.txt`).
- License: GPL v3.0 (`LICENSE` file); bundled libraries LGPL 2.1+ (`licenses/` folder).
- Optional repository variables: `RUNNER_MACOS` (JSON runner labels instead of `macos-26`), `XCODE_VERSION` (e.g. `26.1`).

## Building locally
`brew install libmtp libusb xcodegen`, then `Scripts/build-release.sh` creates `build/MusicManager-<version>.dmg` and its `.sha256`.

## Bundled libraries and minimum macOS version
Homebrew libraries (libmtp, libusb) are built for the macOS version Homebrew runs on: on macOS 27 they declare 27.0 as minimum and Xcode warns ("Building for macOS-26.0, but linking with dylib … built for newer version 27.0"). If they stayed that way in the package, the app would **not start on macOS 26**. `Scripts/bundle-libs.sh` therefore lowers the minimum version of the bundled libraries to the app's (`vtool`), and `Scripts/verify-bundle.sh` fails if a library requires more than macOS 26.0. Manual check: `vtool -show-build "Music Manager.app/Contents/Frameworks/libmtp.9.dylib" | grep minos`.
