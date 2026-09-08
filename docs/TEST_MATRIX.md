# Test Matrix

## Summary

- The rules were developed by iterating against **42 hand-constructed scenarios**.
- This is an internal engineering development record, not external certification.
- There is no automated test harness in this repository.

This page organises the *scenario coverage* by category. It intentionally does
**not** reconstruct or paraphrase the 42 individual scenarios — only the
categories and the rule surface that the skill files actually support are listed.
See [`DEVELOPMENT_RECORD.md`](DEVELOPMENT_RECORD.md) for what the development process established.

---

## Coverage by behavioral category

Each row maps a behavioral category to the rule surface in
[`../skill/`](../skill/) that enforces it. "Checked" (✓) means the category's
scenarios were worked through by hand and the listed rule is present and
internally consistent across the skill files — not that an automated test passed.

| # | Category | What it checks | Rule surface | Checked |
|---|---|---|---|---|
| 1 | Validation-layer classification | A change is routed to the least sufficient level; higher levels are not treated as automatically better; L1–L3 stay the normal loop | `SKILL.md` "Fast routing", `validation-ladder.md` L1/L2/L3 | ✓ |
| 2 | Real-time ownership | Application-owned, injectable-clock timing stays L1/L2-testable even at RC time; only a provider-owned schedule under test is real-time-only; unestablished ownership → `UNKNOWN`, fail closed | `SKILL.md` "Level 4 classification sequence" Step B, `evidence-model.md` "What 'real-time-only' means" | ✓ |
| 3 | RC gate vs hard exceptions | L4 is considered only for a real payment-capable RC or a defined hard exception; "RC + timing-sensitive" is never by itself sufficient | `SKILL.md` "Critical Level 4 rule", `evidence-model.md` "Level 4 trigger" / "Hard exceptions" | ✓ |
| 4 | Evidence substitution — conditions | All four substitution conditions must hold; any failure → `DOES NOT APPLY` / `fresh L4 = REQUIRED` | `evidence-model.md` "Evidence substitution" | ✓ |
| 5 | Evidence substitution — fail-closed | `zero blast radius` not confidently established → substitution fails closed | `SKILL.md` "Zero blast radius is a three-state fact", DEVELOPMENT_RECORD.md "evidence substitution fail-closed behavior" | ✓ |
| 6 | Zero blast radius — three states | `YES` / `NO` / `NOT ESTABLISHED` kept distinct; `NOT ESTABLISHED` never reported as demonstrated positive impact | `SKILL.md` / `validation-ladder.md` / `evidence-model.md` three-state tables | ✓ |
| 7 | Subset vs complete zero-blast-radius | Subset / partial zero-blast evidence never becomes whole-decision evidence | DEVELOPMENT_RECORD.md "subset vs complete zero-blast-radius handling", invariant 8 | ✓ |
| 8 | Structural relevance vs delta impact | A configuration participating in / governing a behavior ≠ a change to it affecting that behavior; neutral wording required when impact `NOT ESTABLISHED` | `evidence-model.md` "Structural relevance vs. delta impact" | ✓ |
| 9 | Evidence substitution — decision scoping | A successful Step C result resolves only the current decision / revision / state; no standing L4 exemption | `SKILL.md` "Evidence substitution is decision / revision / state scoped", `evidence-model.md` "Substitution is decision-scoped" | ✓ |
| 10 | Provider-boundary semantics | Semantic interpretation of real provider events / payloads into internal lifecycle state is inside L3 scope, not only the outbound call surface | `SKILL.md` "Provider wire boundary (L3 scope)", `validation-ladder.md` L3 decision table | ✓ |
| 11 | L3 classification vs execution state | An explicitly established provider-boundary change is `L3 = REQUIRED` (settled); unexecuted → `REQUIRED — NOT RUN / EVIDENCE MISSING`, never downgraded to `UNKNOWN` | `SKILL.md` "L3 explicit-fact rule", `evidence-model.md` "Evidence-authority rule for L3" | ✓ |
| 12 | Evidence freshness — stale vs current L3 | Historical L3 evidence is not accepted for the revision under review when the boundary changed | DEVELOPMENT_RECORD.md "stale / current L3 evidence handling" | ✓ |
| 13 | Evidence freshness — historical L4 coverage | An old `PASS` existing ≠ known historical coverage of the behavior a new decision needs; partial historical coverage handled as partial | DEVELOPMENT_RECORD.md "partial historical L4 coverage handling", invariant 22 | ✓ |
| 14 | Scenario scoping | `L4 REQUIRED` (including via a hard exception) does not imply any specific provider-owned scenario; unestablished set → `UNKNOWN / TO BE DEFINED`, no `REAL SANDBOX PASS` | `SKILL.md` "Level 4 scenario scoping rule", `evidence-model.md` "Hard exceptions force L4 but do not define its scenarios" | ✓ |
| 15 | Freeze integrity | Decided only from the four integrity checks (revision, deployment, change, provider-state) | `freeze-protocol.md` "Integrity checks" | ✓ |
| 16 | Scenario FAIL vs contamination vs BLOCKED | Three separate facts; a `scenario FAIL` under intact integrity is genuine L4 evidence, not contamination, and does not by itself cause `BLOCKED` or require a mid-freeze fix | `SKILL.md` "Scenario-result state rule", `freeze-protocol.md` section A–D | ✓ |
| 17 | Mid-freeze temporal ordering | Proposed → deployed → contamination → restart are ordered; a later state and its consequences are not reported before its triggering event | `SKILL.md` "Mid-freeze state-transition rule", `freeze-protocol.md` "Mid-freeze state machine" | ✓ |
| 18 | Active-freeze "L4" is two fields | `current L4 run status` and `fresh L4 restart` reported separately; never a generic `L4 = REQUIRED` | `SKILL.md` "Current Level 4 run status vs. fresh Level 4 restart requirement" | ✓ |
| 19 | Proposed-fix evidence isolation | An undeployed fix's L1/L2/L3 states gate only that fix; never attach to the current frozen run's verdict | `SKILL.md` "Proposed-fix evidence isolation", `freeze-protocol.md` section E | ✓ |
| 20 | Contamination → restart mechanics | Contamination determines only `fresh L4 restart = REQUIRED` / hard exception `ACTIVE` / `Evidence substitution = DOES NOT APPLY`; mechanics follow the actual cause | `SKILL.md` "Restart-mechanics anti-inference rule", `evidence-model.md` "Contamination-restart hard exception does not determine restart mechanics" | ✓ |
| 21 | Deployment verification vs redeployment | `deployment verification required` ≠ `redeployment required`; redeploy only if necessary to reach the authoritative corrected deployed state | `freeze-protocol.md` "Restart", `evidence-model.md` "Baseline requirement vs. operational action" | ✓ |
| 22 | Provider-side-only contamination | Same repository revision may stay `UNCHANGED / STILL AUTHORITATIVE`; redeployment not automatically required | `freeze-protocol.md` "Provider-side-only contamination" | ✓ |
| 23 | Entity restoration | Fresh disposable entity OR an authoritative restoration procedure that proves the starting state and all integrity checks; fail closed to fresh; a restored entity does not restore a contaminated run's evidence | `SKILL.md` "Contaminated / affected test entity handling", `freeze-protocol.md` "Contaminated test entity" | ✓ |
| 24 | Anti-inference discipline | Missing facts stay `UNKNOWN / NOT ESTABLISHED`; an active freeze does not prove `RC = YES` or which hard exception applied; explicit facts outrank adjacent uncertainty | `SKILL.md` "Anti-inference rule", DEVELOPMENT_RECORD.md "anti-inference discipline" | ✓ |
| 25 | Aggregate vs component facts | "No hard exception applies" → `NONE APPLICABLE` only; component predicates stay `UNKNOWN`; an applicable hard exception does not identify which one | `SKILL.md` "Aggregate facts do not establish their component facts", `evidence-model.md` "Aggregate conclusion vs. component facts" | ✓ |
| 26 | Active-L4 status-review discipline | Established run-state facts are not expanded into unestablished execution details; `fresh L4 restart = NOT TRIGGERED` justified only by "no contamination event has occurred" | `SKILL.md` "Active-L4 execution / status review", invariant 24 | ✓ |
| 27 | Aggregate L4 completion | Aggregate run completion is separate from individual scenario results; `REAL SANDBOX PASS` only after every required scenario completes under intact integrity | `freeze-protocol.md` "Exit" | ✓ |
| 28 | Release-authorization separation | A positive validation result never yields "promote", "release approved", or "deploy" unless release authorization is explicitly in scope and every gate is established | `SKILL.md` "Validation sufficiency vs. release authorization", DEVELOPMENT_RECORD.md "release-authorization separation" | ✓ |

---

## Behaviors the scenarios exercised

Iterating the 42 scenarios exercised and settled the following behaviors; each is
now enforced by a rule in `skill/` (see [`DEVELOPMENT_RECORD.md`](DEVELOPMENT_RECORD.md)):

- stale / current L3 evidence handling
- partial historical L4 coverage handling
- subset vs complete zero-blast-radius handling
- evidence substitution fail-closed behavior
- scenario scoping
- freeze / contamination / restart separation
- anti-inference discipline
- release-authorization separation

## Cross-file consistency

A consistency review across the skill files (`SKILL.md`,
`references/validation-ladder.md`, `references/evidence-model.md`,
`references/freeze-protocol.md`, `references/reporting-rules.md`) found no
contradiction, regression, unsupported inference rule, or cross-file conflict at
the time of review.

## Boundary

- The 42 scenarios are not a fixed suite. More should be added only when a real
  project introduces a new payment-validation pattern, a new provider behavior,
  an architecture change, or a discovered regression.
- This page describes the current skill revision only.
- Any future skill edit should be re-checked against the affected scenarios
  before this page is relied on for the edited state.
