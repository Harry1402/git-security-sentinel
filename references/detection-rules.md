# Git Security Sentinel: Detection Rules Reference

This reference document defines the complete detection rule catalog implemented in `scripts/scan_diff.py`.

> **Important Security Notice**: Static pattern matching detects known structural indicators and heuristic signatures. Static analysis cannot definitively prove that an exposed credential has active permissions or that every arbitrary vulnerability will be detected. Always combine static checks with secure secret management, least-privilege permissions, and active credential rotation.

---

## Summary of Rules

| Rule ID | Category | Severity | Confidence | Target Pattern |
|---|---|---|---|---|
| `secret.aws_access_key` | `secrets` | `high` | `high` | AWS IAM / root Access Key IDs |
| `secret.aws_secret_key` | `secrets` | `high` | `medium` | 40-character base64 AWS Secret Keys |
| `secret.github_pat_classic` | `secrets` | `high` | `high` | GitHub Classic Personal Access Tokens (`ghp_...`) |
| `secret.github_pat_fine_grained` | `secrets` | `high` | `high` | GitHub Fine-Grained Personal Access Tokens (`github_pat_...`) |
| `secret.github_token_scoped` | `secrets` | `high` | `high` | GitHub OAuth, User-to-Server, or Server Tokens (`gho_`, `ghu_`, `ghs_`, `ghr_`) |
| `secret.private_key` | `secrets` | `critical` | `high` | Private cryptographic key headers (RSA, EC, DSA, OpenSSH, PGP) |
| `secret.slack_token` | `secrets` | `high` | `high` | Slack Bot, User, or App API tokens (`xoxb-`, `xoxp-`, etc.) |
| `secret.generic_credential` | `secrets` | `high` | `medium` | Generic variable assignments for passwords, tokens, API keys |
| `code.dangerous_eval` | `risky_code` | `medium` | `medium` | Dynamic runtime evaluation via `eval()` or `exec()` |
| `config.hardcoded_localhost` | `configuration` | `low` | `low` | Hardcoded loopback service endpoints (`127.0.0.1`, `localhost`) |

---

## Detailed Rule Specifications

### 1. `secret.aws_access_key`
- **Category**: `secrets`
- **Severity**: `high`
- **Confidence**: `high`
- **Detection Approach**: Regex search for standard 20-character AWS Access Key ID format: `\b(AKIA[0-9A-Z]{16})\b`.
- **Illustrative Example (Synthetic)**:
  ```python
  AWS_KEY = "AKIA[16_CHAR_ALPHANUMERIC_KEY]"
  ```
- **Expected False Positives**: Extremely rare; occasional collisions with hash digests or documentation strings that coincidentally begin with `AKIA`.
- **Recommended Remediation**: Revoke the exposed IAM key in the AWS IAM Console immediately. Use AWS IAM Roles (e.g. AWS Instance Metadata / ECS task roles) or environment variables instead of committing keys.
- **Known Limitations**: Does not catch temporary session keys beginning with `ASIA` unless explicitly extended.

---

### 2. `secret.aws_secret_key`
- **Category**: `secrets`
- **Severity**: `high`
- **Confidence**: `medium`
- **Detection Approach**: Named regex matching assignments to `aws_secret_access_key`, `aws_secret_key`, or `aws_secret` containing a 40-character base64/alphanumeric literal value.
- **Illustrative Example (Synthetic)**:
  ```ini
  aws_secret_access_key = "<YOUR_40_CHARACTER_AWS_SECRET_KEY>"
  ```
- **Expected False Positives**: Configuration files using dummy placeholder strings or documentation examples. Filtered automatically when common placeholder terms (`example`, `test`, `dummy`) or template syntax (`${VAR}`) are used.
- **Recommended Remediation**: Rotate the associated AWS key pair immediately. Migrate secrets to AWS Secrets Manager or Parameter Store.
- **Known Limitations**: Cannot detect standalone AWS secrets without contextual variable names.

---

### 3. `secret.github_pat_classic`
- **Category**: `secrets`
- **Severity**: `high`
- **Confidence**: `high`
- **Detection Approach**: Regex boundary matching standard GitHub Personal Access Token prefixes: `\b(ghp_[0-9a-zA-Z]{36})\b`.
- **Illustrative Example (Synthetic)**:
  ```bash
  export GITHUB_TOKEN="ghp_<YOUR_36_CHARACTER_TOKEN_STRING>"
  ```
- **Expected False Positives**: Near zero. GitHub's `ghp_` prefix is format-specific and checksum-validated by GitHub servers.
- **Recommended Remediation**: Revoke the token via **GitHub Settings > Developer Settings > Personal Access Tokens**. Issue a new token with minimal necessary scopes.
- **Known Limitations**: Only identifies the classic 40-character format.

---

### 4. `secret.github_pat_fine_grained`
- **Category**: `secrets`
- **Severity**: `high`
- **Confidence**: `high`
- **Detection Approach**: Regex matching GitHub Fine-Grained Personal Access Tokens: `\b(github_pat_[0-9a-zA-Z_]{82})\b`.
- **Illustrative Example (Synthetic)**:
  ```python
  gh_token = "github_pat_<YOUR_82_CHARACTER_TOKEN_STRING>"
  ```
- **Expected False Positives**: None identified.
- **Recommended Remediation**: Revoke the fine-grained token in GitHub Developer Settings and check audit logs for unauthorized access.
- **Known Limitations**: Only covers the 93-character fine-grained format.

---

### 5. `secret.github_token_scoped`
- **Category**: `secrets`
- **Severity**: `high`
- **Confidence**: `high`
- **Detection Approach**: Regex pattern matching scoped GitHub OAuth (`gho_`), User-to-Server (`ghu_`), Server-to-Server (`ghs_`), and Refresh (`ghr_`) tokens.
- **Illustrative Example (Synthetic)**:
  ```javascript
  const oauthToken = "gho_<YOUR_SCOPED_TOKEN_STRING>";
  ```
- **Expected False Positives**: Extremely low due to vendor-specific prefixes.
- **Recommended Remediation**: Revoke the app token in GitHub Apps management or OAuth App settings.
- **Known Limitations**: Does not verify whether the token has expired.

---

### 6. `secret.private_key`
- **Category**: `secrets`
- **Severity**: `critical`
- **Confidence**: `high`
- **Detection Approach**: Regex pattern matching standard PEM / OpenSSH / PGP private key header blocks:
  `-----BEGIN (?:RSA|DSA|EC|OPENSSH|PGP|ENCRYPTED)? ?PRIVATE KEY-----`.
- **Illustrative Example (Synthetic)**:
  ```text
  -----BEGIN [TYPE] PRIVATE KEY-----
  MIIEowIBAAKCAQEA0m...
  -----END [TYPE] PRIVATE KEY-----
  ```
- **Expected False Positives**: Intentional public documentation or test dummy keys committed in mock directories.
- **Recommended Remediation**: Never commit private keys to Git. Revoke certificates associated with the compromised key and generate a fresh key pair.
- **Known Limitations**: Detects headers in added lines; multi-line payload bodies are flagged via the header match.

---

### 7. `secret.slack_token`
- **Category**: `secrets`
- **Severity**: `high`
- **Confidence**: `high`
- **Detection Approach**: Regex matching standard Slack token formats: `\b(xox[baprs]-[0-9a-zA-Z]{10,48})\b`.
- **Illustrative Example (Synthetic)**:
  ```python
  slack_bot_token = "xoxb-<YOUR_SLACK_BOT_TOKEN>"
  ```
- **Expected False Positives**: Low. Filtered automatically if standard test mocks are present.
- **Recommended Remediation**: Revoke the token in the Slack API app dashboard (`api.slack.com/apps`).
- **Known Limitations**: Does not validate token validity against Slack's auth API.

---

### 8. `secret.generic_credential`
- **Category**: `secrets`
- **Severity**: `high`
- **Confidence**: `medium`
- **Detection Approach**: Heuristic regex scanning variable assignments for sensitive identifiers (`password`, `api_key`, `secret`, `client_secret`, `auth_token`) assigned string literals of length >= 8 characters.
- **Illustrative Example (Synthetic)**:
  ```python
  DATABASE_PASSWORD = "<YOUR_DATABASE_PASSWORD_HERE>"
  ```
- **Expected False Positives**: Development mock strings, unit test fixtures, or configuration variable lookups. Built-in filters ignore environment variable calls (`process.env`, `os.environ`, `System.getenv`), templates (`${VAR}`), and obvious placeholders (`<PLACEHOLDER>`, `dummy`, `test`).
- **Recommended Remediation**: Store secret in an untracked `.env` file (listed in `.gitignore`) and load via environment variables or a secret vault.
- **Known Limitations**: May not catch heavily obfuscated string constructions or indirect dictionary references.

---

### 9. `code.dangerous_eval`
- **Category**: `risky_code`
- **Severity**: `medium`
- **Confidence**: `medium`
- **Detection Approach**: Static regex matching invocations of `eval(...)` and `exec(...)`.
- **Illustrative Example**:
  ```python
  result = eval(user_supplied_query)
  ```
- **Expected False Positives**: Mathematical expression evaluators, code generation build scripts, or controlled internal tools.
- **Recommended Remediation**: Use safe, structured alternatives such as `json.loads()`, `ast.literal_eval()`, or dedicated parsers.
- **Known Limitations**: Does not perform dynamic taint analysis or dataflow tracking across functions.

---

### 10. `config.hardcoded_localhost`
- **Category**: `configuration`
- **Severity**: `low`
- **Confidence**: `low`
- **Detection Approach**: Regex matching hardcoded loopback URLs (`http://localhost`, `http://127.0.0.1`, `http://0.0.0.0`).
- **Illustrative Example**:
  ```python
  API_BASE = "http://127.0.0.1:8000/v1"
  ```
- **Expected False Positives**: Local development test files or integration test assertions.
- **Recommended Remediation**: Parameterize endpoints using configurable environment variables or config files.
- **Known Limitations**: Flags development convenience patterns; developer review is expected to determine if the pattern is intentional.
