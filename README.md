# ci-templates

Shared dev-tooling config (pre-commit hooks, Makefile, version-bump/tag
scripts, and tool configs for bandit/ty/mypy) for POS repos. Edit a file
here, push to `main`, and a GitHub Action opens a sync PR in every repo
listed in [`.github/sync.yml`](.github/sync.yml).

`ty.toml`, `bandit.toml`, and `mypy.ini` are standalone config files (not
inside `pyproject.toml`) specifically so they *can* be synced — each
target repo's `pyproject.toml` holds per-project metadata (name, version,
deps) that must stay project-specific and is never synced.

## One-time setup

1. **Create a GitHub PAT** with write access to every target repo:
   - GitHub → Settings → Developer settings → Personal access tokens →
     Fine-grained tokens → New token
   - Resource owner: your account
   - Repository access: select the specific POS repos this should sync into
     (you can add more later by re-issuing or widening the token)
   - Permissions: **Contents: Read and write**, **Pull requests: Read and write**
2. **Add the PAT as a secret in this repo**: Settings → Secrets and variables →
   Actions → New repository secret → name it `GH_PAT`.
3. **List your repos** in [`.github/sync.yml`](.github/sync.yml) under `repos:`
   (one `owner/repo` per line).

## Day to day

Edit `.pre-commit-config.yaml`, `Makefile`, or anything under `scripts/`, push
to `main`. The `sync` workflow opens a PR titled with the `chore(sync):`
prefix in each target repo — review the diff and merge it there. Nothing is
force-pushed or auto-merged.

## Adding a new POS repo to the sync

1. Add `owner/repo` to the `repos:` block in `.github/sync.yml`.
2. Make sure the PAT in `GH_PAT` has access to that repo (fine-grained tokens
   need to be re-scoped to include it).
3. Push — the next sync run will open the initial PR there too.

## Checks on this repo itself

This repo isn't a `uv` project (no Python deps to lock/sync), so
`.pre-commit-config.yaml` here is **payload only** — don't run it directly
against this repo, it's meant to be copied into target repos as-is.

[`.pre-commit-config.local.yaml`](.pre-commit-config.local.yaml) is a separate,
minimal config (currently just gitleaks) that actually runs against this
repo's own content, since it holds a PAT-bearing workflow and shouldn't leak
secrets either. Already installed via:

```
uv run pre-commit install -c .pre-commit-config.local.yaml
```

(or `uvx pre-commit install -c .pre-commit-config.local.yaml` if you don't
want a local venv here at all).
