# EVIDENCE_SOURCE_MATRIX

The contract ships **no built-in provider**. The eligibility rules are organizer-defined text, so the authoritative source for each externally checkable action is a decision the organizer must make and declare in `set_evidence_rules`. No source is invented here.

## Existing requirements

| Requirement | What must be proven | Authoritative source | Evidence fields | Claimant binding | Verification method | Failure behavior |
|---|---|---|---|---|---|---|
| Default 1 (specific, verifiable action) | An action named by the organizer | **BLOCKED — AUTHORITATIVE EVIDENCE SOURCE NOT ESTABLISHED** (the rule names no action/platform) | — | — | Declared `subjective` → LLM on claimant text only | Cannot be reported as externally verified |
| Default 2 (not vague) | Text quality | n/a (category C) | — | — | LLM, `subjective` | — |
| Organizer rules of class B | per rule | **Declared by organizer** — until declared: **BLOCKED — NO VERIFIED AUTHORITATIVE SOURCE** | see below | see below | see below | see below |

## Contract-supported rule shape (per class-B requirement)

| Aspect | Rule |
|---|---|
| Source | `https://<public DNS host>/…`; IPs, `localhost`, `*.local`, userinfo, `#`, spaces and any placeholder other than `{claimant}` are rejected at configuration time. The claimant never supplies a URL. |
| Claimant binding (mandatory) | `{claimant}` (lower-case checksum-free hex of the stored claimant wallet) in the URL **or** an `eq`/`eq_ci` check whose value is exactly `{claimant}`. A rule with neither is rejected. Account handles are **not** trusted as identity; a platform record qualifies only if it exposes the wallet. |
| Evidence fields | up to 5 checks `{path, op, value}`; `path` = dotted JSON path (array index allowed); ops: `exists`, `eq` (type-strict), `eq_ci` (string, case-insensitive), `gte` (integer). |
| "Not found" records | optional `missing_status` (≤3 HTTP 4xx codes) maps e.g. 404 → MISSING; any other non-200 aborts. |
| Outcomes | `VERIFIED` (all checks pass) · `MISSING` (field/record absent) · `CONTRADICTED` (present but wrong value/identity/campaign) · `SUBJECTIVE` (no external rule). |
| Failure behavior | HTTP error, empty/oversized/malformed body, non-JSON-object/array, wrong value type → **abort, nothing stored, retry possible**. |

Illustrative fixture used in tests (reserved `example.org`, not a recommended provider): requirement "active member" → `GET https://api.example.org/members/{claimant}`, checks `member.wallet eq_ci {claimant}`, `member.status eq "active"`, `missing_status [404]`; requirement "≥3 contributions" → `GET …/contrib?wallet={claimant}`, checks `data.campaign eq "airdrop-2026"`, `data.count gte 3`.

## Source-quality caveat (cannot be enforced by code)

The contract validates a source's *form and claimant binding*, not whether the host is authoritative. A source that anyone can write to (public paste, user-editable gist) is not authoritative even over https. Choosing that source is the organizer's trust decision.
