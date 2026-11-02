# Neus CI

Public GitHub Actions runner repository for the private `zenderock/Neus-Platform`
source (useinbox.email).

This repository holds **only CI workflows**. `.github/workflows/gate.yml` is
dispatched by hand with a branch, tag or exact commit of the source. It checks
the source out with a read-only token, runs the source's
`scripts/ci/public-gate.sh` (backend, SDK and MCP server, landing, V4 runtime
CSS), and reports `neus-ci/gate` back to that source commit, so the pull request
shows the result. Nothing here runs on pushes or pull requests.

```bash
gh workflow run gate.yml -R releaseszenderock/neus-ci -f SOURCE_REF=main
gh workflow run gate.yml -R releaseszenderock/neus-ci -f SOURCE_REF=my-branch -f CHECKS=backend,sdk -f LABEL="PR #491"
```

## Desktop app (InboxMail)

`.github/workflows/desktop.yml` builds the desktop app (`desktop/` in the
source) from a branch, tag or commit: unit tests and typecheck, then one
package per channel (Windows NSIS and MSIX, Linux `.deb`, snap, Flatpak). The
NSIS package is installed on the runner and the notch is tried for real: a
sample code drops (`INBOXMAIL_NOTCH_SMOKE=10`), the main window is minimized, a
text field is clicked, Fill is clicked and the code must reach that field; the
screenshots go to a `ci-shots/<run>` branch. It reports `neus-ci/desktop` back to the source
commit. Nothing is signed; packages are kept three days.

With `RELEASE_TAG` (for example `v1.0.0`, equal to the app's version), the run
also publishes the direct downloads as a GitHub Release of neus-ci:
`InboxMail-Setup.exe`, `inbox-mail_amd64.deb`, `SHA256SUMS.txt` and
`latest.json` (what the installed apps check for updates). The names
do not change from one version to the next, so
`https://github.com/releaseszenderock/neus-ci/releases/latest/download/InboxMail-Setup.exe`
always serves the newest installer. The job refuses a tag that does not match
the version of both files; it needs `windows-direct` and `linux-direct`.

```bash
gh workflow run desktop.yml -R releaseszenderock/neus-ci -f SOURCE_REF=v5
gh workflow run desktop.yml -R releaseszenderock/neus-ci -f SOURCE_REF=my-branch -f CHANNELS=windows-direct,linux-direct -f LABEL="PR #510"
gh workflow run desktop.yml -R releaseszenderock/neus-ci -f SOURCE_REF=v5 -f CHANNELS=windows-direct,linux-direct -f RELEASE_TAG=v1.0.0
```

## Required repository secret

- `SOURCE_REPO_TOKEN`: a fine-grained token limited to `zenderock/Neus-Platform`,
  with **Contents: Read** and **Commit statuses: Read and write**.
- `TAURI_SIGNING_PRIVATE_KEY` and `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`: the
  updater key of InboxMail (its public half is in `desktop/src-tauri/tauri.conf.json`).
  The direct builds (`windows-direct`, `linux-direct`) sign their packages with it,
  and a release ships `latest.json`, which the installed apps read to update
  themselves. Without the key those two builds fail.

The canonical copy of this workflow and the architecture notes live in the
private source under `infra/github/neus-ci/` and `docs/ci.md`.
