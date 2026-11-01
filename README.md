# Neus CI

Public GitHub Actions runner repository for the private `zenderock/Neus-Platform`
source (useinbox.email).

This repository holds **only CI workflows**. `.github/workflows/gate.yml` takes
a branch, tag or exact commit of the source. It checks
the source out with a read-only token, runs the source's
`scripts/ci/public-gate.sh` (backend, SDK and MCP server, landing, V4 runtime
CSS), and reports `neus-ci/gate` back to that source commit, so the pull request
shows the result. It is dispatched by hand, or by `watch-main.yml`.

`.github/workflows/watch-main.yml` gives `main` its automatic run. Actions cannot
start on the private source, so it polls: every five minutes it reads the head
of `main` and, if that commit has no `neus-ci/gate` status and no gate run was
dispatched for it, dispatches `gate.yml` on that exact SHA. A push to `main` is
checked within about five to fifteen minutes (GitHub delays scheduled runs).
Other branches and pull requests are still dispatched by hand. GitHub disables
a schedule after 60 days without a commit in this repository; re-enable it with
`gh workflow enable watch-main.yml -R releaseszenderock/neus-ci`.

```bash
gh workflow run gate.yml -R releaseszenderock/neus-ci -f SOURCE_REF=main
gh workflow run gate.yml -R releaseszenderock/neus-ci -f SOURCE_REF=my-branch -f CHECKS=backend,sdk -f LABEL="PR #491"
```

## Required repository secret

- `SOURCE_REPO_TOKEN`: a fine-grained token limited to `zenderock/Neus-Platform`,
  with **Contents: Read** and **Commit statuses: Read and write**.

The canonical copy of this workflow and the architecture notes live in the
private source under `infra/github/neus-ci/` and `docs/ci.md`.
