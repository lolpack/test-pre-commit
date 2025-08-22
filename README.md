# pyrefly-pre-commit

A **pre-commit** hook for [Pyrefly](https://github.com/facebook/pyrefly).

This repository provides three hooks:

- `pyrefly-typecheck` (installs Pyrefly from PyPI at the repo tag's version)
- `pyrefly-typecheck-system` (uses an already-installed `pyrefly` on your PATH)
- `pyrefly-typecheck-flex` (installs a specific version of Pyrefly)

## Usage

Add one of the following to your project's `.pre-commit-config.yaml`:

**Option A: managed install (recommended)** — pins the PyPI version so all contributors run the same Pyrefly:

```yaml
repos:
  - repo: https://github.com/facebook/pyrefly-pre-commit
    # Tag of this repo; keep it in sync with the Pyrefly version you want.
    rev: v0.29.1
    hooks:
      - id: pyrefly-typecheck
        # Optional extra args, e.g. select a config or enable --strict
        # args: [ "--strict" ]
```

**Option B: system install** — uses an already-installed `pyrefly` (for monorepos or custom builds):

```yaml
repos:
  - repo: https://github.com/facebook/pyrefly-pre-commit
    rev: v0.29.1
    hooks:
      - id: pyrefly-typecheck-system
```

Then install and run:

```bash
pip install pre-commit  # or: uvx pre-commit
pre-commit install
pre-commit run --all-files
```

## Why two hooks?

Some users prefer fully reproducible hooks that **pin** tool versions (via PyPI). Others need to use a **system** build (e.g., nightlies or a local wheel). This repository supports both.

## Behavior and defaults

- We run `pyrefly check` at the repo root and **ignore filenames from pre-commit** (`pass_filenames: false`), since Pyrefly checks project state rather than individual files.
- The hook targets `stages: [pre-commit, pre-merge-commit, pre-push, manual]` so you can run it locally and in CI.
- You can **skip temporarily** with `SKIP=pyrefly-typecheck git commit -m "..."`.
- Add `args` to pass flags to Pyrefly, e.g. `["--strict"]` or a custom `--config` path.
- If your config lives in a non-root subtree, set `args: ["--project-root=path/to/subdir"]`.

## CI example

Use the official `pre-commit/action` to run the hook in GitHub Actions:

```yaml
name: pre-commit
on:
  pull_request:
  push:
    branches: [ main ]
jobs:
  pre-commit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - uses: pre-commit/action@v3.0.1
        with:
          extra_args: --all-files
```

## Versioning

Tags in this repository are expected to match the PyPI version of `pyrefly` provided by the managed-install hook. For example, tag `v0.29.1` pins `pyrefly==0.29.1`.

## License

MIT

## Choosing your Pyrefly version

Different teams have different needs for controlling the version of Pyrefly. This repo supports **three patterns**:

### 1. Pinned hook (default)
The `pyrefly-typecheck` hook pins Pyrefly to the version declared in this repo’s tag (e.g., `v0.29.1` → `pyrefly==0.29.1`).  
This ensures all contributors run exactly the same version.

```yaml
repos:
  - repo: https://github.com/facebook/pyrefly-pre-commit
    rev: v0.29.2
    hooks:
      - id: pyrefly-typecheck
```

### 2. User-selected version (flex)
If you want to pick your own Pyrefly version, use the **flexible hook** (un-pinned) and declare `additional_dependencies` yourself:

```yaml
repos:
  - repo: https://github.com/facebook/pyrefly-pre-commit
    rev: v0.1.0   # tag of this repo, not the Pyrefly version
    hooks:
      - id: pyrefly-typecheck-flex
        additional_dependencies: ["pyrefly==0.29.2"]
```

This avoids conflicts with a pinned manifest and gives you full control.

### 3. System install
For nightlies, local wheels, or custom builds, use the **system hook** and install Pyrefly yourself (e.g., `pipx install pyrefly==0.30.0`).

```yaml
repos:
  - repo: https://github.com/facebook/pyrefly-pre-commit
    rev: v0.1.0
    hooks:
      - id: pyrefly-typecheck-system
```

---

**Recommendation**:  
- Use **pinned** for reproducibility.  
- Use **flex** if you want to control Pyrefly upgrades yourself.  
- Use **system** if you need a local/custom build.
