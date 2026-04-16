# Changelog

## 1.0.0

- Froze the public `wiki` provider contract: CLI commands, provider config keys, tool names, and recall block structure remain unchanged.
- Locked `schema_version: 1` as the stable note schema while keeping legacy notes readable and unsupported notes read-only.
- Finalized deterministic semantic merge, episodic dedupe, and rule-based ingest/recall quality improvements as the long-term supported path.
- Synced package metadata, plugin metadata, examples, and release docs to the `1.0.0` stabilization target.
- Added contract-focused regression coverage for provider schemas, tool payloads, and recall block formatting.
- Added release-engineering guardrails for `1.0.0`: `twine check`, clean-install verification from wheel/sdist, and an isolated Hermes smoke runbook.

## 0.1.0

- Added `CompilerService` to centralize ingest orchestration.
- Added `SessionWritebackService` so the Hermes provider stays a thin adapter.
- Expanded `doctor`, `reindex`, and `compact` with structured diagnostics and dry-run maintenance reports.
- Added regression tests for compiler orchestration, provider writeback fallback, and CLI diagnostics.
- Documented the `conda hermes-dev` Python 3.12 workflow and the manual Hermes smoke-test prerequisite.
