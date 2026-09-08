# Validation Ladder

## L1 — Local / Automated Validation

Use L1 for deterministic payment behavior that can be proven without:

- a real database;
- a real payment provider;
- network access;
- real elapsed time.

Typical coverage:

- state-machine behavior;
- entitlement decisions;
- lifecycle mapping;
- duplicate-event behavior;
- parsing;
- failure handling;
- deterministic timestamp-boundary logic using controlled time;
- price/plan resolution;
- invalid transition rejection.

Every behavior-affecting payment change should normally receive relevant L1 coverage.

L1 does not require a freeze.

---

## L2 — Provider-Independent Integration Validation

Use L2 when correctness depends on real application integration such as:

- persistence;
- database constraints;
- transactions;
- concurrency;
- ordering;
- conflict resolution;
- idempotency under concurrent processing;
- reconciliation races.

Use a real isolated test database or equivalent integration environment.

The payment-provider boundary remains fake.

Time-sensitive application behavior may be fast-forwarded using controlled test state or controlled time.

Do not wait hours or days merely to test application-owned timing logic.

L2 does not require a real provider or a freeze.

---

## Controlled/injected time is valid evidence for application-owned timing

Controlled, injected, or fast-forwarded time remains valid L1/L2 evidence for
timing logic that the application owns — the clock it reads can be supplied or
advanced deterministically in a test.

Promotion to Release Candidate does not change this. Reaching the RC gate does
NOT transform deterministic application-owned timing (for example grace-period
expiration computed from an injected `now`, or a retry/recovery scheduler with a
controllable clock) into real-time-only behavior. At RC time such behavior is
still proven at L1/L2.

Real elapsed time is required only for behavior the provider owns, where the
provider's actual schedule is the thing under test and no provider
time-simulation facility can reproduce it (see `references/evidence-model.md`).

---

## L3 — Provider Sandbox Integration Validation

Use L3 when the real provider boundary itself changed or must be re-proven.

Examples:

- provider SDK/API calls;
- checkout/provider request compatibility;
- webhook authenticity/signature handling;
- provider payload parsing;
- real provider payload schema interpretation;
- provider event type → internal lifecycle-state mapping;
- semantic interpretation of provider webhook fields used by subscription
  lifecycle logic;
- provider identifiers;
- subscription create/update/cancel API compatibility;
- customer portal integration;
- reconciliation provider API compatibility.

L3 uses a real provider sandbox when available.

It should normally be short-lived.

Pin or record the revision used for the run and do not change the tested environment during that run.

### L3 decision table

| Established fact about the change | L3 classification |
|---|---|
| Provider event type → internal lifecycle-state mapping / interpretation changed | **L3 REQUIRED** |
| Real provider payload schema interpretation changed | **L3 REQUIRED** |
| Semantics of provider webhook fields used by subscription lifecycle logic changed | **L3 REQUIRED** |
| Provider SDK / API calls, request shape, identifiers, or signature / authenticity handling changed | **L3 REQUIRED** |
| Only downstream internal state → entitlement / business-rule behavior changed, provider parsing / interpretation unchanged | **L3 NOT REQUIRED** |
| It is not established which of the above occurred (could be purely internal downstream mapping, could be provider-payload interpretation) | **L3 UNKNOWN** |

Once the first group of cases is **explicitly established**, do not reopen it as
`UNKNOWN` on the strength of a hypothetical internal-only alternative. An
explicitly established provider-boundary change is `L3 REQUIRED`, full stop.

### L3 REQUIRED when

- a real provider event or payload is interpreted differently;
- a provider webhook event type → internal lifecycle-state mapping changes;
- the semantics of provider webhook fields used by subscription lifecycle logic
  change;
- provider SDK/API calls, request shape, identifiers, or signature / authenticity
  handling change.

When any of these is explicitly established by the task, repository, or
authoritative evidence, `L3 = REQUIRED` is settled. If no fresh provider-sandbox
evidence has been run for the revision under review, report the execution state
alongside the classification:

`L3 = REQUIRED — NOT RUN / EVIDENCE MISSING`

Missing execution evidence against a `REQUIRED` classification is never
`L3 = UNKNOWN` (see `references/evidence-model.md`, "Evidence-authority rule for
L3: classification state and execution state are separate").

### L3 NOT REQUIRED when

- only downstream internal state → entitlement / business-rule mapping changes;
- provider payload parsing and its semantic interpretation are unchanged;
- the change stays inside application-owned logic behind an unchanged provider
  boundary.

### L3 UNKNOWN when

- a webhook / lifecycle-mapping change could be either purely internal downstream
  mapping or part of provider-payload interpretation, and the evidence does not
  establish which.

Then do not default to NOT REQUIRED. Inspect and classify the provider-boundary
impact — including semantic payload interpretation — before deciding. Fail
closed: no PASS while L3 is UNKNOWN.

`L3 = UNKNOWN` is only for this genuinely unresolved case — whether the provider
boundary changed at all is not established. It does not apply once a
provider-boundary change has been explicitly established (that case is
`L3 = REQUIRED`).

### Important

`PROVIDER INTEGRATION PASS` proves the provider wire boundary.

It does NOT prove:

- real grace expiry;
- real retry cadence;
- real provider-driven recovery timing;
- real renewal timing;
- real cancellation effective-date timing.

L3 is not L4.

---

# Routing examples

## Reconciliation concurrency change

If provider API behavior is unchanged:

- L1 REQUIRED
- L2 REQUIRED
- L3 NOT REQUIRED
- L4 NOT REQUIRED during ordinary development

## Provider SDK change

If provider calls changed:

- L1 REQUIRED
- L3 REQUIRED
- L2 only if persistence/concurrency behavior is also affected
- L4 still waits for the Release Candidate gate

## Webhook lifecycle-mapping change

If a real provider event / payload is interpreted into internal lifecycle state
differently (provider event type → state mapping, or provider field semantics
used by lifecycle logic) — and this is explicitly established (CASE A):

- L1 REQUIRED
- L3 REQUIRED — settled; if no fresh provider-sandbox run exists, report
  `L3 = REQUIRED — NOT RUN / EVIDENCE MISSING`. Do not reopen this as
  `L3 = UNKNOWN` because an internal-only alternative could be hypothesised, and
  do not phrase it as "REQUIRED vs. UNKNOWN to be resolved". The only remaining
  L3 action is to run the provider-sandbox validation and record the evidence.
- L2 only if persistence/concurrency/ordering is also affected
- L4 still waits for the Release Candidate gate

If only the downstream internal state → entitlement mapping changed and provider
payload parsing / interpretation is unchanged:

- L1 REQUIRED
- L3 NOT REQUIRED

If the evidence does not establish which of the two applies (CASE B — genuinely
unresolved boundary ownership):

- L1 REQUIRED
- L3 UNKNOWN — classify the provider-boundary impact before deciding

## Grace-period logic change during normal development

Even though it is timing-sensitive:

- L1 REQUIRED
- L2 when integrated state/persistence behavior is involved
- L3 only if the provider boundary changed
- L4 NOT REQUIRED merely because timing changed

At Release Candidate time this is unchanged when the grace clock is
application-owned and injectable: L4 remains NOT REQUIRED for that behavior.
L4 is reconsidered only for a genuinely provider-owned real-elapsed-time gap
(see `references/evidence-model.md`).

---

# Evidence substitution / zero-blast-radius prerequisite states

When deciding whether prior Level 4 evidence can be reused for a new Release
Candidate, the zero-blast-radius prerequisite has three distinct states. Do not
collapse them:

| State | Meaning | Reuse condition |
|---|---|---|
| `YES` | zero blast radius affirmatively demonstrated | satisfied |
| `NO` | positive blast radius demonstrated | fails — demonstrated impact |
| `NOT ESTABLISHED` | insufficient evidence either way | fails closed — insufficient evidence |

Both `NO` and `NOT ESTABLISHED` lead to `Evidence substitution = DOES NOT APPLY`
and `fresh L4 = REQUIRED`, but they are not the same factual state:
`NOT ESTABLISHED` does not mean positive blast radius was proven, and must not be
reported as `affected / in blast-radius path = YES`.

The prerequisite is a result to be authoritatively demonstrated, not a mandate
for one exclusive review mechanism unless authoritative policy says so. An
independent blast-radius review may be one valid method (and may be recommended
when the task raises it), not the only path. See
`references/evidence-model.md`, "Evidence substitution", "Absence of proof is not
proof of the opposite", "Structural relevance vs. delta impact", and "Evidence
method for zero blast radius is not prescribed".

## Participation is not delta impact

Distinguish two separate facts:

- **A. relationship / participation** — a provider-side setting participates in,
  governs, or configures a covered provider-owned real-time behavior;
- **B. delta impact** — the effect of the *changed value* on that behavior.

A changed provider-side setting may participate in a covered real-time behavior
(A) without its delta impact (B) being established. If the effect of the changed
value on that behavior has not been authoritatively established:

- report `zero blast radius = NOT ESTABLISHED`;
- do **not** report `zero blast radius = NO`;
- do **not** describe the behavior as positively "affected by the change",
  "impacted by the change", or "touched by the change".

Reuse failure and the fresh L4 requirement still follow fail-closed from
`NOT ESTABLISHED`.

## Step C result is decision-scoped

When all substitution conditions are met, report:

- `Evidence substitution = APPLIES`;
- `fresh L4 = NOT REQUIRED FOR THIS DECISION`.

Do not phrase this as:

- "permanently L4-exempt";
- "future L4 unnecessary unless a fixed list of events occurs";
- "release authorized" / "promote the RC".

Any future payment-capable, production-intended Release Candidate or applicable
hard-exception decision re-enters the Level 4 classification sequence at Step A,
using the evidence applicable to that future revision / state. Today's
substitution decision and zero-blast-radius demonstration may be cited as
historical evidence, but reuse eligibility must be re-established for the future
decision. See `references/evidence-model.md`, "Substitution is decision-scoped"
and "Validation sufficiency vs. release authorization".

---

# Failure handling

A failure at L1, L2, or L3 blocks progression until corrected.

If a failure is found before L4 begins:

- fix it;
- rerun the affected evidence level;
- do not classify the situation as freeze contamination.

Freeze contamination rules apply only after an L4 freeze has actually begun.