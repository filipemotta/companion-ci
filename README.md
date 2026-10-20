# companion-ci

Reusable GitHub Actions workflow for the companion repositories of my articles
on Medium (the Istio, Hands-On series and others).

- **Every push and pull request:** `validate.sh` from the repository root
  (`bash -n`, shellcheck, kubeconform). A validator that is missing fails the job.
- **Push to the default branch, manual runs and the weekly schedule:** the whole
  lab end to end in a kind cluster on the runner. `out/` is kept as an artifact
  for 14 days, and the teardown runs even when the lab fails.

## Use

`.github/workflows/lab.yml` in the companion repository:

```yaml
name: lab

on:
  push:
  pull_request:
  workflow_dispatch:
  schedule:
    - cron: "0 6 * * 1"

permissions:
  contents: read

jobs:
  lab:
    uses: filipemotta/companion-ci/.github/workflows/companion.yml@<commit sha>
    with:
      lab: bash scripts/test-all.sh
```

Inputs, all optional:

- `validate` (boolean, default `true`): run `validate.sh`.
- `lab` (default `bash scripts/test-all.sh`): command that runs the lab and exits
  non-zero on failure. An empty string skips the lab.
- `teardown` (default `bash scripts/teardown.sh`): runs after the lab, always.
- `lab-timeout-minutes` (default `90`).
- `kind-version` (default `v0.33.0`) and `kubeconform-version` (default `v0.8.0`),
  both downloaded from their GitHub releases and checked against the published
  SHA-256.

Pin the call to a commit SHA, not to `main`.

The lab runs on `ubuntu-24.04` (4 CPUs, 16 GB RAM on public repositories). Steps
that reach real sites on the Internet, or pull from Docker Hub without
credentials, can fail for reasons outside the repository; the step logs in `out/`
say which one.
