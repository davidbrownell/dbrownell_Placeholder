---
type: Change
title: Update supported Python versions to 3.11 - 3.14
description: Dropped Python 3.10, added Python 3.14, and updated the copier-UvScaffolding template from v0.5.11 to v0.5.14.
tags:
  - python-versions
  - ci
  - copier
  - release
resource: https://github.com/davidbrownell/dbrownell_Placeholder/tree/PythonVersions
sources:
  - resource: https://github.com/gt-csse/copier-UvScaffolding
    title: copier-UvScaffolding template (v0.5.11 -> v0.5.14)
  - resource: https://peps.python.org/pep-0619/
    title: PEP 619 - Python 3.10 Release Schedule
  - resource: https://peps.python.org/pep-0745/
    title: PEP 745 - Python 3.14 Release Schedule
generated:
  by: Claude Code (claude-opus-5-5)
  at: 2026-10-24T14:56:59Z
status: stable
---

# Update supported Python versions to 3.11 - 3.14

## Summary

The package now supports Python 3.11, 3.12, 3.13, and 3.14. Python 3.10 is no longer supported. The project was also regenerated from version v0.5.14 of the [copier-UvScaffolding](https://github.com/gt-csse/copier-UvScaffolding) template, which hardens the CI/CD workflows and the release action.

Because dropping a Python version changes the package's installation requirements, this change bumps the minor version.

## What Changed

### Supported Python versions

| File | Change |
|------|--------|
| `pyproject.toml` | `requires-python` changed from `>= 3.10` to `>= 3.11`. Removed the `Python :: 3.10` classifier and added `Python :: 3.14`. |
| `.github/workflows/CICD.yml` | The test matrix changed from `3.13, 3.12, 3.11, 3.10` to `3.14, 3.13, 3.12, 3.11`. `build_python_version` changed from `3.10` to `3.11`. |
| `.copier-answers.yml` | `python_versions` changed to `3.11, 3.12, 3.13, 3.14`. |
| `uv.lock` | Regenerated for the new `requires-python`. Removed `exceptiongroup`, which is only needed on Python < 3.11. |

### Template update (v0.5.11 -> v0.5.14)

- **Workflow concurrency**: `CICD.yml` defines a concurrency group. A new pull request run cancels the superseded run. Each push gets its own group so that a release is never silently skipped.
- **Job timeouts**: Every job in `CICD_impl.yml` sets `timeout-minutes`.
- **Renamed values**:
  - The `python_package_version` input is now `build_python_version`.
  - The `package_coverage` job is now `postprocess_coverage`.
  - The `package_name` output is now `wheel_filename`.
- **Wheel validation**: The build fails unless exactly one wheel is produced.
- **Release action (`.github/actions/release_impl`)**:
  - Inputs (`wheel_filename`, `package_version`, `coverage_total`) are passed explicitly. The action no longer reads implicit job-level environment variables.
  - Secrets and inputs are passed to scripts through `env:`. They are no longer interpolated directly into shell scripts, which prevents script injection.
  - The minisign private key is written to `RUNNER_TEMP` and removed by an `EXIT` trap, even when signing fails.
  - The new `retry.sh` retries the tag push and the GitHub release creation up to 3 times.
  - Creating the GitHub release is idempotent. A rerun uploads to the existing release.
  - The coverage badge is skipped when no coverage total is available.
- **Removed redundant steps**: Deleted the `uv sync --frozen` steps that `uv run` makes unnecessary.
- **Documentation**:
  - `DEVELOPMENT.md` documents the `+minor`/`+major` commit-message conventions for semantic versioning and the updated signing command.
  - `README.md` corrects the signature file extension (`.minisign` to `.minisig`).
  - `src/AutoGitSemVer.yaml` and `.github/release_sources.yaml` now note that the two files must be kept in sync.
- **Minor tooling**: Reformatted the `ruff` ignore list. Changed the pytest verbosity flags to `-vvv`. Added a `noqa: CPY001` to the sample test file.

## Why

- **Python 3.10 reaches end of life in October 2026** (PEP 619). After that date it receives no security fixes, so continuing to test against it and advertise support for it no longer has value.
- **Python 3.14 was released in October 2025** (PEP 745) and should be covered by CI and the package classifiers.
- **Template update**: The template update brings in reliability and security fixes for the release pipeline:
  - Transient network failures no longer fail a release.
  - Secrets are no longer exposed to script injection or left in shared temp directories.
  - Partially failed releases can be rerun.

## Impact

- Users on Python 3.10 can no longer install new releases. Earlier releases are still available to them.
- Consumers of the reusable `CICD_impl.yml` workflow must rename the `python_package_version` input to `build_python_version`.
