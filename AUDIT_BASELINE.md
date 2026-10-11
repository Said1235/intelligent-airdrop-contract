# AUDIT_BASELINE — state before the evidence-acquisition change

Inspected before modifying anything (Phase Zero).

| Item | Finding |
|---|---|
| Contract | `contracts/intelligent_airdrop.py`, class `IntelligentAirdrop`, runner `py-genlayer:1jb45aa8ynh2a9c9xn3b7qqh8sm5q93hwfp7jqmwsfhh8jpz09h6` |
| SHA-256 | `db98eaee4505734d5a32664b01a8edd459debea64fed89fbd4f76e9cbba972d3` (round-1 deliverable: original + `verdict:reason` change) |
| Eligibility requirements | **None are fixed in code.** `set_requirements(list[str])` takes free text from the organizer (max 20 × 500 chars), once. Only fallback: 2 generic defaults ("must describe a specific, verifiable action actually taken by the claimant", "must not be a vague or generic statement"). |
| Claimant inputs | `claim_id: u256` (caller-chosen), `evidence_text: str` (free text; may contain URLs). Claimant = `gl.message.sender_address`. |
| Verdict logic | single LLM call → one of 5 whitelisted `status:reason` strings → split → stored (INVARIANT-08). |
| Public methods | write: `set_requirements`, `submit_claim`, `evaluate_claim`; view: `get_requirements`, `get_claim`. Constructor takes no arguments (deploys without data). |
| Leader/validator | `gl.eq_principle.prompt_non_comparative(fn, task, criteria)` where `fn` returns the stored `evidence_text`. Leader asks an LLM; validators ask an LLM whether the leader's answer fits the same text. **No web access anywhere in the contract.** |
| Storage | `organizer`, `requirements: DynArray[str]`, `claims: TreeMap[u256, Claim(claimant, evidence_text, status, reason)]`; explicit key, duplicate ids rejected, no list/latest lookup. |
| Tests | 95 Direct Mode tests (round 1). **No integration tests, no deployment scripts exist in the project.** |
| Linter | `genvm-lint check` ok (3 passed; I200 "newer runner available" only). |
| Baseline run | lint ok; `pytest` → 95 passed (before any change in this round). |

Conclusion: every requirement, including externally checkable ones, is judged only from claimant-supplied text.
