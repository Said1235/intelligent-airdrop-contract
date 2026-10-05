# Intelligent Airdrop

AI-verified eligibility oracle for GenLayer. See `contracts/intelligent_airdrop.py` for the contract and `AUDIT_REPORT.md` for the audit of the `status:reason` change.

```bash
pip install genlayer-test==0.29.2 genvm-linter==0.11.1rc2 cloudpickle pytest
genvm-lint check contracts/intelligent_airdrop.py
pytest
```
