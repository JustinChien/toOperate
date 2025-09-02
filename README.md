This repository demonstrates integrating secret scanning (Gitleaks) and dependency review into a project
using both GitHub Actions and local `pre-commit` hooks.

Purpose
-------

This project shows a minimal configuration to:

- Run Gitleaks as a pre-commit hook to catch secrets before they are committed.
- Run a full Gitleaks scan in GitHub Actions on push/PR and upload the SARIF report.
- Run GitHub's Dependency Review action on pull requests to detect high-severity vulnerabilities.

What is included
----------------

- `src/` - example source files (any secrets have been removed).
- `.pre-commit-config.yaml` - pre-commit hooks (includes Gitleaks, formatters, checks).
- `.github/workflows/security.yaml` - CI workflow running Gitleaks and Dependency Review.

Quickstart (local)
------------------

1. Install dependencies:

   - Python projects: `pip install pre-commit`
2. Install and enable git hooks:

   ```
   pre-commit install
   pre-commit run --all-files
   ```
3. To test Gitleaks locally via pre-commit, make a change that would trigger the hook (see Test Plan).

GitHub Actions
--------------

- On every push and pull request the workflow runs a full Gitleaks scan and (on PRs) the Dependency Review.
- Gitleaks is installed in the workflow by downloading a specific release binary (verify/ pin versions as needed).

Security notes
--------------

- Never commit secrets. Use environment variables or a secrets manager for credentials.
- This repo removed the example hard-coded secrets previously present in `src/`.
- Review pinned versions in `.pre-commit-config.yaml` and `security.yaml` to ensure they meet your org policies.

Test plan (basic)
-----------------

1. Pre-commit / local Gitleaks test:

   - Create a new file containing a fake secret (e.g. `MY_FAKE_TOKEN=abcd1234`) and try to commit it. The pre-commit `gitleaks` hook should block the commit.
2. Workflow Gitleaks test:

   - Push a branch with a commit that introduces a fake secret (do this only in a test repo/branch).
   - The `Security Checks` workflow should run and produce `gitleaks-results.sarif` as an artifact. The workflow will fail if gitleaks exits with code indicating secrets found.
3. Dependency Review test:

   - Open a pull request that adds/modifies a dependency file (e.g., `requirements.txt`, `package.json`) that includes a known high severity dependency (or simulate by referencing a vulnerable version).
   - The Dependency Review step will comment on the PR and can be configured to fail on `high` severity.

How to contribute / customize
-----------------------------

- To change Gitleaks versions or arguments, edit `.pre-commit-config.yaml` (for local checks) and `.github/workflows/security.yaml` (for CI).
- Consider pinning versions consistently between local hooks and CI to avoid mismatches.

Pinned versions and Windows compatibility
-----------------------------------------

- **CI gitleaks version**: `v8.24.3` (installed in `.github/workflows/security.yaml` via `GITLEAKS_VERSION`).
- **Local pre-commit gitleaks rev**: `.pre-commit-config.yaml` currently references `rev: v8.24.3`.

Note: some `gitleaks` releases or their pre-commit wrappers can fail on native Windows shells (the hook may treat the `detect` argument as a filename). If you encounter this on Windows, a known-working local `rev` in some environments has been `v8.18.4` — pin whatever version works for your environment and **align CI and local revs** to avoid surprises.

Recommendation: either:

- Use WSL or Git Bash for running `pre-commit` on Windows (preferred), or
- Pin the local pre-commit `rev` to the Windows-compatible release and update the CI `GITLEAKS_VERSION` to match, or
- Replace the manual CI binary download with the official Gitleaks GitHub Action to simplify and harden installation.

