# EVIDENCE_GAP_ANALYSIS

## Classification of the existing requirements

| Requirement | Class | Why |
|---|---|---|
| Default 1: "describe a specific, verifiable action actually taken by the claimant" | **C** subjective as written | It asks for a *description's quality*; it names no action, platform or record, so no authoritative source can be derived from it. |
| Default 2: "not a vague or generic statement" | **C** subjective | Pure text-quality judgment. |
| Any organizer-written rule such as "joined X", "holds Y", "contributed ≥3 times" | **B** externally verifiable | Provable from an on-chain or platform record. |

The rules are organizer text, so the contract cannot know which are B. The defect: **it treated every B requirement as C** — and as A-like truth once the LLM said "qualifies".

## The exact vulnerability behind the rejection

1. Organizer sets "Must have donated to campaign 7".
2. Mallory submits `evidence_text = "I donated 10 tokens. https://mallory.example/receipt"` (no donation exists, URL is self-hosted).
3. `evaluate_claim` passes the **same string** to the leader's LLM and to every validator's LLM (`prompt_non_comparative`). Nobody fetches the URL; nobody queries any record.
4. All validators judge a fluent, specific-sounding claim against the same unsupported text and agree → `qualifies:meets_all_requirements` is stored.

Consensus here is agreement among LLMs about an unproven assertion; validators add no independent evidence.

## Missing evidence-acquisition steps (per externally checkable requirement)

| Step | Before | Now |
|---|---|---|
| Locate the authoritative record | claimant text | organizer-declared https source with `{claimant}` |
| Fetch it | never | leader **and** each validator, independently |
| Bind to claimant | none | wallet in URL / eq-check; rules without binding are rejected |
| Deterministic check | none | `exists / eq / eq_ci / gte` on the JSON |
| Commit to evidence | none | hash over the exact checked fields; part of the canonical result validators must reproduce |
| Source failure | n/a | aborts evaluation (no record) |
