# API_COMPATIBILITY_AUDIT

Authorities: docs.genlayer.com (web-access, equivalence-principle, prompt-injection pages, full-documentation.txt), sdk.genlayer.com api.txt, and the SDK actually installed for runner `1jb45aa8…` (genvm-manager v0.6.0-rc3, read from disk), exercised with genlayer-test 0.29.2 Direct Mode.

| API used | Docs say | Installed SDK / test | Resolution |
|---|---|---|---|
| `gl.nondet.web.get(url)` | exists; no signature given | `get(url, *, headers)` → `Response` | used with no headers |
| HTTP status field | web-access page example uses `status_code` | `Response.status: int`, `.headers`, `.body: bytes \| None` (api.txt also lists `status`) | **docs page differs from installed SDK** → code uses `.status`; verified by Direct Mode with `mock_web` (500/404/200 branches). Studio not run: ⚠️ UNVERIFIED on a real GenVM. |
| `.body` | bytes | `bytes \| None` | None/empty/oversized handled explicitly |
| `gl.vm.run_nondet_unsafe(leader_fn, validator_fn)` | recommended for custom logic: validator re-runs and compares decision fields | present in SDK; validator exceptions ⇒ Disagree | used. Validator re-derives the full canonical result and compares for equality |
| `gl.eq_principle.strict_eq` | listed for web data | its validator uses `spawn_sandbox`, which Direct Mode cannot execute (`unknown type 14`) | not used (cannot be tested); equivalent behaviour implemented explicitly with `run_nondet_unsafe` |
| `gl.eq_principle.prompt_non_comparative` | validators judge leader output vs criteria | unchanged; used only for requirements declared `subjective` | kept; Direct Mode lacks the `ExecPromptTemplate` backend → test shim in `tests/direct/conftest.py` (LLM backend only) |
| `gl.vm.UserError` | new docs: `.data`; classic SDK: `.message` | installed (classic) has `message: str`, compared by strict equality | `.message` used; messages are short and deterministic |
| `Address.as_hex` | checksum string | present; lower-cased by the contract | verified in tests (URL contains lower-case wallet) |
| `hashlib` / `json` | no stdlib list given | the SDK itself imports `hashlib.sha3_256` | `sha3_256` used (not sha256) because the SDK proves it exists in the runtime; ⚠️ UNVERIFIED that every other stdlib call used behaves identically in GenVM |
| Module-level helpers referenced from nondet closures | not documented | works in Direct Mode with `check_pickling` (no warnings) | ⚠️ UNVERIFIED on a real GenVM whether cloudpickle ships them by value or by reference — **first thing to confirm in Studio** |
| Runner | I200: newer runner `5jycge4q…` available | contract stays on `1jb45…` | intentionally not migrated |
| Request headers, timeouts, redirects | not documented | not exposed | not used; sources needing API keys are unsupported |

Direct Mode quirks found: fixture addresses are raw `bytes` if built before the SDK is imported (tests convert); one contract class per process (no double deploy in a test).
