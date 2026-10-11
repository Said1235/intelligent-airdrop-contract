# REMEDIATION_PLAN

## Done (in this package)
1. Baseline captured (AUDIT_BASELINE.md): lint ok, 95 tests.
2. Evidence gap documented (EVIDENCE_GAP_ANALYSIS.md) and source matrix (EVIDENCE_SOURCE_MATRIX.md).
3. Implemented: `set_evidence_rules` (organizer, once, validated), per-requirement external/subjective declaration, deterministic fetch + checks, independent validator re-derivation, commitment, derived verdict, stored `results/commitment/policy_hash`, `get_evidence_rules`, extended `get_claim`.
4. Tests: 190 Direct Mode tests (95 regression + 95 evidence/adversarial A–O); lint ok; 38 deliberate code breakages all caught.

## Still required from you (cannot be done from this environment)
1. **Deploy to GenLayer Studio and run one real evaluation** with a reachable source. First confirm: `resp.status` field, `UserError.message` equality, and that module-level helpers used inside the nondet closure pickle correctly (API_COMPATIBILITY_AUDIT.md).
2. **Choose a real authoritative source per externally checkable action** and write the rules JSON. Until then those requirements are BLOCKED — NO VERIFIED AUTHORITATIVE SOURCE.
3. **Compare** `sha256sum contracts/intelligent_airdrop.py` with the code pasted into Studio (hash in FINAL_AUDIT.md) before resubmitting.
4. Flow after deploy: `set_requirements([...])` → `set_evidence_rules(<json>)` → `submit_claim` → `evaluate_claim`. This is a new storage layout: deploy as a **new instance**.

## Optional hardening (behaviour changes, not applied)
- Upper-bound / time-window op; per-wallet one-claim index; `MAX_EVIDENCE_CHARS`; reject empty `set_requirements` instead of defaults.
