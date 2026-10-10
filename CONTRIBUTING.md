# Contributing to Git Security Sentinel

Thank you for your interest in contributing to **Git Security Sentinel**! This project is an open-source Agent Skill built for developer privacy, fast deterministic scanning, and local AI contextual review.

## Core Design Principles

1. **Zero External Dependencies**: The core static scanner (`scripts/scan_diff.py`) and local AI reviewer (`scripts/ai_review.py`) use **Python standard library only**. Do not introduce external third-party dependencies (`pip install ...`).
2. **Privacy by Default**: No code or diff content may be sent to third-party hosted cloud APIs. Secrets must be safely redacted before appearing in terminal output, JSON reports, or AI prompts.
3. **Deterministic Authority**: Static pattern-matching rules must remain transparent and reproducible. AI recommendations are advisory and cannot alter deterministic scanner findings or exit codes.
4. **Synthetic Test Secrets**: **NEVER** commit real or usable API keys, tokens, or credentials. Always construct synthetic test secrets programmatically in unit tests.

---

## Development Setup

Git Security Sentinel requires **Python 3.9+** and standard **Git**:

```bash
git clone https://github.com/Harry1402/git-security-sentinel.git
cd git-security-sentinel
```

No package installation step (`pip install`) is necessary.

---

## Running the Automated Test Suite

Run the built-in `unittest` suite locally before submitting any pull request:

```bash
python -m unittest discover -s tests -p "test_*.py" -v
```

All tests must pass, with 100% deterministic results and zero external network access.

---

## Adding New Detection Rules

When proposing a new security rule in `scripts/scan_diff.py`:
1. Assign a distinct rule ID adhering to naming conventions:
   - `secret.<format_name>` for credentials and tokens.
   - `code.<pattern_name>` for risky code execution patterns.
   - `config.<setting_name>` for insecure configuration flags.
2. Provide explicit `severity` (`critical`, `high`, `medium`, `low`) and `confidence` levels (`high`, `medium`, `low`).
3. Ensure proper redaction of matched values in `scan_added_lines` and `sanitize_line_content`.
4. Document the rule in `references/detection-rules.md` including target pattern, synthetic examples, false positives, and remediation guidance.
5. Add unit test coverage in `tests/test_scan_diff.py` using programmatically synthesized test data.

---

## Coding Conventions

- Follow **PEP 8** style guidelines.
- Use explicit type annotations (`from __future__ import annotations`, `typing`).
- Keep functions small, modular, and testable.
- Avoid shell execution shortcuts: never use `shell=True` when invoking Git subprocesses.
- Support Windows, Linux, and macOS cross-platform path handling (`pathlib.Path`).

---

## Pull Request Guidelines

1. Create a descriptive feature branch:
   ```bash
   git checkout -b feature/my-enhancement
   ```
2. Verify Python compilation:
   ```bash
   python -m py_compile scripts/scan_diff.py scripts/ai_review.py tests/test_scan_diff.py
   ```
3. Run tests locally and verify clean exit status.
4. Open a Pull Request against `main` explaining the rationale, security implications, and test coverage.
