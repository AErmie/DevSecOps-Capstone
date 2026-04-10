# DevSecOps-Capstone

[Capstone project reference](https://devsecblueprint.com/learn/devsecops/capstone/index)

## Phase 1: Securing the DNA (SAST + SCA)

This repository implements Phase 1 gates in GitHub Actions to block vulnerable code and dependencies before merge.

### Security Gates

- SAST: Semgrep runs with `p/security-audit` and `p/python` rule packs.
- SAST: CodeQL runs as a dedicated workflow and enforces a high-severity alert gate.
- SCA: Trivy scans the built Docker image for OS and library vulnerabilities.

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
