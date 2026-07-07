# PXView-build

Public build runner for the private `haikumuse/PXView` repository.
GitHub Actions on public repos are free and unlimited, so this repo runs
the heavy build jobs (Linux AppImage + macOS DMG) on behalf of the private
repo, which only spends a few seconds on a curl dispatch when auto-trigger
is enabled.

## Setup (one-time)

1. Create a fine-grained PAT with `Contents: Read-only` access to `haikumuse/PXView`.
2. Add it as a repository secret named `PXVIEW_PAT` in this repo
   (Settings → Secrets and variables → Actions → New repository secret).

## Manual trigger

Actions → "PXView Cross-Repo Build" → Run workflow:

- `ref`: branch / tag / SHA to build (e.g. `view_and_data`, `main`, `abc123`)
- `targets`: `linux` / `mac` / `both`

Artifacts are published under the workflow run.

## Auto trigger (optional)

In the private `PXView` repo, add `.github/workflows/trigger-public-build.yml`:

```yaml
name: Trigger public build
on:
  push:
    branches: [ view_and_data, main ]
jobs:
  trigger:
    runs-on: ubuntu-latest
    steps:
      - name: Repository dispatch
        env:
          GH_TOKEN: ${{ secrets.PXVIEW_BUILD_PAT }}
        run: |
          curl -X POST \
            -H "Accept: application/vnd.github+json" \
            -H "Authorization: Bearer $GH_TOKEN" \
            https://api.github.com/repos/haikumuse/PXView-build/dispatches \
            -d '{"event_type":"build","client_payload":{"ref":"${{ github.ref_name }}"}}'
```

Use a separate PAT (or the same one with `Contents: Read-only` on PXView-build)
stored as `PXVIEW_BUILD_PAT` in the private repo. Each push to the private
repo then spends ~2 seconds of private Actions quota to dispatch the build
to this public repo.
