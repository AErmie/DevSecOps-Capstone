# DevSecOps-Capstone

[![Security Scans Passing](https://github.com/AErmie/DevSecOps-Capstone/actions/workflows/main.yml/badge.svg)](https://github.com/AErmie/DevSecOps-Capstone/actions/workflows/main.yml)

[Capstone project reference](https://devsecblueprint.com/learn/devsecops/capstone/index)

## Phase 1: Securing the DNA (SAST + SCA)

This repository implements Phase 1 gates in GitHub Actions to block vulnerable code and dependencies before merge.

### Security Gates

- SAST: Semgrep runs with `p/security-audit` and `p/python` rule packs.
- SAST: CodeQL runs as a dedicated workflow and enforces a high-severity alert gate.
- SCA: Trivy scans third-party libraries and fails on high/critical dependency vulnerabilities.
- DAST: OWASP ZAP launches against the live API after the container is started inside CI.

### Triggers

- Pull requests: runs through `.github/workflows/pr.yml`.
- Push to `main`: runs through `.github/workflows/main.yml`.
- CodeQL SAST: runs through `.github/workflows/codeql.yml` on pull requests and pushes.

### Blocking Policy

- Semgrep fails the build on `ERROR` findings.
- CodeQL fails the build when open high/critical alerts are detected for the current ref.
- Trivy fails the build on `HIGH` and `CRITICAL` vulnerabilities.
- OWASP ZAP fails the build when runtime alerts are detected against the live target.

### Milestone Outcome

Any pull request or push containing high-severity security risk fails CI and must be remediated before proceeding.

### Phase 1 Status

- Implemented: SAST (Semgrep + CodeQL) and SCA (Trivy library scan) gates are active in GitHub Actions.
- Implemented: High/Critical policy is enforced as a hard fail in CI.
- Current validation behavior: this intentionally vulnerable baseline triggers
  Trivy findings (for example `h11`, `starlette`, `black`) at HIGH/CRITICAL
  severity, so the PR security gate fails as expected until vulnerabilities are
  remediated.

## Phase 2: The Stress Test (DAST)

Phase 2 extends the pipeline with a live DAST gate that exercises the running
Skyline API rather than only scanning source code or dependencies.

### Phase 2 Flow

- GitHub Actions builds the container image and starts the API in an isolated CI job.
- The workflow waits for `http://127.0.0.1:8080/health` to report ready.
- OWASP ZAP scans the live OpenAPI target at `http://127.0.0.1:8080/openapi.json`.
- The DAST job blocks downstream execution if ZAP finds runtime issues.

### Phase 2 Milestone Outcome

The repository now runs DAST automatically after the application is effectively
deployed inside a temporary CI container, proving that the live runtime gates
are checked and can block unsafe changes.

## Phase 3: Locking the Vault (Secrets & Config)

Phase 3 eliminates the hardcoded credential that has been present since the
initial commit and adds automated guardrails that prevent new secrets from
ever reaching the repository.

### Secrets and Config Enhancements

- **`main.py`**: `API_SECRET` is no longer a module-level constant.  The
  `/secure-data` endpoint now reads `os.getenv("API_SECRET")` at request time so
  that the value is injected by the runtime environment, not baked into source.
- **`tests/test_main.py`**: The corresponding test switches from a hardcoded
  token literal to `monkeypatch.setenv` so the test fixture never commits a
  credential of its own.
- **GitHub Actions — `gitleaks.yml`**: New reusable workflow that runs a full
  `gitleaks/gitleaks-action@v2` scan with `fetch-depth: 0` (full history) on
  every PR and push.
- **`pr.yml` / `main.yml`**: Both orchestrators call the new `secret-scan` job
  and gate `build-image` on it (`needs: secret-scan`).  A leak blocks the
  pipeline before a single build artefact is created.
- **`.gitleaks.toml`**: GitLeaks policy file that extends the built-in ruleset
  and adds a narrowly scoped allowlist for historical test-fixture references
  that predate this phase.
- **`.pre-commit-config.yaml`**: GitLeaks hook added so developers are warned
  about leaks at commit time, before code ever reaches GitHub.
- **`unit-sec-test.yml`**: The `docker logs` diagnostic command is now filtered
  through `grep -Ev` to strip `KEY=value` lines so container environment
  variables are never echoed into the public CI log.

### Phase 3 Flow

1. Developer commits → pre-commit GitLeaks hook fires locally.
2. PR opened → `secret-scan` job runs; `build-image` only starts after it passes.
3. If any new secret is found, the job exits non-zero and the PR is blocked.
4. Secrets required at runtime (e.g., `API_SECRET`) are stored as GitHub
   Encrypted Secrets and injected as environment variables; they never appear
   in source code or CI logs.

### Phase 3 Milestone Outcome

Every commit path — local pre-commit and GitHub Actions — now runs a secret
scan.  The pipeline blocks on any new credential leakage, and the existing
codebase has been remediated so that no plaintext secret remains in the
current working tree.  Historical findings are acknowledged by commit SHA in
`.gitleaks.toml`; no literal credential value is stored in any tracked file.

## Phase 4: The Nervous System (Feedback & Reporting)

Phase 4 adds continuous security visibility so developers see exactly why a
change was blocked, which checks failed, and what needs remediation.

### What changed

- **`security-report.yml`**: New reusable workflow that aggregates security
  check outcomes into a single professional Markdown report.
- **`pr.yml` / `main.yml`**: New terminal `security-feedback-report` job runs
  with `if: always()` after the security chain and publishes report artifacts.
- **GitHub Step Summary broadcast**: Every run appends the consolidated report
  to the Actions summary for immediate visibility.
- **GitHub PR comment broadcast**: PR runs publish or update a single
  bot-managed security intelligence comment in real time.
- **README status badge**: Added `Security Scans` badge to signal
  security pipeline health from the main orchestrator workflow.

### Phase 4 Flow

1. Security jobs execute through the existing chain (secret scan, build,
   lint, SAST/SCA, DAST).
2. Reporting job runs regardless of upstream success (`if: always()`).
3. Workflow collects check-run conclusions and synthesizes one Markdown report.
4. Report is uploaded as a GitHub artifact and appended to Step Summary.
5. On pull requests, the bot posts or updates a persistent security comment.

### Phase 4 Milestone Outcome

Skyline now has an automated reporting system that turns raw scanner results
into actionable developer feedback, broadcasts real-time status inside GitHub,
and provides a visible release-readiness badge for security pipeline health.

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

To exercise the live API locally before pushing, run the container and verify
the readiness and OpenAPI endpoints:

```bash
docker build -t python-fastapi:local .
docker run --rm -p 8080:8080 python-fastapi:local
curl http://127.0.0.1:8080/health
curl http://127.0.0.1:8080/openapi.json
```

## Repository Hygiene Files

- `.pre-commit-config.yaml`: local quality/security hooks for markdown, YAML, Python, and repository safety checks.
- `.editorconfig`: editor defaults for encoding, line endings, and trailing whitespace behavior.
- `.gitattributes`: consistent line-ending behavior by file type.
- `.markdownlint.yaml`: markdown linting rules used by pre-commit.
- `.yamllint.yaml`: YAML linting policy used by pre-commit.
