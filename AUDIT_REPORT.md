# AUDIT_REPORT — Intelligent Airdrop, evidence acquisition round

Contract SHA-256: `556b6c057833c28e47c6316aadd3a737d55018d6ad24580f164bc936cb5b8173` (runner `1jb45…`, unchanged). Result: see FINAL_AUDIT.md (**AUDIT UNVERIFIED**; no CRITICAL/HIGH open in the contract logic).

## Findings fixed in this round

### EVD-01 — HIGH — No evidence acquisition (the rejection)
- **Requirement / source:** every externally checkable requirement; no source existed.
- **Current behavior (before):** validators re-judged the same claimant text.
- **Failure scenario:** see EVIDENCE_GAP_ANALYSIS.md (fabricated donation + self-hosted URL → `qualifies`).
- **Why validation failed:** leader and validators shared one unsupported input; agreement proved nothing.
- **Correction:** organizer-declared `external` rules; leader and each validator fetch the source and derive the same canonical result (`_collect_evidence`, `run_nondet_unsafe`).
- **Regression tests:** `test_A_unsupported_assertion_cannot_qualify`, `test_H_all_verified_qualifies_with_commitment`, `test_I_validator_rejects_when_decision_relevant_evidence_differs`.
- **Official source:** web-access page ("leader and validators make independent requests"; return stable fields); equivalence-principle page (`run_nondet_unsafe`: validator re-runs and compares decision fields).

### EVD-02 — HIGH — Claimant-supplied URL as locator/proof
- **Before:** a URL in `evidence_text` was only text.
- **Scenario:** claimant points to a page he controls.
- **Correction:** claimants never supply locators; sources come only from organizer rules; `evidence_text` is not an input to stage 1.
- **Tests:** `test_B_claimant_supplied_url_is_never_fetched_or_trusted`.
- **Source:** prompt-injection page (restrict inputs).

### EVD-03 — HIGH — No claimant identity binding
- **Scenario:** a valid record of someone else's activity is presented as one's own.
- **Correction:** rule must contain `{claimant}` (URL or eq/eq_ci); wallet comes from the stored `sender_address`; unbound rules are rejected at configuration.
- **Tests:** `test_D_identity_mismatch_is_contradicted`, `test_set_evidence_rules_rejects_bad_rules[not bound…]`, `test_N_two_callers_get_independent_evidence`.

### EVD-04 — MEDIUM — Decision-bearing fields not bound to consensus
- **Correction:** consensus value = canonical JSON `{v, claimant, policy, results[], commitment}`; the verdict is derived deterministically from `results` after consensus; `status/reason/results/commitment/policy_hash` are written together. A forged leader result is rejected (`test_validator_rejects_a_forged_leader_result`) and malformed results are rejected by `_parse_evidence`.

### EVD-05 — MEDIUM — Source failure could become a verdict
- **Correction:** non-200 (except organizer-declared `missing_status`), empty/oversized/malformed body, unexpected structure → `UserError`, no write, retry possible; "action disproven" (MISSING/CONTRADICTED) is distinct from "could not verify now".
- **Tests:** `test_KL_source_failure_aborts_and_claim_stays_pending` (11 cases), `test_aborted_evaluation_can_be_retried_when_source_recovers`.

### EVD-06 — MEDIUM — No source/evidence commitment
- **Correction:** `commitment` = SHA3-256 over `[index, url, [[path, op, observed], …]]` of every external rule; part of the canonical string the validators must reproduce byte-for-byte; stored with the claim. Volatile, unchecked response fields (timestamps, counters) are deliberately excluded so honest validators do not diverge; **any checked value** that differs changes the commitment and breaks consensus (`test_I_irrelevant_fields_do_not_break_consensus_but_checked_ones_do`, `test_I_commitment_changes_when_a_checked_value_changes`). Trade-off: unchecked fields are not committed.

### EVD-07 — MEDIUM — Every rule judged by LLM from text
- **Correction:** the LLM runs only for requirements the organizer explicitly declares `subjective`, only if no external requirement is MISSING/CONTRADICTED, sees only the subjective rules, and never sees fetched documents (`test_M_fetched_documents_never_reach_the_llm`, `test_mixed_llm_not_called_when_external_missing_even_if_llm_would_qualify`).

## Residual risks (not fixable in contract code)

| ID | Sev | Risk | Mitigation / recommendation |
|---|---|---|---|
| R-01 | MEDIUM | The organizer is the trust root: the contract checks a source's form and claimant binding, not whether the host is authoritative (a user-writable source is still "https"). | Choose first-party / chain-backed sources; publish rules (`get_evidence_rules`). |
| R-02 | MEDIUM | A class-B rule declared `subjective` is again judged from claimant text. Declaration is explicit but not policed. | Review the rules before use; the default requirements are subjective by nature (BLOCKED as external). |
| R-03 | MEDIUM | Sources can change between leader and validator fetch; no block/finality pinning exists in the API surface used. | Prefer finalized/immutable records; a mismatch yields disagreement/retry, never acceptance. Liveness cost only. |
| R-04 | LOW | Expressiveness: no upper bound / time-window op, no array search (only fixed indices), no auth headers, redirects/timeouts not controllable, no pagination. | Use per-wallet endpoints exposing a scalar; extend ops only if needed. |
| R-05 | LOW | Carry-over from round 1: caller-chosen global `claim_id` can be squatted; one wallet may hold many claims; `evidence_text` unbounded (affects only the submitter's subjective stage); `set_requirements(None/[])` installs defaults. | Key downstream logic on `claimant`, dedupe by wallet. |
| R-06 | LOW | Prompt injection remains possible in the subjective stage (inherent to LLM judgment). | Reduced to explicitly subjective rules; human review for high-value payouts. |
| R-07 | — | Studio/integration, real consensus, source-to-deployment match not demonstrated here. | REMEDIATION_PLAN.md steps 1–3. |

## Result binding matrix

| Field | Leader produces | Validator verifies independently | Stored | Decision-bearing | Derived |
|---|---|---|---|---|---|
| `results[i]` (VERIFIED/MISSING/CONTRADICTED/SUBJECTIVE) | yes | yes — re-fetches and re-derives; byte-equality | yes (`results`) | yes | by deterministic checks |
| `commitment` | yes | yes — part of the compared string | yes | evidence link | hash of checked fields |
| `claimant`, `policy` in result | yes | yes | `policy_hash` yes | binds to rules | from storage |
| `status`, `reason` | no | n/a — not proposed | yes | yes | **derived after consensus** from `results` (+ whitelisted LLM pair for subjective only) |
| subjective LLM pair | yes | validators judge it (`prompt_non_comparative`) then whole-pair whitelist | via status/reason | yes (only if all external VERIFIED) | split of validated string |
| `evidence_text` | claimant | not used for external rules | yes | subjective stage only | — |

No stored decision-bearing field comes from the leader unverified.
