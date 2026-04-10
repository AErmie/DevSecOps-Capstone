# DevSecOps-Capstone

[Capstone project reference](https://devsecblueprint.com/learn/devsecops/capstone/index)

## Phase 1: Securing the DNA (SAST + SCA)

This repository implements Phase 1 gates in GitHub Actions to block vulnerable code and dependencies before merge.

### Security Gates

- SAST: Semgrep runs with `p/security-audit` and `p/python` rule packs.
- SAST: CodeQL runs as a dedicated workflow and enforces a high-severity alert gate.
- SCA: Trivy scans third-party libraries and fails on high/critical dependency vulnerabilities.

### Triggers

- Pull requests: runs through `.github/workflows/pr.yml`.
- Push to `main`: runs through `.github/workflows/main.yml`.
- CodeQL SAST: runs through `.github/workflows/codeql.yml` on pull requests and pushes.

### Blocking Policy

- Semgrep fails the build on `ERROR` findings.
- CodeQL fails the build when open high/critical alerts are detected for the current ref.
- Trivy fails the build on `HIGH` and `CRITICAL` vulnerabilities.

### Milestone Outcome

Any pull request or push containing high-severity security risk fails CI and must be remediated before proceeding.

## Local Validation

Use these commands before opening a pull request:

```bash
python -m pip install -r requirements.txt
python -m pytest tests/
python -m pylint .
python -m black --check .
# Run twice: first pass may auto-fix files, second confirms a clean state.
pre-commit run --all-files
pre-commit run --all-files
```

## Repository Hygiene Files

- `.pre-commit-config.yaml`: local quality/security hooks for markdown, YAML, Python, and repository safety checks.
- `.editorconfig`: editor defaults for encoding, line endings, and trailing whitespace behavior.
- `.gitattributes`: consistent line-ending behavior by file type.
- `.markdownlint.yaml`: markdown linting rules used by pre-commit.
- `.yamllint.yaml`: YAML linting policy used by pre-commit.
