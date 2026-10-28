# Publishing a release (maintainer)

The official DMG is built and checked by the maintainer on their own Mac and uploaded by hand. GitHub Actions (`.github/workflows/`) builds the same tag a second time for comparison and never publishes anything. DMGs are never committed: `.gitignore` excludes them.

## Versioned release
1. Set `CFBundleShortVersionString` (e.g. `2.4` or `2.4 Beta 1`) and `CFBundleVersion` in `project.yml`, and add the section to `CHANGELOG.md` (heading `## 2.4 Beta 1 — build 40`).
2. Optional: write curated notes in `docs/RELEASE-NOTES-<version>.md` (e.g. `docs/RELEASE-NOTES-2.4.md`, `docs/RELEASE-NOTES-2.4-beta1.md`).
3. `brew install libmtp libusb xcodegen` (once), then `Scripts/build-release.sh` creates `build/MusicManager-<version>.dmg` and its `.sha256`.
4. On GitHub: Releases ▸ Draft a new release ▸ tag (e.g. `v2.4` for the final release, `v2.4-beta1` for a beta), tick "pre-release" only for betas, attach the DMG and the `.sha256`, paste the release notes and the text of `docs/INSTALL.md`. Betas are tagged on the `beta` branch, finals on `main`.

## CI build of a tag
Pushing a `v*` tag, including one created by publishing a release on GitHub, runs the **Release** workflow:
1. It checks the tag matches `CFBundleShortVersionString` in `project.yml` on the tagged commit, and fails otherwise.
2. It runs `Scripts/build-release.sh` on a GitHub macOS runner.
3. It creates a **draft** release named `Music Manager v<version> (CI build)` with the DMG, its `.sha256`, the release notes (from `docs/RELEASE-NOTES-<version>.md`, else the `CHANGELOG.md` section), the bundled libmtp/libusb versions and the changeset since the previous tag.

The draft is visible only to people with write access. It is never published or marked *Latest*, and the workflow never changes any other release, so the DMG uploaded by hand is never replaced or duplicated. Re-running the workflow replaces only its own draft. Delete the draft once the comparison is done.

The tagged commit must contain `.github/workflows/`, because GitHub runs the workflows of the tagged commit: merge `main` into `beta` before tagging a beta. To rebuild an existing tag: Actions ▸ Release ▸ Run workflow ▸ tag.

The two DMGs will not have the same SHA-256 (ad hoc signatures and disk image timestamps differ). Compare their contents instead, e.g. mount both and run `diff -r` on the apps, or compare `vtool -show-build` and `otool -L` of the binary and the bundled libraries.

## Pull requests
Every pull request that touches the app (`MusicManager/`, `Scripts/`, `project.yml`) builds the DMG. It is attached to the run page for 7 days and nothing is published.

## Requirements
- `Scripts/` must be committed: CI builds with `Scripts/build-release.sh`, and `project.yml` runs `Scripts/bundle-libs.sh` as a build phase.
- `MMFeedbackURL` in `project.yml` points to `https://github.com/RickyDive/musicmanager/issues/new/choose`; Help ▸ Send Feedback opens that page.
- The repository must contain the source (LGPL requirement for the bundled libraries; the links to the sources are in `THIRD-PARTY-LICENSES.txt`).
- License: GPL v3.0 (`LICENSE` file); bundled libraries LGPL 2.1+ (`licenses/` folder).
- Optional repository variables: `RUNNER_MACOS` (JSON runner labels instead of `macos-26`), `XCODE_VERSION` (e.g. `26.1`).

## Bundled libraries and minimum macOS version
Homebrew libraries (libmtp, libusb) are built for the macOS version Homebrew runs on: on macOS 27 they declare 27.0 as minimum and Xcode warns ("Building for macOS-26.0, but linking with dylib … built for newer version 27.0"). If they stayed that way in the package, the app would **not start on macOS 26**. `Scripts/bundle-libs.sh` therefore lowers the minimum version of the bundled libraries to the app's (`vtool`), and `Scripts/verify-bundle.sh` fails if a library requires more than macOS 26.0. Manual check: `vtool -show-build "Music Manager.app/Contents/Frameworks/libmtp.9.dylib" | grep minos`.
