# Intelligent Airdrop

AI-assisted eligibility oracle for GenLayer with contract-side evidence acquisition.

Flow: `set_requirements([...])` → `set_evidence_rules(<json>)` → `submit_claim` → `evaluate_claim`.
Each requirement is declared `external` (https source queried with the claimant's wallet + deterministic JSON checks) or `subjective` (LLM on the claimant's text). Rule format: `EVIDENCE_SOURCE_MATRIX.md`. Audit: `FINAL_AUDIT.md`.

```bash
pip install genlayer-test==0.29.2 genvm-linter==0.11.1rc2 cloudpickle pytest
genvm-lint check contracts/intelligent_airdrop.py
pytest
```
