# Test Matrix

## Summary

- **42 behavioral stress tests** were completed against the skill during
  development.
- The **final integrated test, Test 42 = PASS.**
- Current skill revision: **`CLOSED / PASS`**.
- This is internal engineering validation, not external certification.

This page organises the *behavioral coverage* by category. It intentionally does
**not** reconstruct or paraphrase the 42 individual tests — only the categories
and outcomes that the acceptance record and the skill files actually support are
listed. See [`ACCEPTANCE.md`](ACCEPTANCE.md) for the authoritative record.

---

## Coverage by behavioral category

Each row maps a behavioral category to the rule surface it exercises in
[`../skill/`](../skill/) and to the outcome recorded at acceptance. "Status"
reflects the acceptance record's final integrated validation and its
pre-acceptance consistency audit.

| # | Category | What it checks | Rule surface | Status |
|---|---|---|---|---|
| 1 | Validation-layer classification | A change is routed to the least sufficient level; higher levels are not treated as automatically better; L1–L3 stay the normal loop | `SKILL.md` "Fast routing", `validation-ladder.md` L1/L2/L3 | PASS |
| 2 | Real-time ownership | Application-owned, injectable-clock timing stays L1/L2-testable even at RC time; only a provider-owned schedule under test is real-time-only; unestablished ownership → `UNKNOWN`, fail closed | `SKILL.md` "Level 4 classification sequence" Step B, `evidence-model.md` "What 'real-time-only' means" | PASS |
| 3 | RC gate vs hard exceptions | L4 is considered only for a real payment-capable RC or a defined hard exception; "RC + timing-sensitive" is never by itself sufficient | `SKILL.md` "Critical Level 4 rule", `evidence-model.md` "Level 4 trigger" / "Hard exceptions" | PASS |
| 4 | Evidence substitution — conditions | All four substitution conditions must hold; any failure → `DOES NOT APPLY` / `fresh L4 = REQUIRED` | `evidence-model.md` "Evidence substitution" | PASS |
| 5 | Evidence substitution — fail-closed | `zero blast radius` not confidently established → substitution fails closed | `SKILL.md` "Zero blast radius is a three-state fact", acceptance "evidence substitution fail-closed behavior = PASS" | PASS |
| 6 | Zero blast radius — three states | `YES` / `NO` / `NOT ESTABLISHED` kept distinct; `NOT ESTABLISHED` never reported as demonstrated positive impact | `SKILL.md` / `validation-ladder.md` / `evidence-model.md` three-state tables | PASS |
| 7 | Subset vs complete zero-blast-radius | Subset / partial zero-blast evidence never becomes whole-decision evidence | acceptance "subset vs complete zero-blast-radius handling = PASS", invariant 8 | PASS |
| 8 | Structural relevance vs delta impact | A configuration participating in / governing a behavior ≠ a change to it affecting that behavior; neutral wording required when impact `NOT ESTABLISHED` | `evidence-model.md` "Structural relevance vs. delta impact" | PASS |
| 9 | Evidence substitution — decision scoping | A successful Step C result resolves only the current decision / revision / state; no standing L4 exemption | `SKILL.md` "Evidence substitution is decision / revision / state scoped", `evidence-model.md` "Substitution is decision-scoped" | PASS |
| 10 | Provider-boundary semantics | Semantic interpretation of real provider events / payloads into internal lifecycle state is inside L3 scope, not only the outbound call surface | `SKILL.md` "Provider wire boundary (L3 scope)", `validation-ladder.md` L3 decision table | PASS |
| 11 | L3 classification vs execution state | An explicitly established provider-boundary change is `L3 = REQUIRED` (settled); unexecuted → `REQUIRED — NOT RUN / EVIDENCE MISSING`, never downgraded to `UNKNOWN` | `SKILL.md` "L3 explicit-fact rule", `evidence-model.md` "Evidence-authority rule for L3" | PASS |
| 12 | Evidence freshness — stale vs current L3 | Historical L3 evidence is not accepted for the revision under review when the boundary changed | acceptance "stale / current L3 evidence handling = PASS" | PASS |
| 13 | Evidence freshness — historical L4 coverage | An old `PASS` existing ≠ known historical coverage of the behavior a new decision needs; partial historical coverage handled as partial | acceptance "partial historical L4 coverage handling = PASS", invariant 22 | PASS |
| 14 | Scenario scoping | `L4 REQUIRED` (including via a hard exception) does not imply any specific provider-owned scenario; unestablished set → `UNKNOWN / TO BE DEFINED`, no `REAL SANDBOX PASS` | `SKILL.md` "Level 4 scenario scoping rule", `evidence-model.md` "Hard exceptions force L4 but do not define its scenarios" | PASS |
| 15 | Freeze integrity | Decided only from the four integrity checks (revision, deployment, change, provider-state) | `freeze-protocol.md` "Integrity checks" | PASS |
| 16 | Scenario FAIL vs contamination vs BLOCKED | Three separate facts; a `scenario FAIL` under intact integrity is genuine L4 evidence, not contamination, and does not by itself cause `BLOCKED` or require a mid-freeze fix | `SKILL.md` "Scenario-result state rule", `freeze-protocol.md` section A–D | PASS |
| 17 | Mid-freeze temporal ordering | Proposed → deployed → contamination → restart are ordered; a later state and its consequences are not reported before its triggering event | `SKILL.md` "Mid-freeze state-transition rule", `freeze-protocol.md` "Mid-freeze state machine" | PASS |
| 18 | Active-freeze "L4" is two fields | `current L4 run status` and `fresh L4 restart` reported separately; never a generic `L4 = REQUIRED` | `SKILL.md` "Current Level 4 run status vs. fresh Level 4 restart requirement" | PASS |
| 19 | Proposed-fix evidence isolation | An undeployed fix's L1/L2/L3 states gate only that fix; never attach to the current frozen run's verdict | `SKILL.md` "Proposed-fix evidence isolation", `freeze-protocol.md` section E | PASS |
| 20 | Contamination → restart mechanics | Contamination determines only `fresh L4 restart = REQUIRED` / hard exception `ACTIVE` / `Evidence substitution = DOES NOT APPLY`; mechanics follow the actual cause | `SKILL.md` "Restart-mechanics anti-inference rule", `evidence-model.md` "Contamination-restart hard exception does not determine restart mechanics" | PASS |
| 21 | Deployment verification vs redeployment | `deployment verification required` ≠ `redeployment required`; redeploy only if necessary to reach the authoritative corrected deployed state | `freeze-protocol.md` "Restart", `evidence-model.md` "Baseline requirement vs. operational action" | PASS |
| 22 | Provider-side-only contamination | Same repository revision may stay `UNCHANGED / STILL AUTHORITATIVE`; redeployment not automatically required | `freeze-protocol.md` "Provider-side-only contamination" | PASS |
| 23 | Entity restoration | Fresh disposable entity OR an authoritative restoration procedure that proves the starting state and all integrity checks; fail closed to fresh; a restored entity does not restore a contaminated run's evidence | `SKILL.md` "Contaminated / affected test entity handling", `freeze-protocol.md` "Contaminated test entity" | PASS |
| 24 | Anti-inference discipline | Missing facts stay `UNKNOWN / NOT ESTABLISHED`; an active freeze does not prove `RC = YES` or which hard exception applied; explicit facts outrank adjacent uncertainty | `SKILL.md` "Anti-inference rule", acceptance "anti-inference discipline = PASS" | PASS |
| 25 | Aggregate vs component facts | "No hard exception applies" → `NONE APPLICABLE` only; component predicates stay `UNKNOWN`; an applicable hard exception does not identify which one | `SKILL.md` "Aggregate facts do not establish their component facts", `evidence-model.md` "Aggregate conclusion vs. component facts" | PASS |
| 26 | Active-L4 status-review discipline | Established run-state facts are not expanded into unestablished execution details; `fresh L4 restart = NOT TRIGGERED` justified only by "no contamination event has occurred" | `SKILL.md` "Active-L4 execution / status review", invariant 24 | PASS |
| 27 | Aggregate L4 completion | Aggregate run completion is separate from individual scenario results; `REAL SANDBOX PASS` only after every required scenario completes under intact integrity | `freeze-protocol.md` "Exit", acceptance consistency audit | PASS |
| 28 | Release-authorization separation | A positive validation result never yields "promote", "release approved", or "deploy" unless release authorization is explicitly in scope and every gate is established | `SKILL.md` "Validation sufficiency vs. release authorization", acceptance "release-authorization separation = PASS" | PASS |

---

## Final integrated validation (from the acceptance record)

| Component behavior | Result |
|---|---|
| Test 42 (final integrated) | PASS |
| Stale / current L3 evidence handling | PASS |
| Partial historical L4 coverage handling | PASS |
| Subset vs complete zero-blast-radius handling | PASS |
| Evidence substitution fail-closed behavior | PASS |
| Scenario scoping | PASS |
| Freeze / contamination / restart separation | PASS |
| Anti-inference discipline | PASS |
| Release-authorization separation | PASS |

## Pre-acceptance consistency audit

A final consistency audit across the four skill files
(`SKILL.md`, `references/validation-ladder.md`, `references/evidence-model.md`,
`references/freeze-protocol.md`) returned:

> **PASS — no contradiction, regression, unsupported inference rule, or
> cross-file conflict found.**

## Acceptance boundary

- No Test 43 is required for current acceptance.
- `CLOSED / PASS` applies to the current skill revision only.
- Any future skill edit requires a targeted regression review before this record
  may be relied upon for the edited state.
- Future tests should be added only when a real project introduces a new
  payment-validation pattern, a new provider behavior, an architecture change, or
  a discovered regression.
