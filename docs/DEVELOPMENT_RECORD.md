# Development Record

## Status

Internal engineering development record for the current skill revision. Not an
acceptance sign-off, not an external certification, and not a statement that the
skill has been run against a real payment codebase.

## Scope

The `saas-payment-validation` skill — [`../skill/SKILL.md`](../skill/SKILL.md)
and [`../skill/references/`](../skill/references/) — was developed by iterating
against **42 hand-constructed scenarios** (Tests 1–42), each written to probe a
specific distinction the rules need to hold. This record describes what that
process covered for the **current skill revision**; the scenarios are summarized
in [`TEST_MATRIX.md`](TEST_MATRIX.md).

## What the scenarios exercised

Iterating the 42 scenarios exercised and settled the following behaviors; each is
now enforced by a rule in `skill/` (see [`TEST_MATRIX.md`](TEST_MATRIX.md) for the
category-by-category breakdown):

- stale / current L3 evidence handling
- partial historical L4 coverage handling
- subset vs complete zero-blast-radius handling
- evidence substitution fail-closed behavior
- scenario scoping
- freeze / contamination / restart separation
- anti-inference discipline
- release-authorization separation

## Cross-file consistency

A consistency review was performed across the skill files:

- `SKILL.md`
- `references/validation-ladder.md`
- `references/evidence-model.md`
- `references/freeze-protocol.md`
- `references/reporting-rules.md`

At the time of review, no contradiction, regression, unsupported inference rule,
or cross-file conflict was found. The files consistently preserve every checked
boundary, including:

- L1–L4 responsibility boundaries;
- application-owned timing vs provider-owned real-time classification;
- the RC gate vs hard exceptions, and the non-waivability of hard exceptions by
  evidence substitution;
- L3 classification state vs execution state;
- historical evidence validity vs current evidence freshness;
- the evidence-substitution decision order (Step A → B → C);
- the three-state zero-blast-radius semantics (`YES` / `NO` / `NOT ESTABLISHED`);
- structural relevance vs configuration-delta impact;
- historical L4 coverage knowledge vs mere old-`PASS` existence;
- the rule that subset coverage / subset zero-blast evidence never becomes
  full-decision evidence;
- scenario-scope anti-inference;
- active-L4 scenario states (`PASS` / `FAIL` / `PENDING`), with aggregate
  completion separate from individual scenario results;
- `BLOCKED` only when required validation cannot continue;
- scenario failure as separate from contamination;
- the proposed → deployed → contaminated → restart ordering;
- current affected L4 run vs fresh restart requirement;
- contamination-restart hard-exception semantics;
- contamination cause not automatically determining restart mechanics;
- baseline requirement vs operational action;
  `deployment verification required` ≠ `redeployment required`;
- provider-side-only contamination retaining the same authoritative repository
  revision;
- fresh entity OR authoritative restoration; a restored entity not restoring
  contaminated run evidence;
- active-L4 run-state facts not creating unestablished execution details;
- `REAL SANDBOX PASS` only after every required scenario completes under intact
  integrity;
- validation verdict not authorizing release / promotion / deployment;
- missing facts remaining `UNKNOWN / NOT ESTABLISHED` rather than being inferred.

## Validated invariants

The major invariants currently enforced by the skill. This list restates
existing rules; it introduces none.

1. **Minimum sufficient validation.** Choose the least validation that still
   proves correctness. L1–L3 are the normal engineering loop; L4 is a
   release-candidate real-time evidence mechanism, not the default "stronger
   test". Higher levels are not automatically better.
2. **L4 gate.** Level 4 is considered only when a payment-capable,
   production-intended Release Candidate is actually being promoted, or an
   applicable hard exception exists. "RC + timing-sensitive" is never by itself
   sufficient.
3. **Ordered L4 classification sequence.** Step A (RC / hard-exception gate) →
   Step B (real-time-only ownership) → Step C (evidence substitution). Stop at
   the first step that resolves L4. If Step A resolves as `NO`,
   `fresh L4 = NOT REQUIRED` and Step B is `NOT REACHED` — never reported as
   `L4 = UNKNOWN` or a combined `L4 = UNKNOWN / NOT REQUIRED`.
4. **Real-time-only is defined by behavioral ownership.** Application-owned
   timing with an injectable / deterministically controllable clock stays
   L1/L2-testable even at RC time. Only a provider-owned schedule, whose actual
   elapsed-time behavior is the thing under test, is real-time-only.
   Unestablished ownership is `UNKNOWN`; fail closed until resolved.
5. **Hard exceptions.** First real paid production launch; a provider migration
   that changes real lifecycle / timing behavior; a required restart of a
   previously `CONTAMINATED` L4 run. Each forces a fresh L4 run and is not
   eligible for evidence substitution.
6. **Hard exceptions force L4 but do not define its scenarios.** The exact L4
   scenario set is derived only from authoritative architecture / provider
   evidence. If not established: `Level 4 scenario scope = UNKNOWN / TO BE
   DEFINED`, and no `REAL SANDBOX PASS` is issued.
7. **Evidence substitution conditions.** A fresh L4 run may be skipped only when
   all hold: current L2 evidence covers the changed application-owned timing; the
   provider-boundary evidence is valid (including fresh L3 where required); a
   prior `REAL SANDBOX PASS` exists; and zero blast radius into the previously
   validated real-time-only behavior is authoritatively and demonstrably
   established. Otherwise fresh L4 is `REQUIRED` — fail closed.
8. **Zero blast radius is a three-state fact.** `YES` / `NO` / `NOT ESTABLISHED`.
   `NO` and `NOT ESTABLISHED` both fail the reuse condition closed but are
   distinct factual states; `NOT ESTABLISHED` is never reported as demonstrated
   positive impact.
9. **Structural relevance is not delta impact.** A configuration participating
   in / governing / configuring a covered behavior does not establish that a
   *change* to it affected that behavior. When impact is `NOT ESTABLISHED`,
   neutral wording is required, not "affected / impacted / touched / changed by".
10. **Evidence substitution is decision / revision / state scoped.** A successful
    Step C result resolves only the current decision for the current RC revision
    / state. Every future relevant RC or hard-exception decision re-enters at
    Step A.
11. **L3 classification state vs execution state are separate.** An explicitly
    established provider-wire-boundary change is `L3 = REQUIRED` (settled),
    reported `L3 = REQUIRED — NOT RUN / EVIDENCE MISSING` when unexecuted, and
    never downgraded to `L3 = UNKNOWN`. `L3 = UNKNOWN` is only for genuinely
    unresolved boundary ownership, which never overrides an established explicit
    change.
12. **Provider wire boundary (L3 scope)** includes semantic interpretation of
    real provider events / payloads into internal lifecycle state — not only the
    outbound call surface.
13. **Required evidence vs observed evidence.** No PASS is issued from static
    inspection, existing tests, checklists, documentation, a mentioned test
    command, or simulated time. The verdict vocabulary (`IMPLEMENTATION PASS` /
    `PROVIDER INTEGRATION PASS` / `REAL SANDBOX PASS` / `CONTAMINATED`) is used
    only for executed evidence. L3 is never treated as L4; simulated time is
    never called real-time validation.
14. **Mid-freeze temporal ordering.** Proposed change → deployed change →
    contamination → restart requirement are ordered, distinct states. Each
    carries its own evidence-substitution status. A later state and its
    consequences are not reported before its triggering event has actually
    occurred.
15. **Active freeze: "L4" is two fields.** `current L4 run status` and
    `fresh L4 restart` are reported separately — never a generic
    `L4 = REQUIRED`.
16. **Scenario outcome, freeze integrity, and operational run state are three
    separate facts.** A `scenario FAIL` under intact integrity is genuine L4
    evidence of a failure: not contamination, does not by itself cause
    `BLOCKED`, and does not by itself require a mid-freeze fix. `BLOCKED` is
    reported only when continuation of required validation is actually prevented.
17. **Freeze integrity is decided only from the four integrity checks**
    (revision, deployment, change, provider-state). Contamination requires an
    actual integrity-breaking event and is not limited to a deployed code fix —
    provider-side scenario-state mutation outside the validation plan also
    contaminates.
18. **Contamination determines only** that `fresh L4 restart = REQUIRED`, the
    contamination-restart hard exception is `ACTIVE`, and
    `Evidence substitution = DOES NOT APPLY`. It does not by itself determine
    restart mechanics.
19. **Restart mechanics follow the actual contamination cause, not the verdict.**
    Baseline constraints are separate from operational actions.
    `deployment verification required` ≠ `redeployment required`.
    Provider-side-only contamination may keep the same authoritative repository
    revision (`UNCHANGED / STILL AUTHORITATIVE`).
20. **Contaminated / affected test entity handling.** Use a fresh disposable
    entity, OR an authoritative restoration procedure that proves the required
    starting state and all integrity checks; fail closed to a fresh entity only
    when restoration validity is not established. No contamination cause makes a
    fresh entity universally mandatory, and no "burned forever" rule is asserted,
    unless authoritative policy says so. A restored entity does not by itself
    restore a contaminated run's evidence.
21. **Proposed-fix evidence isolation.** An undeployed proposed fix's L1/L2/L3
    states gate only that fix. They never attach to the current frozen run's
    observed evidence or verdict.
22. **A completed clean freeze is closed historical evidence** for its pinned
    revision. A later proposed / undeployed fix does not reopen, contaminate, or
    invalidate it. Any future fresh-L4 question belongs to the future (post-fix)
    evidence subject and is triggered and classified independently.
23. **Anti-inference discipline.** Missing facts stay `UNKNOWN / NOT
    ESTABLISHED`. An active L4 freeze does not prove `RC = YES` or which hard
    exception applied. An aggregate conclusion is not decomposed into component
    predicates, and an applicable hard exception does not identify which one
    applies. Explicit facts outrank adjacent uncertainty.
24. **Active-L4 status-review discipline.** Established run-state facts are not
    expanded into unestablished execution details. `fresh L4 restart = NOT
    TRIGGERED` is justified only by "no contamination event has occurred".
25. **Validation verdict vs release authorization are kept separate.** A positive
    validation result never yields "promote the RC", "release approved", or
    "deploy now" unless release authorization is explicitly in scope and every
    authoritative release gate is established.

## Boundary

- The 42 scenarios are not a fixed suite. More should be added only when a real
  project introduces a new payment-validation pattern, a new provider behavior,
  an architecture change, or a discovered regression.
- This record describes the **current skill revision only**.
- Any future skill edit should be re-checked against the affected scenarios
  before this record is relied on for the edited state.

## Files covered

- `skill/SKILL.md`
- `skill/references/validation-ladder.md`
- `skill/references/evidence-model.md`
- `skill/references/freeze-protocol.md`
- `skill/references/reporting-rules.md`

## Basis

The rules were developed by working the scenarios by hand against a small
synthetic payment codebase used purely as reading material for the decision
procedure. No real application source code, provider, credential, or customer
data was involved. There is no automated test harness in this repository, and the
skill has not been run against a real payment codebase — this record is
documentation only.

## Explicit non-claims

- This is **not** an external certification.
- This is **not** production release approval.
- This does **not** assert that any application is production-ready.
- This does **not** assert that the skill has been exercised on a real payment
  system.
