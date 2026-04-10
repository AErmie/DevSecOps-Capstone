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

### Phase 1 Status

- Implemented: SAST (Semgrep + CodeQL) and SCA (Trivy library scan) gates are active in GitHub Actions.
- Implemented: High/Critical policy is enforced as a hard fail in CI.
- Current validation behavior: this intentionally vulnerable baseline triggers
  Trivy findings (for example `h11`, `starlette`, `black`) at HIGH/CRITICAL
  severity, so the PR security gate fails as expected until vulnerabilities are
  remediated.

## Local Validation

Use these commands before opening a pull request:

```bash
python -m pip install -r requirements.txt
python -m pip install pre-commit
python -m pytest tests/
python -m pylint .
python -m black --check .
pre-commit run --all-files
# Re-run the same command once more to confirm a clean post-fix state.
```

## Repository Hygiene Files

- `.pre-commit-config.yaml`: local quality/security hooks for markdown, YAML, Python, and repository safety checks.
- `.editorconfig`: editor defaults for encoding, line endings, and trailing whitespace behavior.
- `.gitattributes`: consistent line-ending behavior by file type.
- `.markdownlint.yaml`: markdown linting rules used by pre-commit.
- `.yamllint.yaml`: YAML linting policy used by pre-commit.
