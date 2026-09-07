# Release Candidate and Level 4 Evidence Model

## Level 4 purpose

L4 proves behavior that cannot be fully established without real elapsed time against a real payment provider.

It is a release-validation mechanism, not a normal development test tier.

---

# Level 4 trigger

A fresh Level 4 run is normally required only when BOTH are true:

1. a payment-capable / production-intended Release Candidate is actually being promoted;

AND

2. the release needs real-time payment evidence for which no reusable valid previous Level 4 evidence exists.

If no Release Candidate is being promoted, ordinary development remains on L1-L3.

---

# Fresh Level 4 requirement vs. an already-active Level 4 run

`Fresh L4 required` means: **a new real-time validation run must be executed.**

It is **not** equivalent to: **an existing L4 run is currently active.**

For a mid-freeze decision:

- do not classify `L4 REQUIRED` unless you mean a fresh run is actually
  required;
- if the current run already exists, report the current-run status separately
  (`ACTIVE / IN PROGRESS`, `BLOCKED`, `CONTAMINATED`, or `COMPLETED CLEANLY`)
  from the fresh-restart status (`NOT TRIGGERED`,
  `CONDITIONAL / NOT YET TRIGGERED`, or `REQUIRED`);
- see `references/freeze-protocol.md`, "Current Level 4 run status vs. fresh
  Level 4 restart requirement".

Preserve the hard-exception rules:

- **after** an actual contamination event, fresh L4 = `REQUIRED` and the
  contamination-restart hard exception is `ACTIVE`;
- **before** any contamination event, do not activate the restart hard
  exception; fresh L4 restart = `CONDITIONAL / NOT YET TRIGGERED` while a
  contaminating action is only proposed.

---

# Evidence-authority rule: classification facts must be supported independently

Each classification fact must be supported by its own evidence, not reconstructed
from downstream evidence that happens to be consistent with it.

In particular:

- RC status and hard-exception status are **separate facts**. Establishing one
  does not establish the other.
- An active Level 4 run proves only that Level 4 was **entered**. It does not
  prove whether entry came from RC gating or from a hard exception, and it does
  not prove which hard exception (if any) applied.
- If RC status is not explicitly established by prompt, repository, authoritative
  evidence, or explicit task context:
  `RC status = UNKNOWN / NOT ESTABLISHED`.
- If the prompt, repository, authoritative evidence, or explicit task context
  **explicitly establishes** that no payment-capable / production-intended RC
  promotion is being made for the current decision:
  `RC promotion status = NO`. An explicit `NO` is not downgraded to
  `UNKNOWN / NOT ESTABLISHED` because broader release chronology is unknown. Keep
  the current-decision RC promotion status separate from historical facts such as
  first-paid-launch chronology, which may independently remain `UNKNOWN`.
- If the hard-exception identity is not established:
  `hard exception = UNKNOWN / NOT ESTABLISHED`.

Do not reconstruct missing historical facts (release chronology, first-paid-launch
status, provider-migration status, prior-`REAL SANDBOX PASS` ordering) from the
mere existence of downstream evidence or the current validation state. Fail
closed: an unestablished fact stays `UNKNOWN / NOT ESTABLISHED` and is not filled
by implication.

## Aggregate conclusion vs. component facts

An established **aggregate** conclusion does not establish each component
predicate beneath it.

- `no hard exception applies` establishes only
  `hard exception applicability = NONE APPLICABLE`.
- It does **not** establish `first paid launch = NO`, `provider migration = NO`,
  or `prior contamination restart = NO`.
- Report each individual predicate as `UNKNOWN / NOT ESTABLISHED` unless it is
  independently established.

Conversely, an established *applicable* hard exception does not identify **which**
hard exception applies; report `hard exception identity = UNKNOWN / NOT
ESTABLISHED` unless the identity is explicitly established.

## Absence of proof is not proof of the opposite

`zero blast radius = NOT ESTABLISHED` is the absence of the reuse evidence that
evidence substitution requires. It is **not** proof that blast radius is
non-zero.

- Use `zero blast radius = NOT ESTABLISHED` — not `blast radius = YES`, and not
  `zero blast radius = NO` — unless affirmative impact evidence exists.
- `NOT ESTABLISHED` (insufficient evidence) and `NO` (demonstrated impact) are
  different factual states. Both make the reuse condition fail —
  `Evidence substitution = DOES NOT APPLY`, `fresh L4 = REQUIRED`, fail closed —
  but they must not be collapsed, and reaching `DOES NOT APPLY` via
  `NOT ESTABLISHED` does not authorize asserting positive impact.

## Structural relevance vs. delta impact

A configuration or component being structurally connected to a behavior is a
different fact from a *change* to that configuration having a demonstrated effect
on the behavior.

Example:

- Provider retry/dunning configuration participates in provider-owned scheduling
  behavior.
- The configuration value changed.

This establishes:

- `structural relevance = YES`;
- `configuration delta exists = YES`.

It does **not** establish:

- `behavioral impact of delta = YES`

unless affirmative impact evidence exists.

Therefore:

- `actual delta impact = NOT ESTABLISHED`
- `zero blast radius = NOT ESTABLISHED`

is a valid combination. Do not convert structural relevance — the configuration
participates in / governs / configures the behavior — into demonstrated blast
radius. Reuse still fails closed from `zero blast radius = NOT ESTABLISHED`
(`Evidence substitution = DOES NOT APPLY`, `fresh L4 = REQUIRED`); the failure is
for absence of reuse evidence, not for demonstrated impact. Do not describe the
covered behavior as "affected by", "impacted by", or "touched by" the change
while `actual delta impact = NOT ESTABLISHED` — use neutral wording such as
"behavior associated with the changed configuration".

---

# Evidence-authority rule for L3: classification state and execution state are separate

Whether L3 is **required** (classification) and whether L3 evidence has been
**run** (execution) are two different facts. Do not convert missing execution
evidence into classification uncertainty.

When the task, repository, or authoritative evidence **explicitly establishes**
that a provider wire-boundary behavior changed — for example:

- provider event type → internal lifecycle-state interpretation changed;
- real provider payload schema interpretation changed;
- behavior-affecting provider field semantics changed;
- authenticity / signature / API / SDK boundary behavior changed;

the classification is settled:

`L3 = REQUIRED`

If no fresh provider-sandbox evidence exists for the revision under review, the
**execution** state is:

`NOT RUN / EVIDENCE MISSING`

Report both together:

`L3 = REQUIRED — NOT RUN / EVIDENCE MISSING`

In particular:

`L3 REQUIRED + evidence missing` **≠** `L3 UNKNOWN`

Use `L3 = UNKNOWN` only when whether the provider boundary changed is itself
unknown (see `references/validation-ladder.md`, "L3 UNKNOWN when", and SKILL.md,
"L3 explicit-fact rule (strict)" — CASE B). Once the provider-boundary change is
explicitly established (CASE A), do not reopen `whether L3 is required` as a
question, and do not describe it as "REQUIRED unless someone argues it is
internal" or "REQUIRED vs. UNKNOWN to be resolved". The only remaining L3 action
is to run the required provider-sandbox validation and record the evidence.

This is the L3 instance of "Explicit facts outrank uncertainty": an explicit `NO`
or `YES` (here, an explicitly established provider-boundary change) is not
downgraded to `UNKNOWN` because a broader or adjacent fact — such as whether the
evidence has been executed — is still open.

---

# Evidence is revision / state scoped

Every evidence state belongs to a specific subject — a specific revision or a
specific run. Do not transfer an evidence state across subjects.

- Do **not** transfer an evidence state from a **proposed, undeployed fix** to
  the **current frozen revision**. A proposed fix's L1 / L2 / L3
  (`REQUIRED` / `NOT REQUIRED` / `UNKNOWN` / `PASS` / `FAIL`) gates only that
  fix; it neither blocks nor satisfies `REAL SANDBOX PASS` for the current run,
  and it does not change the current run's observed evidence or verdict.
- Do **not** transfer current-run L4 evidence to a **future post-fix revision**
  unless an explicit reuse / substitution rule authorizes it (see "Evidence
  substitution").

When reporting a mid-freeze decision, label evidence by subject:

**Current frozen run:**

- scenario `FAIL`;
- `REAL SANDBOX PASS` cannot currently be issued **because the required scenario
  failed** — this reason is complete on its own.

**Proposed fix (undeployed):**

- L1 `REQUIRED`;
- L2 `UNKNOWN`;
- L3 `UNKNOWN`.

These are separate evidence subjects. Do not write the current run's verdict as
"scenario failed and L3 is UNKNOWN" when that L3 belongs only to the proposed
fix.

---

# Release-status anti-inference rule

If `RC status = UNKNOWN / NOT ESTABLISHED`:

- do NOT say "RC promotion is blocked";
- do NOT say "the RC may proceed";
- do NOT infer any other RC disposition.

There is no established RC to block or advance. Report only what is established.
For example, when a required Level 4 scenario has failed:

> `REAL SANDBOX PASS` cannot currently be issued because a required scenario
> failed.

Only if the task explicitly establishes that an RC promotion is in progress may
release blocking be stated (e.g. "the RC cannot be promoted on this evidence").

This rule is about how the outcome is described. It does not change the
scenario-FAIL vs. `CONTAMINATED` distinction, provider-state integrity rules,
temporal contamination rules, restart mechanics, evidence-substitution
semantics, or fail-closed behavior.

---

# Failed-L4-scenario follow-on anti-inference rule

A failed L4 scenario does NOT establish:

- that a fix must happen during the freeze;
- that validation is operationally blocked;
- that the next state is contamination;
- that contamination, if it later occurs, must come from a fix.

The scenario failure itself is not contamination. Any later contamination
requires a separate actual freeze-integrity-breaking event (deployed code /
config change, provider-side state mutation outside the plan, deployment /
revision integrity loss, unauthorized behavior-affecting change, or any other
required integrity condition becoming unestablishable) — see
`references/freeze-protocol.md`, section B.

If continuation and fix necessity are not established, report:

- `run continuation status = UNKNOWN / NOT ESTABLISHED`;
- `in-freeze fix required = UNKNOWN / NOT ESTABLISHED`.

Do not evaluate or recommend the mid-freeze zero-blast-radius fix exception until
`in-freeze fix required = YES` is established. Do not infer a release disposition
while `RC status = UNKNOWN / NOT ESTABLISHED`.

---

# Level 4 decision order

Evaluate in this order. Stop at the first step that resolves Level 4.

## Step A — Is this an RC or a hard exception?

Is a payment-capable / production-intended Release Candidate actually being
promoted, or does a hard exception below apply?

- **No** → **fresh L4 = NOT REQUIRED.** Step A resolves the decision here;
  Step B is **NOT REACHED**. Stop. Ordinary development stays on L1-L3.
- **Yes** → go to Step B.

"No" means both: no payment-capable / production-intended RC promotion is being
made, **and** no hard exception ("Hard exceptions" below) is established as
applicable.

If the prompt explicitly says no RC promotion is being made, report
`RC promotion status = NO`. Do not weaken an explicit `NO` into
`UNKNOWN / NOT ESTABLISHED` merely because broader release chronology (for
example first-paid-launch status) is unknown — that historical fact stays
`UNKNOWN` on its own and does not change the established `NO` for this decision
(see "Evidence-authority rule").

When Step A resolves as `NO`, report the state precisely and separately:

- `RC promotion status = NO`;
- `hard exception = none established as applicable`;
- `fresh L4 = NOT REQUIRED`;
- `Step B (real-time-only classification) = NOT REACHED` — do not report
  `L4 = UNKNOWN`, and do not report a combined `L4 = UNKNOWN / NOT REQUIRED`.

Fresh L4 must be reconsidered later when **either** an RC promotion occurs **or**
a hard exception becomes applicable. RC promotion is not the only future trigger;
never state that it is.

The Release Candidate is only the gate that permits Level 4 to be considered. It
does not by itself make any behavior real-time-only, and "RC + timing-sensitive
change" is not sufficient to require Level 4.

## Step B — Does the behavior needing validation genuinely depend on real elapsed time?

Decide this per changed behavior, by behavioral ownership (see "What
'real-time-only' means" below).

- **No** — the behavior is application-owned and can be proven with
  controlled/injected time or deterministic state → **L4 NOT REQUIRED for that
  behavior.** Prove it at L1/L2. Step C is not reached; **Evidence substitution
  = NOT RELEVANT.**
- **UNKNOWN** — the repository/prompt does not establish whether the timing is
  application-owned or provider-owned → **L4 UNKNOWN.** Classify
  ownership/timing first; do not assume provider-driven timing. Fail closed: do
  not issue a PASS while this is UNKNOWN. Step C is not reached; **Evidence
  substitution = NOT RELEVANT** until ownership/timing is resolved.
- **Yes** — a provider-owned schedule is the thing under test → go to Step C.

## Step C — Can prior valid Level 4 evidence be reused?

Step C is evaluated **only when Step B = YES** — there is genuinely
real-time-only behavior that requires evidence coverage.

- If **Step B = NO** → L4 NOT REQUIRED; Step C is not reached; **Evidence
  substitution = NOT RELEVANT**.
- If **Step B = UNKNOWN** → L4 UNKNOWN; Step C is not reached; **Evidence
  substitution = NOT RELEVANT** until ownership/timing is resolved.
- If **Step B = YES** → evaluate the evidence-substitution conditions below.
  Only in this branch may Evidence substitution be **APPLIES** or **DOES NOT
  APPLY**.

Apply the evidence-substitution conditions below to the real-time-only behavior.

- **Yes**, all conditions demonstrably hold → **fresh L4 NOT REQUIRED for this
  decision.** A future relevant payment-capable RC or applicable hard-exception
  decision re-enters at Step A (see "Substitution is decision-scoped").
- **No**, or zero blast radius cannot be established confidently → **fresh L4
  REQUIRED.** Fail closed.

---

# Hard exceptions

A fresh L4 run is required for:

- the first real paid production launch;
- a provider migration that changes real lifecycle/timing behavior;
- a required restart of a previously `CONTAMINATED` L4 run.

These are not eligible for evidence substitution.

---

# Hard exceptions force L4 but do not define its scenarios

A hard exception may force a fresh L4 run **without** requiring Step B to prove
that a named changed behavior is real-time-only. Reaching L4 through a hard
exception does NOT authorize inventing real-time-only behaviors.

Separate these questions:

## Question 1 — Is a fresh L4 required?

For a hard exception such as the first paid launch: **YES.**

## Question 2 — What exact real-time / provider lifecycle scenarios must L4 cover?

Determine this only from authoritative architecture / provider evidence: the
actual payment architecture, the provider contract, documented provider
behavior, or other authoritative evidence.

If the scenario set is not established:

`Level 4 scenario scope = UNKNOWN / TO BE DEFINED`

Do not automatically list renewal, dunning, retry, recovery, grace expiry, or
cancellation-boundary scenarios merely because this is a first paid launch. Each
such scenario must be shown to exist before it becomes a required L4 scenario.

Fail closed: no `REAL SANDBOX PASS` until the required scenario set has been
authoritatively defined and executed under a valid freeze.

---

# Contamination restart does not determine L1-L3

A `CONTAMINATED` L4 run and its required restart answer only the Level 4
question. They do not by themselves classify L1, L2, or L3.

After a contaminating fix:

- re-evaluate L1-L3 from the fix's actual blast radius;
- if the fix may have changed provider payload interpretation (payload schema,
  provider event type → internal lifecycle-state mapping, or provider field
  semantics used by lifecycle logic) but this is not established, report
  `L3 = UNKNOWN`;
- declare `L3 NOT REQUIRED` only after confirming the provider wire boundary,
  including semantic payload interpretation, is unchanged.

The restart being a hard exception (fresh L4 REQUIRED, no evidence substitution)
does not shortcut or answer this L3 classification.

---

# Contamination-restart hard exception does not determine restart mechanics

The contamination-restart hard exception determines only:

- fresh L4 evidence is `REQUIRED`;
- `Evidence substitution = DOES NOT APPLY`.

It does NOT determine:

- whether the repository revision must change;
- whether a new commit must be created;
- whether code must change;
- whether application configuration must change;
- whether a redeploy must occur;
- whether a fresh test entity is mandatory (versus an authoritative restoration
  procedure that proves the required starting state and integrity).

Those are separate facts, determined by the **actual contamination cause**:

- **code / config change deployed into the frozen environment** → the restart
  baseline must reflect the authoritative corrected / post-fix revision and
  configuration; establish revision integrity and verify deployment integrity
  against that baseline; a different repository revision is required only if the
  authoritative correction actually requires one, and a redeploy action is
  required only if it is necessary to establish the authoritative corrected
  deployed state — neither is inferred from the "code/config contamination"
  classification alone; for the affected scenario's test entity, use a fresh
  disposable entity **or** an authoritative restoration procedure that proves the
  required starting state and all integrity checks — a fresh entity is the
  fail-closed choice only when restoration validity is not established, not an
  automatic requirement;
- **provider-side scenario-state mutation or timing deviation only**, with no
  repository revision, deployed revision, config, or deployment change → the same
  repository revision may remain authoritative; redeployment is not automatically
  required; re-establish a clean frozen validation baseline (clean provider-side
  scenario state plus all required integrity checks) before restart.

## Baseline requirement vs. operational action

Evidence may establish a **constraint on the clean baseline** without
establishing the **operational action** needed to reach it.

- Evidence may establish "the restart baseline must reflect corrected
  code/config" **without** establishing "a redeployment action is required".
- Likewise, "deployment integrity must be verified" does **not** prove
  "redeployment must occur" — the environment may already be running the
  authoritative corrected state.

Keep these separate: authoritative baseline revision/config; revision-integrity
verification; deployment-integrity verification; new-commit requirement;
redeployment requirement. Operational actions (a new commit, a new repository
revision, a redeploy) must be derived from the **actual contamination cause and
the current environment state**, never from a contamination classification
alone. If whether redeployment is necessary has not been established, report
`redeployment requirement = UNKNOWN / NOT ESTABLISHED`.

Do not infer a new-revision, new-commit, code-change, config-change, or redeploy
requirement from the `CONTAMINATED` verdict or from the hard exception being
`ACTIVE`. If revision identity is still valid and unchanged, report it explicitly
(`repository revision = UNCHANGED / STILL AUTHORITATIVE`). See
`references/freeze-protocol.md`, "Restart", and SKILL.md, "Restart-mechanics
anti-inference rule (strict)".

---

# Evidence substitution

Evidence substitution is only considered inside Step C, i.e. when Step B = YES
(genuine real-time-only behavior needs coverage). If Step B resolved as NO or
UNKNOWN, evidence substitution is **NOT RELEVANT** and the conditions below are
not evaluated.

At Release Candidate time, a new L4 run may be skipped only when ALL relevant conditions can be demonstrated:

1. current L2 evidence covers the timing-sensitive application behavior changed since the previous valid L4 run;

2. current provider-boundary evidence is valid, including fresh L3 evidence when required;

3. a previous `REAL SANDBOX PASS` exists;

4. the difference between the previously validated revision and the new Release Candidate has zero blast radius into the real-time-sensitive behavior covered by that previous L4 evidence.

If zero blast radius cannot be established confidently:

- evidence substitution DOES NOT APPLY;
- require a fresh L4 run.

Fail closed.

Distinguish the two ways condition 4 can be unmet — they are different factual
states even though both fail closed to `Evidence substitution = DOES NOT APPLY`
and `fresh L4 = REQUIRED`:

- `zero blast radius = NO` — positive blast radius has been demonstrated;
- `zero blast radius = NOT ESTABLISHED` — insufficient evidence either way;
  do not report this as demonstrated positive impact.

### Evidence method for zero blast radius is not prescribed

Condition 4 is a **result**: zero blast radius into the previously validated
real-time-only behavior, authoritatively and demonstrably established. It is not
a mandate for one specific review mechanism unless authoritative policy says so.

An independent blast-radius review may be one valid method — and may be
recommended when the task raises it — but do not state it is the only possible
path to establishing zero blast radius.

---

# Validation sufficiency vs. release authorization

Payment-validation evidence can establish:

- `L1 / L2 / L3 PASS`;
- `Evidence substitution = APPLIES`;
- `fresh L4 = NOT REQUIRED`.

None of these, alone or together, establishes:

- `release approved`;
- `RC promotion authorized`;
- `deployment authorized`;
- any other go / no-go release disposition.

Release disposition requires its own authoritative evidence and its own scope:
the authoritative release policy plus every required release gate
(business / operational approval, deployment approval, release-management
approval, and any other promotion prerequisite).

When release authorization is not established, stop at the validation
conclusion. For a positive Level 4 reuse decision, the correct closing form is:

> For the payment-validation question presented, the Level 4 requirement is
> satisfied through valid evidence substitution, so a fresh L4 run is not
> required for this decision.

Do not append an imperative such as "Promote the RC", "release", or "deploy"
that is sourced only from validation evidence. Keep the **validation verdict**
separate from **release authorization** (see SKILL.md, "Validation sufficiency
vs. release authorization (strict)").

---

# Substitution is decision-scoped

A successful evidence-substitution decision is scoped to the specific decision,
revision, and state relationship it actually proved. Prior substitution success
may be recorded as historical evidence, but it does not transfer forward.

For a future revision or a future relevant decision:

- do not automatically inherit today's reuse decision;
- re-evaluate the current Step A / Step B / Step C inputs against the evidence
  and revision / state applicable at that time;
- today's zero-blast-radius result applies only to the
  previously-validated-revision → today's-RC relationship it actually proved. A
  later application change can invalidate its relevance even when provider
  identity and provider-owned timing behavior are unchanged.

This is the forward-in-time instance of "Evidence is revision / state scoped"
above. Do not write a closed "fresh L4 only matters again if X / Y / Z happens"
list unless authoritative policy explicitly defines an exhaustive trigger set;
instead state that any future payment-capable, production-intended RC or
applicable hard-exception decision re-enters the Level 4 classification sequence
at Step A.

---

# Evidence-substitution status across a mid-freeze contamination transition

Evidence-substitution status depends on which state the mid-freeze decision is
actually in (see `references/freeze-protocol.md`, "Mid-freeze state machine" and
"Contamination outcome"). The transition is temporal — do not report a later
branch before the event that triggers it has occurred.

- **Before contamination has occurred** — no actual freeze-integrity-breaking
  event has happened yet (e.g. a proposed blast-radius-unproven fix is not yet
  deployed into the frozen environment). This is a mid-freeze integrity
  decision, not the Step A→B→C sequence and not a restart. Report
  `Evidence substitution = NOT RELEVANT`. Do not report `DOES NOT APPLY` here.
- **After contamination has occurred** — an actual freeze-integrity-breaking
  event has happened, the affected run is `CONTAMINATED`, and the
  contamination-restart hard exception applies. Only then report
  `Evidence substitution = DOES NOT APPLY` (hard exceptions are not eligible for
  substitution). "After contamination" means ANY actual contamination event, not
  only deployment of a code fix — for example:
  - a blast-radius-unproven fix deployed into the frozen environment →
    `CONTAMINATED` → `DOES NOT APPLY`;
  - provider-side scenario state manually changed outside the validation plan →
    `CONTAMINATED` → `DOES NOT APPLY`;
  - a required integrity condition can no longer be established after some event →
    `CONTAMINATED` → `DOES NOT APPLY`.

Do not report `DOES NOT APPLY` for the restart branch before contamination has
actually happened, and — per the anti-inference rule — do not infer that a
proposed redeploy, or any other proposed integrity-breaking action, has already
occurred. Once contamination has actually occurred, do not fall back to
`NOT RELEVANT`.

---

# What "real-time-only" means

"Real-time-only" is defined by **behavioral ownership**, not by the phrase
"timing-sensitive".

## Real-time-only — the provider owns the schedule and its actual elapsed-time behavior is what must be proven

- the provider's real retry / dunning schedule;
- naturally delayed or retried provider events;
- provider-driven recovery attempts on the provider's real schedule;
- a grace window whose expiration is enforced by the provider on real elapsed time;
- cancellation effective at a real provider-controlled billing-period boundary;
- renewal across a real provider-controlled billing-period boundary;
- interactions whose correctness depends on multiple real provider-scheduled
  processes racing over elapsed time.

## NOT real-time-only — the application owns the clock and it can be injected or advanced deterministically

- grace-period expiration computed from an injected `now`;
- an application retry / recovery scheduler whose clock is controllable in tests;
- any deterministic timestamp-boundary logic;
- application-owned state transitions on a fast-forwarded test clock.

If application behavior can be established deterministically using controlled
time or state, test it at L1/L2 — including at Release Candidate time.

Do not classify application-owned grace-period or recovery/retry timing as
provider-driven timing unless repository evidence explicitly establishes that the
provider owns the schedule. When ownership is not established, mark real-time-only
behavior `UNKNOWN` and resolve ownership before deciding Level 4.

---

# Parallelism

When independent L4 lifecycle scenarios are required, run them in parallel on separate disposable test entities when safe.

The L4 duration should be determined by the longest required independent real-time scenario, not by unnecessarily serializing every scenario.

---

# Decision examples

## Normal development

Timing-sensitive grace logic changed.

No Release Candidate.

Result:

- L4 NOT REQUIRED.

## Release Candidate, application-owned controllable timing, prior valid REAL SANDBOX PASS

RC being promoted. All changed timing (grace, retry/recovery, cancellation,
renewal) reads an injected / deterministically controllable application clock;
repository evidence does not show the provider owning any schedule. A previous
valid `REAL SANDBOX PASS` exists, current L2 evidence covers the changes, and the
provider boundary has been re-proven where needed.

Result:

- Step A: RC gate met — continue;
- Step B: NOT real-time-only → **L4 NOT REQUIRED**; stop here;
- Step C is **not reached**;
- **Evidence substitution = NOT RELEVANT.**

Do not report that substitution "APPLIES anyway" or that it "independently
confirms" the decision. The prior `REAL SANDBOX PASS` is not consumed here
because Step C was never evaluated.

## Release Candidate, application-owned grace/retry timing changed

The changed grace-period expiration and recovery scheduling read an injected /
controllable application clock. Repository evidence does not show the provider
owning the schedule.

Result:

- Step B: NOT real-time-only — prove the change at L1/L2;
- fresh L4 NOT REQUIRED for that behavior;
- L4 remains required only if some other genuinely provider-owned
  real-elapsed-time behavior has an evidence gap.

## Release Candidate, ownership of the changed timing not established

The prompt says only "grace timing changed" / "recovery scheduling changed", and
neither the prompt nor the repository establishes whether that timing is
application-owned or provider-owned.

Result:

- real-time-only behavior = UNKNOWN;
- L4 = UNKNOWN;
- classify ownership first; do not assume provider-driven timing; fail closed
  (no PASS) until resolved.

## Release Candidate, provider-owned schedule changed

A provider migration or configuration change alters the provider's real dunning
cadence, provider-driven recovery schedule, or a real billing-period boundary,
and that provider schedule is what must be proven.

Result:

- Step B: real-time-only YES;
- previous real-time evidence cannot automatically substitute when that behavior changed;
- fresh L4 REQUIRED unless an authoritative analysis establishes zero relevant
  blast radius into the provider-owned real-elapsed-time behavior.

## First paid launch

Result:

- L4 REQUIRED (hard exception);
- evidence substitution DOES NOT APPLY;
- the Level 4 scenario set is NOT implied by the exception. Derive the exact
  scenarios from the actual payment architecture and real provider behavior; if
  the scenario set is not established, report
  `Level 4 scenario scope = UNKNOWN / TO BE DEFINED` and issue no
  `REAL SANDBOX PASS`.