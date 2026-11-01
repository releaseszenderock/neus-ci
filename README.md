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

## Required repository secret

- `SOURCE_REPO_TOKEN`: a fine-grained token limited to `zenderock/Neus-Platform`,
  with **Contents: Read** and **Commit statuses: Read and write**.

The canonical copy of this workflow and the architecture notes live in the
private source under `infra/github/neus-ci/` and `docs/ci.md`.
