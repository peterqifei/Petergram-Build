# Petergram Build

Build infrastructure for [Petergram](https://gitlab.com/my-denia-group/Petergram) (Windows desktop / future macOS) and [Petergram Android](https://gitlab.com/my-denia-group/Petergram-Android).

**GitLab owns the source, merge requests, releases, and source locks.** This GitHub repository holds only CI workflows. Do not push generated vendor trees, patches, credentials, or Petergram source copies here.

## Build policy

- **Manual only**: workflows use `workflow_dispatch`. No build on push, pull request, tag, schedule, or GitLab merge.
- **Batch changes**: build after substantial feature batches; exceptional builds are for significant regressions.
- **Apple Silicon only**: eventual macOS builds target `arm64`, not Intel/universal; iOS uses arm64 for devices.
- **Public CI is not private**: build logs and uploaded artifacts are publicly accessible. Never provide real Telegram API credentials, persistent signing keys, account sessions, or private binaries to a public workflow.
- **Source provenance**: each run records the resolved GitLab commit, its upstream/source lock and artifact hashes. GitLab remains the release authority.
- **No release signing in CI**: public CI creates disposable development builds only. Shipping a signed installer/APK/IPA requires a separate controlled signing flow.

## Current support

| Platform | GitHub-hosted runner | What is actually wired |
| --- | --- | --- |
| Android arm64-v8a | `ubuntu-24.04` | Manual attempt to reconstruct the locked Nagram tree and build a disposable debug APK, followed by artifact checks |
| Windows x64 | `windows-2025` | Existing verified GitLab build remains authoritative; a full tdesktop/Qt/MSVC dependency build is **not** yet portable to a standard 14 GB SSD runner |
| macOS arm64 | `macos-15` | No Petergram macOS port/build entry point exists yet; runtime alone is not a product build |
| iOS arm64 | `macos-15` | No Petergram iOS source/build entry point exists yet; cannot emit an authentic Petergram IPA |

GitHub standard runners are free for public repositories, **not** unlimited in CPU/RAM/disk. A job may still fail from storage, memory or timeout pressure. Slow builds are acceptable; impossible builds should fail with useful diagnostics rather than pretending they passed.

## Android manual build

In **Actions → Android arm64 debug → Run workflow**, select a GitLab branch, tag, or exact commit. The job fetches that GitLab revision, reconstructs its pinned Nagram base, installs its pinned Android toolchain, generates a one-off debug keystore with placeholder Telegram API credentials, and attempts the project's existing `scripts/build.sh` pipeline.

**Do not install the CI debug APK over the daily-use Android app.** The one-off signing key differs from the local project key. Android will reject the in-place update; a CI debug APK is only an unsigned-official-release development candidate, not a daily release or store package.

GitHub artifacts expire after 7 days. No automatic GitLab releases, sign-in, deployment, or changes to the source repos are performed.

## Why not run every platform now?

The existing Windows project explicitly reconstructs a pinned tdesktop source tree into an untracked `vendor/`, with a Windows-specific overlay, full `prepare-libraries.ps1`, and an MSVC build. The Android project uses its own pinned base and signing separation. Neither workflow can honestly be replaced by an arbitrary `git clone && cmake` or `gradlew` command.

Apple CI jobs will be added when they can compile **Petergram** sources, not just upstream Telegram or an empty Xcode template.
