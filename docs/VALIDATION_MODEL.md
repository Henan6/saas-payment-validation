# Validation Model

An engineering design document for the payment-validation framework. It explains
what the model is trying to protect, how the pieces fit together, and why each
distinction is drawn where it is. The normative rules live in
[`../skill/SKILL.md`](../skill/SKILL.md) and
[`../skill/references/`](../skill/references/); this document is the rationale.

---

## 1. The problem being modelled

A payment change can be "tested" in many ways that all feel adequate and none of
which actually cover the failure. The model exists to make the *kind* of evidence
explicit, so that:

- a change gets the evidence it needs and no more;
- a strong-looking test is never silently accepted for a claim it does not
  support;
- a long-running real-time validation stays trustworthy;
- a validation result is never mistaken for a release decision.

The core move is to separate **required evidence** (what should be run) from
**observed evidence** (what actually ran and produced a result), and to forbid
filling any gap by inference from a state that merely looks compatible.

---

## 2. The four levels

### L1 — deterministic / local implementation validation

Deterministic behavior that needs no database, no provider, no network, and no
real elapsed time: state machines, lifecycle mapping, entitlement derivation,
payload parsing, duplicate-event handling, invalid-transition rejection, and
timestamp-boundary logic driven by a controlled clock. Every behavior-affecting
payment change normally gets relevant L1 coverage. No freeze.

### L2 — real integration validation

Correctness that depends on real application integration: persistence, database
constraints, transactions, concurrency, ordering, conflict resolution,
idempotency under concurrent processing, reconciliation races. Uses a real
isolated test database or equivalent. The provider boundary stays fake.
Application-owned time may be fast-forwarded with a controlled clock — you do not
wait real hours to test logic the application owns.

### L3 — real provider sandbox / wire-boundary validation

Used when the real provider boundary changed or must be re-proven. The
**provider wire boundary** is not only the outbound call surface; it includes:

- provider SDK / API calls and outbound request shape;
- webhook signature / authenticity handling;
- real provider payload schema interpretation;
- **provider event type → internal lifecycle-state mapping**;
- semantic interpretation of provider webhook fields that changes system
  behavior.

A change to how a real provider event is interpreted into internal lifecycle
state is a provider-wire-boundary change *even when no outbound call changed*.
L3 runs against a real provider sandbox, is short-lived, and pins the revision
under test.

`PROVIDER INTEGRATION PASS` proves the wire boundary. It does **not** prove real
grace expiry, real retry cadence, real provider-driven recovery timing, real
renewal timing, or real cancellation effective-date timing. **L3 is not L4.**

### L4 — frozen real-time validation

Proves behavior that cannot be established without **real elapsed time against a
real payment provider** — provider-owned schedules and lifecycle boundaries. It
is a release-validation mechanism, not a development tier. It requires a pinned
revision, a deployed and verified frozen environment, disposable test entities,
an explicitly defined scenario set, and an intact freeze for the whole
observation window.

---

## 3. The RC gate

Level 4 is *considered* only when one of these holds:

- a payment-capable, **production-intended Release Candidate** is actually being
  promoted; or
- a **hard exception** applies.

Reaching the gate does not make L4 required. "RC + timing-sensitive change" is
explicitly **not** sufficient. The RC is only what permits L4 to be evaluated;
after the gate, L4 is required only if a genuine real-time-only evidence gap
remains.

An explicit "no RC promotion is being made" is an established `NO` for the
current decision — not `UNKNOWN` — even when broader release chronology (for
example first-paid-launch status) is still unknown. Those are different facts and
are reported separately.

---

## 4. Hard exceptions

A fresh L4 run is required, regardless of per-behavior real-time-only
classification, for:

- the **first real paid production launch**;
- a **provider migration** that changes real lifecycle / timing behavior;
- a **required restart of a previously `CONTAMINATED` L4 run**.

Hard exceptions are **not eligible for evidence substitution**.

Two guards apply:

- **Aggregate ≠ components.** "No hard exception applies" establishes only
  `hard exception applicability = NONE APPLICABLE`. It does not establish
  `first paid launch = NO`, `provider migration = NO`, or
  `prior contamination restart = NO`; each stays `UNKNOWN` unless independently
  established.
- **Applicable ≠ identified.** An established *applicable* hard exception does not
  say *which* one applies; identity stays `UNKNOWN / NOT ESTABLISHED` unless
  explicitly established.

Hard exceptions force L4 but **do not define its scenarios** (see §8).

---

## 5. The Level 4 classification sequence (Step A → B → C)

Applied in order; stop at the first step that resolves L4.

**Step A — RC / hard-exception gate.**
No → `fresh L4 = NOT REQUIRED`; Step B is `NOT REACHED`. Report those as separate
facts — never `L4 = UNKNOWN`, never a combined `L4 = UNKNOWN / NOT REQUIRED`. If
real-time ownership was never classified because Step B was not reached, say
`real-time-only classification = NOT REACHED` separately. Yes → Step B.

**Step B — real-time-only classification**, decided per changed behavior by
*behavioral ownership*:

- **Application-owned**, clock injectable or deterministically controllable
  (grace expiry from an injected `now`, a retry scheduler with a controllable
  clock) → **not** real-time-only; stays L1/L2-testable even at RC time.
- **Provider-owned**, where the provider's actual schedule is the thing under
  test (dunning cadence, provider-driven recovery on the provider's real
  schedule, a real billing-period renewal or cancellation boundary) →
  real-time-only.
- **Not established** which → `real-time-only = UNKNOWN`, `L4 = UNKNOWN`; resolve
  ownership first; fail closed (no PASS).

All changed behavior not real-time-only → L4 NOT REQUIRED, stop. Real-time-only
`YES` → Step C.

**Step C — evidence substitution** (see §6). Evaluated *only* when Step B = YES.

---

## 6. Evidence substitution

At Release Candidate time a fresh L4 run may be skipped **only when all** of these
are demonstrable:

1. current L2 evidence covers the timing-sensitive application behavior changed
   since the previous valid L4 run;
2. current provider-boundary evidence is valid, including fresh L3 where required;
3. a previous `REAL SANDBOX PASS` exists;
4. the difference between the previously validated revision and the new Release
   Candidate has **zero blast radius** into the real-time-sensitive behavior that
   prior L4 evidence covered.

If any condition fails — in particular if zero blast radius cannot be established
confidently — `Evidence substitution = DOES NOT APPLY` and `fresh L4 = REQUIRED`.
Fail closed.

### Zero blast radius is a three-state fact

| State | Meaning | Reuse condition |
|---|---|---|
| `YES` | affirmatively demonstrated | satisfied |
| `NO` | positive blast radius demonstrated | fails — demonstrated impact |
| `NOT ESTABLISHED` | insufficient evidence either way | fails closed — missing reuse evidence |

`NO` and `NOT ESTABLISHED` both lead to `DOES NOT APPLY` / `fresh L4 = REQUIRED`,
but they are different factual states. `NOT ESTABLISHED` must never be reported as
`affected / in blast-radius path = YES` or as demonstrated impact.

### Structural relevance is not delta impact

A provider-side configuration can *participate in*, *govern*, or *configure* a
provider-owned behavior without a *change* to it being shown to alter that
behavior. `configuration participates in the behavior = YES` and
`configuration delta has positive blast radius = YES` are different claims. When
impact is not established, use neutral wording ("behavior associated with the
changed configuration"), not "affected / impacted / touched / changed by".

### The method for establishing zero blast radius is not prescribed

Condition 4 is a *result* to be authoritatively demonstrated, not a mandate for
one specific mechanism. An independent blast-radius review may be one valid
method and may be recommended when the task raises it — it is not the only path,
unless authoritative policy says so.

### Substitution is decision / revision / state scoped

`Evidence substitution = APPLIES` resolves **this** decision for **this** RC
revision / state only. It is not a standing L4 exemption. Every future relevant
payment-capable RC or applicable hard-exception decision re-enters the sequence
at Step A, using the evidence applicable at that time. Today's zero-blast-radius
demonstration may be cited as historical evidence, but reuse eligibility must be
re-established; a later application change can invalidate its relevance even when
provider identity and provider-owned timing are unchanged.

### Status reporting

- Step A or Step B resolved L4 → `Evidence substitution = NOT RELEVANT`.
- `APPLIES` / `DOES NOT APPLY` only when Step B = YES (Step C was actually
  reached).
- Mid-freeze exception: once an actual contamination event has occurred, the
  affected run is `CONTAMINATED`, the contamination-restart hard exception is
  `ACTIVE`, and `Evidence substitution = DOES NOT APPLY` — not `NOT RELEVANT`.

---

## 7. Provider-owned real-time behavior

"Real-time-only" is defined by ownership, not by the phrase "timing-sensitive".

**Real-time-only** — the provider owns the schedule and its actual elapsed-time
behavior is what must be proven: provider retry / dunning schedule; naturally
delayed or retried provider events; provider-driven recovery on the provider's
real schedule; a grace window whose expiry the provider enforces on real elapsed
time; cancellation or renewal effective at a real provider-controlled
billing-period boundary; correctness that depends on multiple real
provider-scheduled processes racing over elapsed time.

**Not real-time-only** — the application owns the clock and it can be injected or
advanced deterministically: grace expiry computed from an injected `now`; an
application retry / recovery scheduler with a controllable test clock; any
deterministic timestamp-boundary logic; application state transitions on a
fast-forwarded clock.

Application-owned timing is not reclassified as provider-driven merely because a
Release Candidate is in play. When ownership is not established in the repository
or prompt, `real-time-only = UNKNOWN`; resolve it before deciding L4.

---

## 8. Scenario scoping

`Level 4 REQUIRED` does not prove that any particular provider-owned lifecycle
behavior exists. Dunning cadence, provider-driven recovery timing, provider-owned
grace expiry, provider-controlled renewal or cancellation timing — none may be
assumed. Each must be shown to exist from the actual payment architecture, the
provider contract, documented provider behavior, or other authoritative evidence.

Two separate questions:

- **Is L4 required?** — answered by the Step A → B → C sequence and the hard
  exceptions.
- **Which exact L4 scenarios are required?** — answered only from authoritative
  architecture / provider evidence.

If the scenario set is not established: `Level 4 scenario scope = UNKNOWN / TO BE
DEFINED`, the freeze must not start, and no `REAL SANDBOX PASS` may be issued.
This holds even for a hard exception such as first paid launch: the exception
forces the run, not the scenario list.

When reuse failed via `zero blast radius = NOT ESTABLISHED`, the required
scenario is *not* "the scenario the change affected" — positive impact was not
established. The set is derived authoritatively and must account for the
previously validated provider-owned behavior *associated with* the changed
configuration.

---

## 9. Evidence freshness

Every evidence state belongs to a specific subject — a specific revision or a
specific run — and does not transfer across subjects.

- **Classification state vs execution state.** An explicitly established
  provider-wire-boundary change is `L3 = REQUIRED` — settled. If no fresh
  provider-sandbox run exists for the revision under review, that is
  `L3 = REQUIRED — NOT RUN / EVIDENCE MISSING`, never `L3 = UNKNOWN`. `UNKNOWN` is
  only for genuinely unresolved boundary ownership (whether the boundary changed
  at all is not established).
- **Historical vs current.** A completed clean freeze remains valid historical
  evidence for its pinned revision. It does not cover a later revision. An old
  `PASS` existing is not the same as *known historical coverage* of the behavior
  a new decision needs.
- **Proposed-fix evidence isolation.** During an active freeze, an undeployed
  proposed fix's L1/L2/L3 states gate only that fix. They never attach to the
  current frozen run's observed evidence or verdict. The current run's inability
  to issue `REAL SANDBOX PASS` is explained by the required scenario result
  alone.

---

## 10. Freeze integrity

A long real-time validation is meaningful only if the observed system does not
silently change underneath it. Before the first scenario action: prerequisite
L1–L3 evidence complete; the scenario set explicitly defined and justified; an
exact revision pinned; the environment deployed from it and the deployed revision
verified; disposable test entities prepared.

A clean L4 result requires **all four** integrity checks throughout the window:

1. **Revision integrity** — the intended frozen revision is known.
2. **Deployment integrity** — the environment is proven to be running that
   revision.
3. **Change integrity** — no unauthorized behavior-affecting change entered the
   environment during observation.
4. **Provider-state integrity** — provider-side state relevant to the scenario
   was not manually changed outside the validation plan.

If any required condition can no longer be established: `CONTAMINATED`.

---

## 11. Contamination

Contamination is decided **only** from the four integrity checks. It is a
separate fact from scenario outcome and from operational run state:

- **Scenario outcome** — did the executed scenario produce its predefined
  expected result? A `scenario FAIL` where the procedure ran to plan and all four
  integrity conditions still hold is a genuine product/provider result and valid
  L4 evidence of a failure. It is **not** contamination and does not by itself
  trigger a restart.
- **Freeze integrity** — `NOT CONTAMINATED` while all four checks hold;
  `CONTAMINATED` when one can no longer be established.
- **Operational run state** — `BLOCKED` only when continuation of required
  validation is actually prevented (established by prompt or evidence); otherwise
  `run continuation status = UNKNOWN / NOT ESTABLISHED`. Not inferred from a
  single scenario failure.

A contaminating event may be any actual integrity break: a deployed code/config
change entering the frozen environment; provider-side scenario-state mutation
outside the plan (or a predefined provider action run outside its allowed timing
window); deployment or revision integrity loss; an unauthorized behavior-
affecting change; any other required condition becoming unestablishable.
Contamination is **not** limited to "a later code fix".

**Temporal ordering.** Proposed change → deployed change → contamination →
restart requirement are ordered, distinct states. A later state and its
consequences (`CONTAMINATED`, `fresh L4 restart = REQUIRED`,
`Evidence substitution = DOES NOT APPLY`) are not reported before the triggering
event has actually occurred. Before deployment of a blast-radius-uncertain fix:
`current L4 run status = ACTIVE / IN PROGRESS` (operationally `BLOCKED` if
established), `fresh L4 restart = CONDITIONAL / NOT YET TRIGGERED`,
`Evidence substitution = NOT RELEVANT`.

During an active freeze, "L4" is two fields, never a generic `L4 = REQUIRED`:

- `current L4 run status` — `ACTIVE / IN PROGRESS` / `BLOCKED` / `CONTAMINATED` /
  `COMPLETED CLEANLY`;
- `fresh L4 restart` — `NOT TRIGGERED` / `CONDITIONAL / NOT YET TRIGGERED` /
  `REQUIRED`.

---

## 12. Restart mechanics

A `CONTAMINATED` run restarts the affected real-time validation from a **clean,
explicitly re-established frozen baseline**. The `CONTAMINATED` verdict alone
determines only:

- `fresh L4 restart = REQUIRED`;
- the contamination-restart hard exception is `ACTIVE`;
- `Evidence substitution = DOES NOT APPLY`.

It does **not** by itself determine whether a new commit, a different repository
revision, a code or config change, or a redeploy is needed. Those follow the
**actual contamination cause**:

**Code / config contamination** (a code/config change deployed into the frozen
environment, or unresolved blast radius): the restart baseline must reflect the
authoritative corrected revision/config; establish revision integrity; verify
deployment integrity against that baseline. A different repository revision is
required only if the correction actually requires one. A redeploy is required
only if it is necessary to make the deployed environment match the authoritative
corrected state — otherwise `redeployment requirement = UNKNOWN / NOT
ESTABLISHED`. Never "code/config contamination → redeploy" as a reflex.

> `deployment verification required` ≠ `redeployment required`.

**Provider-side-only contamination** (provider-side scenario state mutated, or a
predefined provider action ran outside its timing window; no repository, deployed
revision, config, or deployment change): the same repository revision may remain
authoritative — state that explicitly
(`repository revision = UNCHANGED / STILL AUTHORITATIVE`) rather than inventing a
new-revision requirement. Redeployment is not automatically required.
Re-establish clean provider-side scenario state and re-verify all four integrity
checks before restart.

**Contaminated / affected test entity.** For the affected scenario's entity: use
a fresh disposable entity, **or** an explicitly allowed authoritative restoration
procedure that can prove the required starting state and all integrity checks. If
restoration validity is not established, fail closed to a fresh entity. A fresh
entity may be the practical default, but no contamination cause makes it
*universally* mandatory unless authoritative policy says so, and no "burned
forever" rule is asserted without such policy. A restored entity does not by
itself restore a contaminated run's evidence — the affected validation is still
restarted from a clean baseline.

**Re-classify the fix** (only when contamination involved an actual code/config
fix): send it back through normal L1–L3 classification before re-entry. For
webhook / lifecycle-mapping fixes, if provider payload interpretation may have
changed but this is not established → `L3 = UNKNOWN`; classify before re-entry;
no PASS while `L3 = UNKNOWN`.

---

## 13. The release-authorization boundary

A payment-validation answer establishes only the validation state that was asked
for. `Evidence substitution = APPLIES`, `fresh L4 = NOT REQUIRED`, and
`L1 / L2 / L3 PASS` do **not**, alone or together, establish "release approved",
"RC promotion authorized", "deployment authorized", or any other go/no-go
disposition.

A release disposition may be stated only when **both**:

1. release disposition is explicitly in scope for the task; and
2. the authoritative release policy and every required release gate (business /
   operational / deployment / release-management approval, plus any other
   prerequisite) are established.

Otherwise the answer stops at the validation conclusion — for example: "for the
payment-validation question presented, the Level 4 requirement is satisfied
through valid evidence substitution, so a fresh L4 run is not required for this
decision." No imperative "promote", "ship", or "deploy" that is sourced only from
validation evidence.

**Validation verdict ≠ release authorization.**

---

## 14. Verdict vocabulary

Used only for executed evidence:

| Verdict | Meaning |
|---|---|
| `IMPLEMENTATION PASS` | required L1/L2 implementation evidence actually passed |
| `PROVIDER INTEGRATION PASS` | a real-provider short run actually demonstrated current provider wire compatibility |
| `REAL SANDBOX PASS` | required real-time lifecycle scenarios actually completed under a valid L4 freeze |
| `CONTAMINATED` | L4 integrity was lost; the affected observation cannot be represented as a clean pass |

When evidence has not actually been established, the vocabulary is `REQUIRED` /
`NOT REQUIRED` / `UNKNOWN` / `NOT RUN` / `EVIDENCE MISSING` / `BLOCKED`. L3 is
never treated as L4. Simulated time is never called real-time validation.
