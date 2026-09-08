# Level 4 Freeze Protocol

Read this file only after Level 4 has been determined to be required.

## Purpose

A long-running real-time validation is meaningful only if the system being observed does not silently change underneath it.

The freeze protects the evidence.

---

# An active freeze does not establish why Level 4 was entered

The existence of an active Level 4 freeze proves only that Level 4 was entered.
It does not establish whether entry came from RC gating or from a hard exception,
nor which hard exception applied, nor any historical release fact.

Do not infer `RC status`, first-paid-launch status, provider-migration status, or
other historical release facts from the freeze being active. If such a fact is
not explicitly established, report it as `UNKNOWN / NOT ESTABLISHED` (see
`references/evidence-model.md`, "Evidence-authority rule"). Conversely, when the
current task explicitly establishes that no RC promotion is being made, report
`RC promotion status = NO` — do not weaken an explicit `NO` into `UNKNOWN`.

---

# A completed clean freeze is closed historical evidence

A Level 4 run that completed cleanly — all required scenarios finished under
intact integrity and `REAL SANDBOX PASS` was issued — remains valid historical
evidence for its pinned frozen revision. Nothing in this protocol reopens it:

- a later **proposed, undeployed** fix does not reopen, contaminate, or
  invalidate the completed run; its evidence stays tied to the pinned revision
  (see `references/evidence-model.md`, "Evidence is revision / state scoped");
- the completed run's restart status stays `NOT TRIGGERED` and contamination
  stays `none`;
- any future fresh-Level-4 question belongs to the **future** evidence subject
  (the post-fix revision) and is triggered and classified independently — through
  RC gating **or** an applicable hard exception, never by reference back to the
  completed run.

---

# Entry

Before starting the first action that begins the real-time scenario:

1. required prerequisite L1-L3 evidence must be complete;
2. the required L4 scenario set must be explicitly defined and justified from
   actual payment / provider behavior (see "Scenario set must be defined before
   the freeze" below);
3. pin an exact repository revision;
4. deploy the validation environment from that revision;
5. verify the deployed revision;
6. prepare disposable test entities;
7. begin the real-time scenario only after the environment is pinned.

The freeze boundary is an exact revision, not merely a calendar date.

---

# Scenario set must be defined before the freeze

Do not start a generic "test everything real-time" freeze.

Before the freeze starts, the required L4 scenario set must be explicitly defined
and justified from the actual payment architecture and real provider behavior —
not assumed from the fact that L4 is required, and not assumed from a hard
exception such as first paid launch.

Each scenario must state:

- what behavior is being observed;
- who owns the timing (provider / application / other);
- why real elapsed time is necessary, or why the hard-exception policy requires
  observing it;
- the expected observable outcome.

If the scenario set is not established:

- the freeze must not start;
- `Level 4 scenario scope = UNKNOWN / TO BE DEFINED`;
- no `REAL SANDBOX PASS` may be issued.

---

# Payment-surface scope

Treat as in scope anything whose behavior may affect the observed payment result, including:

- checkout/payment creation;
- subscription state;
- entitlement decisions;
- webhook processing;
- reconciliation;
- lifecycle mapping;
- payment-owned persistence;
- provider client behavior;
- provider configuration relevant to the test;
- validation-environment deployment configuration;
- shared code that can materially alter those behaviors.

Judge by blast radius, not filename.

When uncertain, treat the change as in scope until zero blast radius is established.

---

# During the freeze

Do not silently:

- redeploy the validation environment;
- modify payment behavior under observation;
- change provider-side configuration relevant to the test;
- manually mutate provider state used by the scenario.

Unrelated development may continue if it does not alter the frozen validation environment.

---

# Integrity checks

A clean Level 4 result requires all of these:

## 1. Revision integrity

The intended frozen revision is known.

## 2. Deployment integrity

The validation environment is proven to be running the intended frozen revision.

## 3. Change integrity

No unauthorized behavior-affecting change entered the tested environment during the observation window.

## 4. Provider-state integrity

Provider-side state relevant to the scenario was not manually changed outside the validation plan.

If any required integrity condition cannot be established:

`CONTAMINATED`

---

# Contamination outcome (applies to any integrity-breaking event)

Any actual integrity-breaking event may trigger contamination, including:

- an unauthorized or blast-radius-uncertain deployed change;
- provider-side state mutation outside the validation plan;
- deployment / revision integrity loss;
- any other required freeze-integrity condition that can no longer be
  established.

Once the affected run is `CONTAMINATED`:

- the contamination-restart hard exception is `ACTIVE`;
- fresh L4 restart = `REQUIRED`;
- `Evidence substitution = DOES NOT APPLY` (a hard-exception restart is not
  eligible for evidence substitution).

This status follows directly from the contamination event. A mid-freeze audit
must not have to infer it from another reference, and it applies whether the
integrity break was a deployed code fix or a provider-side scenario-state change.

Temporal guard: report `CONTAMINATED`, `fresh L4 restart = REQUIRED`, and
`Evidence substitution = DOES NOT APPLY` only after the integrity-breaking event
has actually occurred. Before it, see the mid-freeze state machine below
(`BLOCKED`; contamination `NOT YET OCCURRED`;
`fresh L4 restart = CONDITIONAL / NOT YET TRIGGERED`;
`Evidence substitution = NOT RELEVANT`).

---

# Scenario failure vs. freeze contamination vs. operational blocking vs. in-freeze fix

These are independent facts (A-D below). Decide each on its own evidence; do not
derive one from another. Section E adds a further separation: the current run's
evidence and a proposed undeployed fix's evidence are distinct subjects.

## A. Scenario failure

Conditions:

- the validation procedure was executed according to plan;
- every required freeze-integrity condition remains establishable;
- the observed result differs from the predefined expected result.

Then:

- record `scenario FAIL`;
- the freeze remains valid — this is NOT `CONTAMINATED`;
- no contamination-restart hard exception is triggered;
- `fresh L4 restart` is NOT triggered by the failure itself;
- `REAL SANDBOX PASS` cannot be issued while a required scenario is failing.

A `scenario FAIL` under intact integrity is a genuine product/provider-behavior
result and is valid Level 4 evidence of a failure. Do not assume contamination
merely because a scenario failed.

## B. Freeze contamination

The scenario failure itself is not contamination. Any later contamination
requires a separate actual freeze-integrity-breaking event.

Decided only from the four integrity checks above. A differing observed result is
not an integrity break. A separate later contaminating event may be any of:

- a deployed code / application-configuration change entering the frozen
  environment;
- provider-side scenario-state mutation outside the validation plan (or a
  predefined provider action run outside its allowed timing window);
- deployment integrity loss;
- revision integrity loss;
- an unauthorized behavior-affecting change;
- any other required integrity condition becoming unestablishable.

Contamination is not limited to a later code fix. Do not use wording equivalent
to "contamination can only enter at the fix".

## C. Operational blocking

- report `BLOCKED` only if continuation of the required validation is actually
  prevented (established by the prompt or the evidence);
- do NOT derive `BLOCKED` from `scenario FAIL` alone;
- if continuation is not established either way, report
  `run continuation status = UNKNOWN / NOT ESTABLISHED`.

Other independent scenarios may continue if their execution remains valid and
independent of the failed scenario.

## D. Is an in-freeze fix actually required?

Do not evaluate the blocking-fix exception (the documented zero-blast-radius
mid-freeze fix under "Mid-freeze bug decision", Step 2 → "Yes") unless it is
established that **required validation cannot continue without the fix**.

A `scenario FAIL` does not by itself establish this. Do not infer
`scenario FAIL → blocking bug → mid-freeze fix exception`.

- established YES → evaluate the proposed fix under the mid-freeze bug decision;
- established NO → do not enter the blocking-fix exception path; do not recommend
  redeploying during the freeze;
- `UNKNOWN / NOT ESTABLISHED` → do not enter the blocking-fix exception path; do
  not classify any fix as a non-contaminating exception yet; report
  `in-freeze fix required = UNKNOWN / NOT ESTABLISHED`.

## E. Evidence isolation between the current run and the proposed fix

Before a proposed fix is deployed, keep two evidence scopes separate:

**Current-run evidence** — remains tied to the pinned frozen revision and the
observations already made against it (scenario outcomes, freeze integrity,
operational state). Its verdict is determined only by those observations.

**Proposed-fix evidence** — separate. Classify the proposed fix's L1-L3 as
needed to decide whether and how it may proceed. Do **not** attach those
proposed-fix evidence states (`REQUIRED` / `NOT REQUIRED` / `UNKNOWN` / `PASS` /
`FAIL`) to the current frozen run's verdict.

Consequences:

- the current run's inability to issue `REAL SANDBOX PASS` is explained by the
  required L4 scenario result alone; do not add an undeployed proposed fix's
  `L3 = UNKNOWN` (or any other proposed-fix state) as a further reason;
- a proposed-fix `L3 = UNKNOWN` does not block or satisfy anything for the
  current run — it only gates the proposed fix.

Only if the fix actually enters the frozen environment does it become relevant
to that environment's integrity / evidence state:

- an uncertain / non-zero-blast-radius fix that enters → the temporal
  contamination rules apply (see "Mid-freeze state machine");
- a fix that remains outside → its `UNKNOWN` / `PASS` / `FAIL` classification
  stays external to the current run.

## Decision order

1. Did the scenario fail?
2. Is freeze integrity still intact?
3. Can required validation continue?
4. Is an in-freeze fix required in order to continue?
5. Only if #4 = YES, evaluate zero blast radius for the proposed fix.

If #3 or #4 is `UNKNOWN / NOT ESTABLISHED`:

- do not classify the fix as a non-contaminating exception yet;
- do not recommend redeploying during the freeze;
- other independent scenarios may continue if valid and independent.

---

# Current Level 4 run status vs. fresh Level 4 restart requirement

During an already-active L4 freeze these are two separate facts. Never use
`L4 REQUIRED` as shorthand for both.

## Current run status — the run that already exists

One of:

- `ACTIVE / IN PROGRESS`;
- `BLOCKED` (operationally cannot continue — report only if established);
- `CONTAMINATED`;
- `COMPLETED CLEANLY`.

## Fresh restart status — whether a new real-time run must be executed

One of:

- `NOT TRIGGERED`;
- `CONDITIONAL / NOT YET TRIGGERED`;
- `REQUIRED`.

## Required temporal rule

`scenario FAIL` + intact freeze + blocking fix **not yet deployed**:

- current L4 run status = `ACTIVE / IN PROGRESS`, operationally `BLOCKED`;
- fresh L4 restart = `CONDITIONAL / NOT YET TRIGGERED`;
- contamination-restart hard exception = `NOT ACTIVE`.

Uncertain / non-zero-blast-radius fix **deployed** into the frozen environment:

- current affected run = `CONTAMINATED`;
- fresh L4 restart = `REQUIRED`;
- contamination-restart hard exception = `ACTIVE`.

An active run being `BLOCKED` is not a fresh-restart requirement. A generic
`L4 REQUIRED` must not stand in for either the current-run status or the
restart status.

## Canonical field set for a pre-deployment mid-freeze decision

For a required scenario that failed while freeze integrity is intact, an
in-freeze fix is required, and the fix's blast radius / provider-boundary impact
is not established and it has not been deployed. Report the two evidence
subjects separately (see "E. Evidence isolation between the current run and the
proposed fix").

**Current frozen run:**

- scenario outcome = `FAIL` (genuine product/provider-behavior result; valid L4
  evidence of a failure)
- freeze integrity = `NOT CONTAMINATED`
- current L4 run status = `ACTIVE / IN PROGRESS` (operationally `BLOCKED` only if
  continuation of the required validation is established as prevented; otherwise
  `run continuation status = UNKNOWN / NOT ESTABLISHED`)
- contamination = `NOT YET OCCURRED`
- fresh L4 restart = `CONDITIONAL / NOT YET TRIGGERED`
- contamination-restart hard exception = `NOT ACTIVE`
- Evidence substitution = `NOT RELEVANT`
- RC status = `UNKNOWN / NOT ESTABLISHED`
- `REAL SANDBOX PASS` = cannot currently be issued **because the required L4
  scenario = `FAIL`** (this reason is complete on its own; do not append the
  proposed fix's L3 state)

**Proposed fix (not deployed — evidence external to the current run):**

- in-freeze fix required = `YES`
- fix zero blast radius = `NOT ESTABLISHED`
- L1 = `REQUIRED`
- L2 = `UNKNOWN`
- L3 = `UNKNOWN` — must be resolved before the fix proceeds where applicable;
  does not block or satisfy anything for the current frozen run

If the uncertain / non-zero-blast-radius fix is later deployed into the frozen
environment:

- current affected run = `CONTAMINATED`
- fresh L4 restart = `REQUIRED`
- contamination-restart hard exception = `ACTIVE`
- Evidence substitution = `DOES NOT APPLY`
- restart baseline = the authoritative corrected / post-fix revision / config
- deployment verification = `REQUIRED` before the new freeze (verify the deployed
  environment against that authoritative corrected baseline)
- redeployment action = only if necessary to establish that authoritative
  corrected deployed state; `deployment verification required` ≠ `redeployment
  required`
- entity handling = fresh disposable entity **OR** an authoritative restoration
  procedure that proves the required starting state and all integrity checks;
  fail closed to a fresh entity only if restoration validity is not established
  (a fresh entity is not automatically mandatory)

---

# Mid-freeze bug decision

During a mid-freeze contamination decision:

- determine contamination from **freeze integrity and blast radius** only;
- do not infer RC status, launch status, provider-migration status, or other
  historical release facts from the freeze being active or from the fix being
  needed;
- if those fields are not relevant to the contamination decision, they may be
  reported as `UNKNOWN / NOT ESTABLISHED`.

## Mid-freeze state machine (temporal ordering)

These states are ordered in time. Do not report a later state, or its
consequences, before the event that causes it has actually occurred.

**State 1 — blocking issue discovered**

- current L4 run status = `ACTIVE / IN PROGRESS`;
- operational status may be `BLOCKED` (if established);
- fresh L4 restart = `NOT TRIGGERED` (nothing contaminating proposed yet).

**State 2 — proposed fix evaluated**

- zero blast radius established → a documented non-contaminating exception may be
  possible (see Step 2 → "Yes" below);
- zero blast radius NOT established → the fix is contaminating **if deployed**.

**Before the fix is deployed into the frozen environment:**

- current L4 run status = `ACTIVE / IN PROGRESS`, operationally `BLOCKED`;
- it is **NOT yet** `CONTAMINATED`;
- fresh L4 restart = `CONDITIONAL / NOT YET TRIGGERED`;
- contamination-restart hard exception = `NOT ACTIVE`;
- `Evidence substitution = NOT RELEVANT`.

**After the fix is deployed into the frozen environment with unresolved blast
radius:**

- current affected run status = `CONTAMINATED`;
- the contamination-restart hard exception activates;
- fresh L4 restart = `REQUIRED` (a new real-time run must be executed — distinct
  from the now-contaminated run that already existed);
- `Evidence substitution = DOES NOT APPLY`.

The same "after contamination" state applies to any other actual
integrity-breaking event, including a provider-side scenario-state change made
outside the validation plan (see "Contamination outcome" above).

Do not infer that a proposed redeploy has already occurred.

If a bug appears during L4:

## Step 1

Does the bug prevent the required validation from continuing?

### No

Do not modify the frozen environment merely to clean up the bug.

Record it for later.

The current validation may continue.

### Yes

Evaluate the proposed fix.

---

## Step 2

Can the fix be demonstrated to have zero blast radius into payment behavior being validated?

### Yes

It may be treated as a non-contaminating exception, provided:

- the change is explicitly recorded;
- the new deployed revision is verified;
- provider-side scenario state remains valid;
- the reasoning for zero blast radius is recorded.

### No or uncertain

The change is contaminating.

The affected run becomes:

`CONTAMINATED`

Do not represent it as `REAL SANDBOX PASS`.

---

# Restart

Every `CONTAMINATED` run restarts the affected real-time validation from a
**clean, explicitly re-established frozen validation baseline**.

"A new frozen baseline" does not automatically mean "a new repository revision".
What must change is determined by the **contamination cause**, not by the
`CONTAMINATED` verdict itself (see also SKILL.md, "Restart-mechanics
anti-inference rule (strict)").

## Classify what must change by contamination cause

**Code / config contamination** — a code or config change was deployed into the
frozen environment, or blast radius into payment behavior is unresolved:

- identify the authoritative corrected revision / configuration;
- use that state as the restart baseline;
- establish revision integrity (the intended authoritative revision is known) —
  this may require a different repository revision, but only if the authoritative
  correction actually requires one; do not infer a new revision merely from the
  "code/config contamination" classification;
- establish deployment integrity by verifying what is actually deployed against
  that authoritative corrected baseline;
- redeploy only if a redeploy action is necessary to make the deployed
  environment match the authoritative corrected baseline;
- re-run all Entry integrity checks;
- for the affected scenario's test entity, apply "## Contaminated test entity"
  below — a fresh disposable entity is not automatically mandatory; an
  authoritative restoration procedure that proves the required starting state and
  all integrity checks is equally valid, and a fresh entity is the fail-closed
  choice only when restoration validity is not established.

Do not write a universal "code/config contamination → redeploy". Deployment
verification is always required before a new freeze; a redeployment action is
required only when it is actually necessary to establish the authoritative
corrected deployed state. If whether redeployment is necessary has not been
established, report `redeployment requirement = UNKNOWN / NOT ESTABLISHED`.

**Provider-side-only contamination** — provider-side scenario state was mutated,
or a predefined provider-side action ran outside its allowed timing window, and
no repository revision, deployed revision, application config, or deployment
changed:

- the repository revision may remain unchanged and still authoritative — if so,
  state that explicitly (`repository revision = UNCHANGED / STILL AUTHORITATIVE`)
  rather than inventing a new-revision requirement;
- redeployment is not automatically required;
- re-establish clean provider-side scenario state and re-verify every required
  integrity check (revision, deployment, change, provider-state) before restart.

Do not require a new repository revision, a new commit, a code change, a config
change, or a redeploy merely because the previous run was `CONTAMINATED`.

## Restart scope

- default to restarting the affected real-time validation in full;
- a narrower restart is acceptable only when already-completed lifecycle stages
  can be shown to be unaffected and that justification is explicitly recorded.

When uncertain, restart.

## Contaminated test entity

This applies to the affected scenario's test entity whenever it can no longer be
treated as clean evidence — provider-side scenario state on the entity was
contaminated, **or** the entity was exercised in a run that became
`CONTAMINATED` for any cause, including code/config contamination:

- that entity cannot be treated as clean evidence for the affected scenario;
- use a fresh disposable entity, OR an explicitly allowed authoritative
  restoration procedure that can prove the required starting state and all
  integrity checks;
- if restoration validity is not established, fail closed and use a fresh entity.

A fresh entity may be the default practical choice, but no contamination cause
(code/config included) makes a fresh entity *universally mandatory* unless
authoritative policy explicitly says so. Do not assert an irreversible
"burned forever" rule for the entity unless authoritative policy establishes it.

## Re-classify the fix before re-entry

This step applies only when contamination involved an actual code / config fix.
Provider-side-only contamination (no code/config change) has no fix to
re-classify; re-enter once the clean provider-side baseline and all integrity
checks are re-established.

After a contaminating mid-freeze fix, send the fix back through the normal
L1-L3 classification before re-entering the freeze. Freeze-contamination status
does not substitute for L1-L3 classification.

For webhook / lifecycle-mapping fixes:

- provider payload interpretation changed (provider event type → internal
  lifecycle-state mapping, or provider webhook field semantics used by lifecycle
  logic) → L3 REQUIRED;
- purely internal downstream mapping only, provider payload parsing and
  interpretation unchanged → L3 may be NOT REQUIRED;
- not established which applies → L3 UNKNOWN; classify the provider-boundary
  impact before re-entry. Fail closed: no PASS while L3 is UNKNOWN.

---

# Exit

Before declaring `REAL SANDBOX PASS`:

1. confirm all required real-time scenarios actually completed;
2. verify revision integrity;
3. verify deployment integrity;
4. verify change integrity;
5. verify provider-state integrity;
6. classify every exception;
7. ensure no contaminating event invalidated the required evidence.

Only then may the run be recorded as:

`REAL SANDBOX PASS`

Otherwise use:

`CONTAMINATED`

or state that evidence is incomplete.