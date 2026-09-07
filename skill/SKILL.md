---
name: saas-payment-validation
description: >
  Plan, review, execute, or audit engineering validation for behavior-affecting
  SaaS payment code and payment-capable releases. Use when a change touches
  checkout, billing state, subscriptions, entitlements, payment webhooks,
  reconciliation, provider APIs, payment persistence/concurrency, or
  release-candidate payment lifecycle behavior. Use it to choose L1-L4
  evidence, decide whether Level 4 is actually required, and protect real-time
  validation from contamination. Do not use for billing copy/CSS, pricing
  strategy, provider selection, or general payment questions that do not
  require engineering validation.
---

# SaaS Payment Validation

## Goal

Choose the minimum sufficient validation for a payment change without weakening correctness.

Levels 1-3 are the normal engineering loop.

Level 4 is a release-candidate real-time evidence mechanism. It is not the default "stronger test" for ordinary payment development.

## First classify the task

Determine:

- what payment behavior changed;
- what other payment behavior may be affected by blast radius;
- whether this is normal development or a payment-capable Release Candidate;
- whether the real provider boundary changed;
- whether correctness depends on real persistence/concurrency;
- whether any behavior can only be demonstrated through real elapsed time;
- what validation evidence has actually been executed.

If something cannot be established, mark it `UNKNOWN`.

Never invent test results, release status, provider capabilities, or previous evidence.

## Anti-inference rule (strict)

Do not infer missing facts merely because they would be compatible with the
current validation state.

Examples:

- an active Level 4 freeze does NOT prove RC = YES;
- `L4 REQUIRED` does NOT prove a particular hard exception applies;
- a prior `REAL SANDBOX PASS` does NOT prove this is not the first paid launch
  unless the chronology is actually established;
- provider sandbox usage does NOT prove provider-owned timing exists;
- repository presence of reconciliation / concurrency code does NOT prove L2 is
  required for the current change.

If a requested output field is not established by prompt, repository,
authoritative evidence, or explicit task context, report:

`UNKNOWN / NOT ESTABLISHED`

Do not fill it by implication.

### Explicit facts outrank uncertainty

`UNKNOWN / NOT ESTABLISHED` is only for genuinely missing information. When the
prompt, repository, authoritative evidence, or explicit task context *explicitly
establishes* a fact, report that fact — do not downgrade an explicit `NO` (or an
explicit `YES`) to `UNKNOWN` because some broader or adjacent fact is still
unknown.

In particular, for **RC promotion status for the current decision**:

- if the prompt explicitly states that no payment-capable, production-intended
  Release Candidate promotion is being made, report `RC promotion status = NO` —
  not `UNKNOWN / NOT ESTABLISHED`;
- "no new Release Candidate promotion decision is being made yet" is an
  established `NO` for this decision, even when release chronology is otherwise
  unknown.

Keep these separate:

- **RC promotion status for this decision** — `YES` / `NO` / `UNKNOWN`, taken
  from what the current task explicitly establishes;
- **historical release facts** — first-paid-launch chronology,
  provider-migration history, prior-`REAL SANDBOX PASS` ordering — which may
  independently remain `UNKNOWN / NOT ESTABLISHED` and do not change an
  established current-decision `NO`.

An established `RC promotion status = NO` with no applicable hard exception
resolves Step A of the Level 4 classification sequence: `fresh L4 = NOT REQUIRED`,
and Step B is `NOT REACHED`. If real-time ownership was never classified because
Step B was not reached, report `real-time-only classification = NOT REACHED`
separately. Do not report `L4 = UNKNOWN`, and do not report a combined
`L4 = UNKNOWN / NOT REQUIRED`, for that decision.

### Aggregate facts do not establish their component facts (strict)

When a task establishes an **aggregate** decision, report that aggregate fact.
Do not decompose it into unsupported historical facts about every component.

- Explicit: `no hard exception applies`
- Correct: `hard exception applicability = NONE APPLICABLE`
- Incorrect unless each is separately established:
  `first paid launch = NO`, `provider migration = NO`,
  `prior contamination restart = NO`

`hard exception applicability = NONE APPLICABLE` does not establish the
underlying historical truth value of any individual hard-exception condition.
Report each such predicate as `UNKNOWN / NOT ESTABLISHED` unless it is
independently established.

This also means not expanding a stated present-tense fact into a historical
predicate. "provider is unchanged" establishes
`provider identity / status = unchanged as stated`; it does **not** by itself
establish the historical `provider migration` hard-exception predicate as `NO`
unless the authoritative definition of "provider migration" (see
`references/evidence-model.md`, "Hard exceptions") makes that predicate follow
directly. Safer default when the task states `no hard exception applies`: report
`hard exception applicability = NONE APPLICABLE`, report any explicitly stated
provider fact as stated, and do not decompose the individual hard-exception
predicates unless one is independently needed and independently established.

The inverse also applies: an established *applicable* hard exception does not
automatically identify **which** hard exception applies. Report
`hard exception identity = UNKNOWN / NOT ESTABLISHED` unless the identity is
explicitly established (see `references/evidence-model.md`, "Evidence-authority
rule").

### Zero blast radius is a three-state fact (strict)

Keep these three states distinct — never collapse them:

- `zero blast radius = YES` — affirmatively demonstrated; the reuse condition is
  satisfied.
- `zero blast radius = NO` — positive blast radius has been demonstrated; the
  reuse condition fails on demonstrated impact.
- `zero blast radius = NOT ESTABLISHED` — insufficient evidence to reuse prior
  Level 4 evidence; the reuse condition fails closed.

`NOT ESTABLISHED` is the absence of the required reuse evidence. It does **not**
mean positive blast radius has been proven. Never transform
`zero blast radius = NOT ESTABLISHED` into
`affected / in blast-radius path = YES` (or into `zero blast radius = NO`) unless
positive impact is independently established.

For evidence substitution, both `NO` and `NOT ESTABLISHED` lead to:

- condition unmet → `Evidence substitution = DOES NOT APPLY` →
  `fresh L4 = REQUIRED`, fail closed.

They are nonetheless different factual states and must be reported as such.
Reaching `DOES NOT APPLY` via `NOT ESTABLISHED` does not license asserting actual
positive impact.

#### Configuration relevance vs. configuration-delta impact (strict)

A component or provider-side configuration may be **relevant to**, **participate
in**, **govern**, or **configure** a provider-owned behavior. That relationship
does **not** mean a *change* to that component/configuration has been proven to
alter the observed behavior.

Keep separate:

- `configuration participates in / governs the behavior = YES` — a structural
  relationship;
- `configuration delta has positive blast radius into the behavior =
  YES / NO / NOT ESTABLISHED` — a demonstrated effect of the change.

The two facts "the configuration changed" and "the configuration participates in
the covered provider-owned behavior" do **not** together establish
`actual positive blast radius = YES`, or "the covered behavior was affected by
the change", unless affirmative impact evidence exists.

If positive impact is not established, do not use wording that semantically
asserts it:

- "affected by the change";
- "impacted by the change";
- "touched by the change";
- "changed behavior" / "the behavior the change altered".

Use neutral wording instead:

- "behavior associated with the changed configuration";
- "behavior governed in part by that provider-side configuration";
- "previously validated behavior for which zero blast radius from the
  configuration delta is NOT ESTABLISHED".

`actual delta impact = NOT ESTABLISHED` together with
`zero blast radius = NOT ESTABLISHED` is a valid, coherent combination: it fails
the reuse condition closed (`Evidence substitution = DOES NOT APPLY`,
`fresh L4 = REQUIRED`) without asserting demonstrated impact.

### Evidence requirements specify the result, not an exclusive mechanism (strict)

A validation requirement specifies **what a piece of evidence must establish**,
not a single mandatory way to gather it, unless authoritative policy explicitly
mandates that mechanism.

For zero blast radius:

- required result: authoritatively and demonstrably established zero blast radius
  into the previously validated real-time-only behavior;
- do not say "an independent blast-radius review is the only path" unless
  authoritative policy says so.

If the task mentions an independent review that has not yet occurred, it may be
recommended as **one possible** method — not declared universally mandatory.

## Validation sufficiency vs. release authorization (strict)

A payment-validation answer establishes only the validation state that was
asked for. It does not establish a release / promotion / deployment disposition.

In particular:

- `Evidence substitution = APPLIES`
- `fresh L4 = NOT REQUIRED`
- `L1 / L2 / L3 PASS`

do **not** by themselves imply:

- "RC promotion authorized";
- "release approved";
- "deploy now";
- any other go / no-go release decision.

State a release / promotion disposition only when **both**:

1. release disposition is explicitly in scope for the task; **and**
2. the authoritative release policy and every required release gate
   (business / operational / deployment / release-management approval, plus any
   other promotion prerequisite) are established.

When release authorization is not established, stop at the validation
conclusion — e.g. "for the payment-validation question presented, the Level 4
requirement is satisfied through valid evidence substitution, so a fresh L4 run
is not required for this decision." Do not add imperative next actions such as
"Promote the RC", "ship it", or "deploy" that are sourced only from validation
evidence.

Keep separate: **validation verdict** vs. **release authorization**.

## Mid-freeze state-transition rule (strict)

Do not collapse these into one status:

- a proposed change;
- a deployed change;
- contamination;
- a restart requirement.

They are ordered in time. Report only the state that the decision is actually in.
Each state carries its own evidence-substitution status; report that status
directly from the state — do not infer it from another reference.

**Before contamination has occurred** — e.g. a proposed contaminating /
blast-radius-uncertain fix NOT yet deployed into the frozen environment:

- current Level 4 run status = `ACTIVE / IN PROGRESS`
- operational status = `BLOCKED` (only if established; otherwise
  `run continuation status = UNKNOWN / NOT ESTABLISHED`)
- contamination = `NOT YET OCCURRED`
- fresh L4 restart = `CONDITIONAL / NOT YET TRIGGERED`
- contamination-restart hard exception = `NOT ACTIVE`
- Evidence substitution = `NOT RELEVANT`

**After contamination has occurred** — any actual freeze-integrity-breaking event
has happened: a blast-radius-uncertain fix deployed into the frozen environment,
provider-side scenario state manually changed outside the validation plan,
deployment / revision integrity lost, or any other required freeze-integrity
condition that can no longer be established:

- current affected run = `CONTAMINATED`
- fresh L4 restart = `REQUIRED`
- contamination-restart hard exception = `ACTIVE`
- Evidence substitution = `DOES NOT APPLY` (hard exceptions are not eligible for
  substitution)

`ACTIVE / IN PROGRESS` (the current run already exists) and
`fresh L4 restart = REQUIRED` (a new run must be executed) are separate facts.
Do not report a generic `L4 = REQUIRED` that collapses them — see "Current
Level 4 run status vs. fresh Level 4 restart requirement (strict)" below.

Do not report `fresh L4 restart = REQUIRED`, `CONTAMINATED`, or
`Evidence substitution = DOES NOT APPLY` before the event that triggers
contamination has actually occurred. Per the anti-inference rule, do not assume a
proposed redeploy — or any other proposed integrity-breaking action — has already
happened. Conversely, once contamination has actually occurred, do not describe
evidence substitution as `NOT RELEVANT`.

## Current Level 4 run status vs. fresh Level 4 restart requirement (strict)

When a Level 4 freeze is already active, "L4" is not one field. Report two:

**A. Current Level 4 run status** — the state of the run that already exists:

- `ACTIVE / IN PROGRESS` — the freeze is live and the run has not been
  contaminated or completed;
- `BLOCKED` — operationally cannot continue (report only if established);
- `CONTAMINATED` — an actual freeze-integrity-breaking event has occurred;
- `COMPLETED CLEANLY` — all required scenarios finished under intact integrity.

**B. Fresh Level 4 restart requirement** — whether a new real-time validation
run must be executed:

- `NOT TRIGGERED`;
- `CONDITIONAL / NOT YET TRIGGERED` — a contaminating event is possible but has
  not occurred (e.g. a blast-radius-uncertain fix is proposed but not deployed);
- `REQUIRED` — an actual contamination event has occurred; the
  contamination-restart hard exception is `ACTIVE`.

Rules:

- Do not report a generic `L4 = REQUIRED` merely because an L4 freeze is active
  or because a scenario failed. An already-active run is not a fresh-restart
  requirement.
- `scenario FAIL` + intact freeze + blocking fix not yet deployed →
  current L4 run status = `ACTIVE / IN PROGRESS` (operationally `BLOCKED` if
  established); fresh L4 restart = `CONDITIONAL / NOT YET TRIGGERED`.
- Uncertain / non-zero-blast-radius fix deployed into the frozen environment →
  current affected run = `CONTAMINATED`; fresh L4 restart = `REQUIRED`.
- If a task asks for "L4" during an active freeze, always report both the
  current L4 run status and the fresh L4 restart status so they cannot be
  confused.

## Active-L4 execution / status review: do not expand known run-state into unestablished detail (strict)

When the task is an execution / status review of an already-active Level 4 run,
answer only from the run-state facts that are actually established. Known
run-state facts must not be expanded into unestablished execution details.

Keep these four fact classes separate; establishing one never establishes
another:

- what scenario execution state is established (PASS / FAIL / PENDING per
  scenario; aggregate run completion);
- what freeze-integrity state is established (the four integrity checks);
- what execution artifacts / resources are established (test entities, fixtures,
  provisioning / preparation state);
- what unrelated mid-freeze events are established (proposed fixes, operator
  actions, provider-side actions).

A clean, active freeze is a narrow fact. Do not infer, merely because a clean
active freeze exists:

- that no fix has been proposed;
- that a fix exists;
- that a test entity has already been provisioned;
- that a disposable entity is already prepared;
- that a specific execution resource (entity, environment slot, fixture,
  credential, queue, operator step) exists;
- that any unspecified operator action has or has not occurred.

Rules:

- If such a fact is not needed for the requested determination, omit it — do not
  add it as narrative support for a conclusion.
- If such a fact is relevant to the determination but not established, report it
  as `UNKNOWN / NOT ESTABLISHED`. Do not fill it by implication from the clean
  freeze.
- Justify `fresh L4 restart = NOT TRIGGERED` only by the established fact that
  applies: **no contamination event has occurred**. Do not add unrelated negative
  facts (e.g. "no proposed contaminating fix", "no operator change") to prop up
  that state.

### Next-action wording when a required scenario is pending

Canonical safe action:

> Execute the pending required scenario under the still-valid freeze according to
> the authoritative scenario plan and freeze protocol.

Do **not** add "use its prepared disposable test entity" — or name any other
specific execution resource — unless preparation / provisioning of that entity is
explicitly established. If entity handling must be mentioned but its state is not
established, say:

> Use the entity handling required by the authoritative scenario plan / freeze
> protocol; provisioning status is `NOT ESTABLISHED`.

This is the active-freeze status-review instance of the anti-inference rule. It
does not change any existing rule for scenario PASS vs. FAIL vs. PENDING,
`BLOCKED` only when required validation cannot continue, contamination vs.
scenario outcome, aggregate completion, `REAL SANDBOX PASS` gating, freeze
integrity, release-authorization separation, anti-inference, or fail-closed
behavior.

## Proposed-fix evidence isolation (strict)

During an active freeze, evidence has two separate scopes. Do not mix them.

**A. Current frozen revision / current L4 run evidence** — the observations
already made against the pinned frozen revision (scenario outcomes, freeze
integrity, operational state).

**B. Proposed-fix evidence** — the L1 / L2 / L3 classification of a fix that is
being considered.

If a proposed fix has **not** entered the frozen environment:

- its L1 / L2 / L3 `REQUIRED` / `NOT REQUIRED` / `UNKNOWN` / `PASS` / `FAIL`
  states belong **only to the proposed fix**;
- they do **not** alter the observed evidence or verdict of the current frozen
  revision;
- they do **not** independently block or satisfy `REAL SANDBOX PASS` for the
  current run;
- they only determine whether and how the proposed fix may proceed.

The current run's inability to issue `REAL SANDBOX PASS` is fully explained by
the required L4 scenario result. Do not append an undeployed proposed fix's
evidence state as an additional reason.

Example — current required L4 scenario = `FAIL`; proposed-fix L3 = `UNKNOWN`;
fix not deployed:

- **Correct:** current `REAL SANDBOX PASS` cannot be issued because the required
  scenario = `FAIL`. Proposed-fix L3 = `UNKNOWN` and must be resolved before that
  fix proceeds where applicable.
- **Incorrect:** current `REAL SANDBOX PASS` cannot be issued because the
  scenario failed "and L3 is UNKNOWN" (that L3 status belongs only to the
  undeployed proposed fix).

Only when the fix actually enters the frozen environment does its classification
become relevant to that environment's integrity / evidence state (at which point
the temporal contamination rules apply).

## Scenario-result state rule (strict)

A Level 4 scenario outcome, freeze integrity, and operational run state are
**three separate facts**. Classify each on its own evidence. Do not collapse
them, and do not infer one from another.

**1. Scenario outcome** — did the executed scenario produce its predefined
expected observable result?

- expected outcome observed → `scenario PASS candidate`;
- expected outcome NOT observed → `scenario FAIL`.

A `scenario FAIL` where the validation procedure ran according to plan and all
freeze-integrity conditions remain establishable is a genuine
product/provider-behavior result. It is valid Level 4 evidence of a failure.

**2. Freeze integrity** — decided only from the four integrity checks (see
`references/freeze-protocol.md`):

- integrity intact / all four conditions establishable → `NOT CONTAMINATED`;
- a required integrity condition can no longer be established → `CONTAMINATED`.

A `scenario FAIL` does **not** by itself break integrity. Do not report
`CONTAMINATED`, `contamination-restart hard exception = ACTIVE`, or
`fresh L4 restart = REQUIRED` because a scenario failed.

**3. Operational run state** — whether required validation can still continue:

- report `BLOCKED` only when the prompt or evidence establishes that continuation
  of the required validation is actually prevented;
- do **not** infer `BLOCKED` merely because one scenario failed;
- if continuation status is not established, report
  `run continuation status = UNKNOWN / NOT ESTABLISHED`.

Other independent Level 4 scenarios may still be able to continue on their own
disposable entities if their execution remains valid and independent.

**Consequences of a failed required scenario:**

- `REAL SANDBOX PASS` cannot currently be issued;
- this does NOT automatically mean `CONTAMINATED`;
- this does NOT automatically mean the whole L4 run is operationally `BLOCKED`;
- the failed required scenario is a **sufficient and complete** reason on its
  own. Do not append an undeployed proposed fix's L1 / L2 / L3 state (e.g.
  "and L3 is UNKNOWN") as an additional reason — see "Proposed-fix evidence
  isolation (strict)";
- per the release-status anti-inference rule in
  `references/evidence-model.md`, state only that `REAL SANDBOX PASS` cannot
  currently be issued because a required scenario failed — do not describe any RC
  disposition ("RC promotion blocked", "RC may proceed") unless an RC promotion
  is explicitly established.

**A `scenario FAIL` itself:**

- is not contamination;
- does not imply that future contamination could only enter through a fix;
- does not imply a mid-freeze fix is required;
- does not imply the blocking-fix exception is available.

**Future contamination** occurs only if a *separate* actual
freeze-integrity-breaking event happens. That event may be any of:

- a deployed code / application-configuration change entering the frozen
  environment;
- provider-side scenario-state mutation outside the validation plan (or a
  predefined provider action run outside its allowed timing window);
- deployment integrity loss;
- revision integrity loss;
- an unauthorized behavior-affecting change;
- any other required integrity condition becoming unestablishable.

Do not describe contamination as something that "can only enter at the fix".
The scenario failure itself is not contamination; any later contamination
requires a separate actual freeze-integrity-breaking event.

**4. Blocking-fix exception gate.** Before evaluating a mid-freeze fix as a
possible non-contaminating exception, first establish:

`required validation cannot continue unless the fix is applied`

- **YES** → evaluate the fix under the mid-freeze bug decision
  (`references/freeze-protocol.md`).
- **NO** → do not enter the blocking-fix exception path; do not recommend
  redeploying during the freeze.
- **UNKNOWN / NOT ESTABLISHED** → do not enter the blocking-fix exception path;
  report `run continuation status = UNKNOWN / NOT ESTABLISHED` and
  `in-freeze fix required = UNKNOWN / NOT ESTABLISHED`.

Do not infer the chain `scenario FAIL → blocking bug → mid-freeze fix
exception`. Each link needs its own evidence.

## Restart-mechanics anti-inference rule (strict)

`CONTAMINATED` means the affected validation evidence must be
restarted / re-established from a clean, explicitly re-established frozen
validation baseline.

It does NOT by itself prove that:

- a new commit must be created;
- a different repository revision must be used;
- code must change;
- application configuration must change;
- a redeploy must occur;
- a fresh test entity is always mandatory;
- any other implementation action must occur.

Restart requirements are a separate fact. Derive them from the **actual
contamination cause**, not from the `CONTAMINATED` verdict.

### Baseline requirement vs. operational action (strict)

The contamination cause may determine **constraints on the clean baseline**
without determining the **exact operational action** needed to reach it. Keep
these five facts separate — establishing one does not establish the others:

- **authoritative baseline revision / config** — the state the restart baseline
  must reflect;
- **revision integrity verification** — proving which revision is authoritative
  and intended;
- **deployment integrity verification** — proving what is actually deployed
  matches that intended state;
- **new commit requirement** — whether a new commit must be created;
- **redeployment requirement** — whether a redeploy action must be performed.

In particular:

> `deployment verification required` ≠ `redeployment required`

Deployment integrity must always be verified before a new freeze. That does not
prove a redeployment action must occur — the environment may already be running
the authoritative state. Derive the redeployment action from the actual
correction and the current deployed state, never from a contamination
classification alone.

**A. Contamination caused by a code/config change deployed into the frozen
environment:**

- **AUTHORITATIVE BASELINE:** the restart baseline must reflect the authoritative
  corrected / post-fix revision and configuration;
- **NEW REPOSITORY REVISION:** `YES` / `NO` / `UNKNOWN` according to what the
  actual correction requires — do not infer `YES` merely from "code/config
  contamination"; a different repository revision is required only if the
  authoritative correction actually requires one;
- **REVISION INTEGRITY:** must be established (the intended authoritative revision
  is known);
- **DEPLOYMENT VERIFICATION:** `REQUIRED` before a new freeze — verify the
  deployed environment against the authoritative corrected revision / config;
- **REDEPLOYMENT:** `YES` / `NO` / `UNKNOWN` according to whether a redeploy
  action is actually necessary to establish the authoritative corrected deployed
  state — do not infer `YES` merely from "code/config contamination"; if whether
  redeployment is necessary has not been established, report
  `redeployment requirement = UNKNOWN / NOT ESTABLISHED`.

Correct next-action wording for this branch: "Use the authoritative corrected
revision/config as the restart baseline and verify the deployed environment
against it. Whether a redeployment action is necessary depends on the actual
correction and the current deployment state." Do not write "code/config
contamination → redeploy".

**B. Contamination caused only by provider-side scenario-state mutation or a
timing deviation (no code/config/deploy change):**

- the same repository revision may remain authoritative;
- restart from a clean, re-established frozen validation baseline;
- redeployment is not automatically required;
- do not require a new Git revision unless another rule or an actual change
  requires one.

If revision identity is still valid and unchanged, report that explicitly
(`repository revision = UNCHANGED / STILL AUTHORITATIVE`) instead of inventing a
new-revision requirement.

This distinction — baseline / verification constraints are separate from the
operational action — holds even inside hypothetical or conditional next-action
branches. A conditional "if the cause turns out to be code/config" branch must
still say "redeploy only if necessary to establish the authoritative corrected
deployed state", never "code/config contamination → redeploy".

### Contaminated / affected test entity handling

This rule applies to the affected scenario's test entity whenever it can no
longer be treated as clean evidence — whether because provider-side scenario
state was contaminated, or because the entity was exercised in a run that later
became `CONTAMINATED` for any cause (including code/config contamination):

- the entity cannot be treated as clean evidence for the affected scenario;
- use a fresh disposable entity, OR an explicitly allowed authoritative
  restoration procedure if the protocol can prove the required starting state and
  all integrity checks;
- if restoration validity is not established, fail closed and use a fresh entity.

A fresh entity may be the default practical choice, but it is not universally
mandatory: do not state that any contamination cause (code/config included)
*always* requires a fresh entity unless authoritative policy explicitly says so.
Do not assert an irreversible "burned forever" rule for the entity unless
authoritative policy establishes it.

## Workflow

1. For a behavior-affecting payment change, read:
   `references/validation-ladder.md`

2. Select the required L1-L3 evidence.

3. If no payment-capable Release Candidate is being promoted (and no hard
   exception applies):
   - L4 is NOT REQUIRED;
   - stop L4 analysis;
   - do not read the freeze protocol.

4. If a payment-capable Release Candidate is being promoted, read:
   `references/evidence-model.md`

5. Apply the Level 4 classification sequence (see "Critical Level 4 rule" below)
   in order. The Release Candidate is only the gate that lets Level 4 be
   considered; it does not by itself make any behavior real-time-only.

6. Decide whether previous valid Level 4 evidence can be reused, and require a
   fresh Level 4 run only if a genuine real-time-only evidence gap remains.

7. Only when Level 4 is actually REQUIRED, read:
   `references/freeze-protocol.md`

8. Keep required evidence separate from evidence that was actually executed.

## Fast routing

| Situation | Expected route |
|---|---|
| Deterministic payment/application behavior | L1 |
| Real DB, ordering, concurrency, transaction behavior | L1 + L2 |
| Real provider API/webhook wire boundary | L1 + L3, plus L2 if applicable |
| Provider event/payload → internal lifecycle-state interpretation explicitly changed (CASE A) | L1 + L3 REQUIRED (`NOT RUN / EVIDENCE MISSING` if unexecuted); do not reopen as UNKNOWN |
| Downstream internal state → entitlement/business-rule mapping only, provider parsing unchanged | L1 (L3 NOT REQUIRED) |
| Webhook/lifecycle-mapping change, whether provider-payload interpretation changed is not established (CASE B) | L3 = UNKNOWN; classify provider-boundary impact first |
| Timing-sensitive change, application-owned clock (injected / controllable `now`) | L1-L3 only |
| Release Candidate, changed timing is application-owned and deterministically controllable | L1-L3 only; L4 NOT REQUIRED for that behavior |
| Release Candidate, ownership of the changed timing not established | classify ownership first; real-time-only = UNKNOWN, L4 = UNKNOWN |
| Release Candidate with a genuine provider-owned real-elapsed-time evidence gap | L1-L3 prerequisites + L4 |
| Billing CSS/copy only | Skill not required |

Higher levels are not automatically better.

## Provider wire boundary (L3 scope)

The "provider wire boundary" is not only the outbound call surface. It includes:

- provider SDK/API calls;
- outbound HTTP request shape;
- webhook signature / authenticity handling;
- real provider payload schema interpretation;
- provider event type → internal lifecycle-state mapping;
- semantic interpretation of provider webhook fields that changes system behavior.

A change to how a real provider event or payload is interpreted into internal
lifecycle state is a provider-wire-boundary change, even when no outbound call
or request shape changed.

### L3 explicit-fact rule (strict)

Explicit provider-boundary facts outrank hypothetical ambiguity. This is the L3
form of "Explicit facts outrank uncertainty" — an established provider-boundary
change is not downgraded to `UNKNOWN` because an internal-only alternative could
be imagined.

If the task, repository, or authoritative evidence explicitly establishes that
any of the following changed:

- provider event type → internal lifecycle-state interpretation;
- real provider payload schema interpretation;
- behavior-affecting provider field semantics;
- other defined provider wire-boundary behavior (authenticity / signature /
  API / SDK boundary);

then the classification is settled:

`L3 = REQUIRED`

If no fresh provider-sandbox evidence has been run for the revision under
review:

`L3 = REQUIRED — NOT RUN / EVIDENCE MISSING`

Once the provider-boundary change is explicitly established, do not say:

- "L3 REQUIRED unless someone argues it is internal";
- "resolve REQUIRED vs. UNKNOWN";
- "classify whether L3 is required".

There is nothing left to classify about *whether* L3 is required. The only
remaining L3 action is to run the required provider-sandbox validation and
record the evidence. Missing execution evidence is `NOT RUN / EVIDENCE MISSING`
against a `REQUIRED` classification — it is never `L3 = UNKNOWN`.

`UNKNOWN` is only for genuinely unresolved boundary ownership — where whether the
provider boundary changed at all is itself not established (see the L3
uncertainty rule below).

- **CASE A — explicit provider-boundary change.** "provider event → internal
  lifecycle-state interpretation changed" (or any other explicitly established
  provider wire-boundary change) → `L3 = REQUIRED` (`NOT RUN / EVIDENCE MISSING`
  if unexecuted). Do not reopen as `UNKNOWN`.
- **CASE B — ambiguous internal change.** "lifecycle mapping changed", but it is
  not established whether provider event / payload interpretation changed →
  `L3 = UNKNOWN`; classify the provider-boundary impact first.

CASE B must not override CASE A.

### L3 uncertainty rule

This rule applies only to CASE B above — where it is genuinely not established
whether the provider boundary changed. It does not reopen a CASE A change that
has already been explicitly established.

If a webhook / lifecycle-mapping change could be either purely internal
downstream mapping (provider payload interpretation unchanged) or part of
provider-payload interpretation (provider event / field semantics → internal
lifecycle-state interpretation changed), and the evidence does not establish
which:

report `L3 = UNKNOWN`.

Do not default to `NOT REQUIRED`. Classify the provider-boundary impact —
including semantic payload interpretation — before deciding. Fail closed: no
PASS while L3 is UNKNOWN.

## Critical Level 4 rule

A normal payment change does NOT trigger L4 merely because:

- it is important;
- it affects subscriptions;
- it affects grace logic;
- it affects reconciliation;
- it affects webhook ordering;
- it is timing-sensitive;
- it touches the provider.

Level 4 is evaluated only when:

- a payment-capable production-intended Release Candidate is actually being promoted; or
- a hard exception defined in `references/evidence-model.md` applies.

### "RC + timing-sensitive" is NOT sufficient for Level 4

Reaching the Release Candidate gate does not make Level 4 required. The RC is
only the gate that allows Level 4 to be *considered*. After the gate, Level 4 is
required only if a genuine real-time-only evidence gap remains.

### Level 4 classification sequence

Apply in order. Stop as soon as a step resolves L4.

1. **RC gate.** Is a payment-capable, production-intended Release Candidate
   actually being promoted, or does a hard exception apply?
   - No → **Step A resolves the decision here.** `fresh L4 = NOT REQUIRED`;
     Step B (real-time-only classification) is `NOT REACHED`. Report these as
     separate facts — do not report `L4 = UNKNOWN`, and do not report a combined
     `L4 = UNKNOWN / NOT REQUIRED`. If real-time ownership was never classified
     because Step B was not reached, say `real-time-only classification = NOT
     REACHED` separately; do not turn that into `L4 = UNKNOWN`. Stop.
   - Yes → continue.

   "No" here means both: no payment-capable, production-intended RC promotion is
   being made, **and** no hard exception (`references/evidence-model.md`, "Hard
   exceptions") is established as applicable. An explicitly stated "no RC
   promotion is being made" is an established `NO` (see "Explicit facts outrank
   uncertainty"), not `UNKNOWN`. Re-evaluate fresh L4 later only if an RC
   promotion is made **or** a hard exception becomes applicable — RC promotion is
   not the only future trigger.

2. **Real-time-only classification.** For each behavior that needs validation,
   establish whether it genuinely depends on REAL ELAPSED TIME and cannot be
   adequately proven with controlled/injected time or deterministic state.
   Classify by behavioral ownership, not by the phrase "timing-sensitive":
   - Application-owned timing whose clock can be injected or deterministically
     controlled (e.g. grace-period expiration driven by an injected `now`, a
     retry/recovery scheduler with a controllable clock) → NOT real-time-only;
     it stays L1/L2-testable even at Release Candidate time.
   - Provider-owned timing where the provider's actual schedule is what must be
     proven (e.g. provider dunning cadence, provider-driven recovery attempts on
     the provider's real schedule, a real billing-period renewal or cancellation
     boundary controlled by the provider) → real-time-only.
   - If the prompt or repository only says something like "grace timing changed"
     or "recovery scheduling changed" but does not establish whether that timing
     is application-owned or provider-owned → mark real-time-only behavior
     `UNKNOWN`. Do not assume provider-driven timing. Resolve ownership first.

   Then:
   - All changed behavior NOT real-time-only → L4 NOT REQUIRED for it. Stop.
   - Real-time-only `UNKNOWN` → L4 `UNKNOWN`; classify ownership/timing before
     proceeding, and fail closed (no PASS) until resolved.
   - Real-time-only YES → continue.

3. **Prior Level 4 evidence.** Can prior valid Level 4 evidence be reused for the
   real-time-only behavior (see `references/evidence-model.md`)?
   - Yes → fresh L4 NOT REQUIRED for this decision (see "Evidence substitution is
     decision / revision / state scoped (strict)" below).
   - No, or zero blast radius cannot be established confidently → fresh L4
     REQUIRED (fail closed).

### Evidence substitution is decision / revision / state scoped (strict)

A successful Step C result — `Evidence substitution = APPLIES`,
`fresh L4 = NOT REQUIRED` — resolves **this** decision, for **this** Release
Candidate revision / state only. It does **not** create a standing or permanent
Level 4 exemption for future revisions, future payment-capable RC decisions, or
future hard-exception decisions.

For every future relevant decision — any future payment-capable,
production-intended Release Candidate, or any future applicable hard-exception
decision — re-run the Level 4 classification sequence from Step A, using the
evidence and the revision / state applicable at that time:

- run Step A again;
- if Step B is reached and `YES`, evaluate Step C against the evidence
  applicable to that future revision / state;
- today's substitution decision (and today's zero-blast-radius demonstration)
  may be cited as historical evidence, but reuse eligibility for the future
  decision must be established again. A later application change can invalidate
  the relevance of today's zero-blast-radius demonstration even when provider
  identity and provider-owned timing behavior are unchanged.

Do not write a closed "fresh L4 only becomes relevant if X / Y / Z happens"
list unless authoritative policy explicitly defines an exhaustive trigger list.

### Level 4 scenario scoping rule

`Level 4 REQUIRED` does not prove that any particular provider-owned lifecycle
behavior exists.

Do not infer or invent:

- provider dunning / retry cadence;
- provider-driven recovery timing;
- provider-owned grace expiration;
- provider-controlled renewal timing;
- provider-controlled cancellation effective-date;
- or any other real-time scenario

unless the prompt, repository, provider contract, or authoritative evidence
establishes that the behavior exists.

For hard exceptions such as the first paid launch:

- L4 may be `REQUIRED` independently of per-behavior real-time-only
  classification;
- the exact L4 scenario set must still be derived from the actual payment
  architecture and real provider behavior;
- if the exact scenario set is not established, report:

  `Level 4 scenario scope = UNKNOWN / TO BE DEFINED`

- fail closed: do not issue `REAL SANDBOX PASS` until the required scenario set
  has been authoritatively defined and executed.

Separate two questions:

- **Is Level 4 REQUIRED?** — answered by the classification sequence above and by
  the hard exceptions in `references/evidence-model.md`.
- **Which exact Level 4 scenarios are REQUIRED?** — answered only from
  authoritative architecture / provider evidence, never from the mere fact that
  Level 4 is required.

### Scenario-scope wording when reuse failed via `zero blast radius = NOT ESTABLISHED`

When fresh L4 is required because the reuse condition failed via
`zero blast radius = NOT ESTABLISHED`, do not describe the required scenario as
"the scenario affected by the change" or "the behavior the change altered" —
positive impact has not been established.

Correct framing:

- authoritatively derive the exact required L4 scenario set from the actual
  payment architecture and real provider behavior;
- the set must account for the previously validated provider-owned behavior
  **associated with** the changed provider-side configuration;
- the actual impact of the configuration delta remains `NOT ESTABLISHED`;
- do not invent additional provider-owned scenarios.

If authoritative scenario policy requires re-validating the relevant previously
covered behavior, state that on the basis of that policy — not by asserting
demonstrated positive impact.

## Evidence authority

Distinguish:

### Required evidence

What should be run.

### Observed evidence

What has actually been executed and whose result is available.

Never issue a PASS merely because:

- tests exist;
- code looks correct;
- documentation says tests are required;
- a checklist exists;
- a test command is mentioned;
- simulated time passed when real elapsed time is the behavior under test.

When evidence has not actually been established, use:

- `REQUIRED`
- `NOT REQUIRED`
- `UNKNOWN`
- `NOT RUN`
- `EVIDENCE MISSING`
- `BLOCKED`

## Verdict vocabulary

Use these only for executed evidence:

- `IMPLEMENTATION PASS`
  - required L1/L2 implementation evidence actually passed.

- `PROVIDER INTEGRATION PASS`
  - a real-provider short run actually demonstrated current provider wire compatibility.

- `REAL SANDBOX PASS`
  - required real-time lifecycle scenarios actually completed under a valid L4 freeze.

- `CONTAMINATED`
  - L4 integrity was lost and the affected observation cannot be represented as a clean pass.

Never treat Level 3 as Level 4.

Never call simulated time "real-time validation."

## Output

When this skill applies, report:

1. Task mode
2. Payment-surface classification
3. L1 — REQUIRED / NOT REQUIRED / UNKNOWN
4. L2 — REQUIRED / NOT REQUIRED / UNKNOWN
5. L3 — REQUIRED / NOT REQUIRED / UNKNOWN
   (during an active freeze where items 3-5 describe a *proposed, undeployed
   fix*, label them as the proposed fix's evidence and keep them out of the
   current frozen run's verdict — see "Proposed-fix evidence isolation (strict)")
6. L4 status:
   - If no L4 freeze is active: `L4 — REQUIRED / NOT REQUIRED / UNKNOWN`
     (a fresh L4 run requirement).
   - If an L4 freeze is already active (mid-freeze decision), report BOTH as
     separate fields, never a generic `L4 = REQUIRED`:
     - `current L4 run status` — `ACTIVE / IN PROGRESS` / `BLOCKED` /
       `CONTAMINATED` / `COMPLETED CLEANLY`;
     - `fresh L4 restart` — `NOT TRIGGERED` / `CONDITIONAL / NOT YET TRIGGERED` /
       `REQUIRED`.
     Do not report `fresh L4 restart = REQUIRED` until contamination has actually
     occurred — see "Mid-freeze state-transition rule (strict)" and "Current
     Level 4 run status vs. fresh Level 4 restart requirement (strict)"; before
     deployment report `current L4 run status = ACTIVE / IN PROGRESS` (with
     operational `BLOCKED` if established) and
     `fresh L4 restart = CONDITIONAL / NOT YET TRIGGERED`.
7. RC promotion status — YES / NO / UNKNOWN / NOT ESTABLISHED
   (report `NO` when the prompt explicitly establishes that no payment-capable,
   production-intended RC promotion is being made — an explicit `NO` is not
   downgraded to `UNKNOWN` because release chronology is otherwise unknown;
   report `UNKNOWN / NOT ESTABLISHED` only when the current decision genuinely
   does not establish it and no specific hard exception is established; an active
   L4 freeze alone does not settle this — see "Anti-inference rule (strict)" and
   "Explicit facts outrank uncertainty")
8. Evidence substitution — APPLIES / DOES NOT APPLY / NOT RELEVANT / UNKNOWN
   (see "Evidence-substitution status rule" below)
9. Real-time-only behavior / Level 4 scenario scope
   (the REQUIRED scenario set, or `UNKNOWN / TO BE DEFINED`)
10. Freeze status (freeze integrity = `NOT CONTAMINATED` / `CONTAMINATED`; and,
    for an already-active freeze, the current L4 run status and fresh L4 restart
    status from item 6)
11. Observed evidence / verdict
    (during an active freeze, report the current frozen run and any proposed
    undeployed fix as separate evidence subjects; the current run's
    `REAL SANDBOX PASS` blocker is the required scenario result alone — do not
    fold in an undeployed fix's L1/L2/L3 state; see "Proposed-fix evidence
    isolation (strict)")
12. Next actions
    (validation next actions only; do not state a release / promotion /
    deployment disposition — "Promote the RC", "release approved", "deploy now" —
    unless release authorization is explicitly in scope and all authoritative
    release gates are established; see "Validation sufficiency vs. release
    authorization (strict)")

Keep the answer concise and actionable.

### Evidence-substitution status rule

Evidence substitution belongs to Step C of the Level 4 classification sequence.
Report its status according to whether Step C was actually reached:

- If the Level 4 decision stops before evidence substitution is evaluated —
  Step A resolves L4 (no RC / hard exception), or Step B resolves L4
  (real-time-only behavior = NO, or = UNKNOWN) — report
  `Evidence substitution = NOT RELEVANT`.
- Report `APPLIES` or `DOES NOT APPLY` only when Step C was reached, i.e.
  Step B = YES (genuinely real-time-only behavior requires evidence coverage).
- Never report `APPLIES` / `DOES NOT APPLY` for a branch that was not reached.
- Exception — mid-freeze contamination transition: once an actual
  freeze-integrity-breaking event has occurred (including provider-side scenario
  state changed outside the validation plan), the affected run is `CONTAMINATED`,
  the contamination-restart hard exception is `ACTIVE`, and
  `Evidence substitution = DOES NOT APPLY` — not `NOT RELEVANT`. See "Mid-freeze
  state-transition rule (strict)". Before any such event has occurred, the
  mid-freeze decision reports `Evidence substitution = NOT RELEVANT`.

If the substitution prerequisites happen to look satisfied but Step C was never
reached, an optional note is allowed — "Substitution prerequisites may appear
satisfied, but substitution was not needed for this decision." — but the formal
status must remain `NOT RELEVANT`. Do not write that substitution "applies
anyway" or "independently confirms" the decision.

## Final safety checks

Before finishing verify:

- L4 was not triggered just because a change is important or timing-sensitive;
- "RC + timing-sensitive" was not by itself treated as sufficient for L4;
- application-owned, deterministically controllable timing was not classified as
  real-time-only;
- unestablished timing ownership was marked `UNKNOWN`, not assumed provider-driven;
- no PASS was inferred from static inspection;
- L3 and L4 were not conflated;
- a webhook / lifecycle-mapping change was classified `L3 NOT REQUIRED` only
  after confirming provider-payload interpretation (schema, provider event type
  → internal lifecycle-state mapping, provider field semantics) is unchanged;
  otherwise it was reported `L3 UNKNOWN`, not defaulted to `NOT REQUIRED`;
- where the task/repository explicitly established a provider wire-boundary
  change (provider event type → internal lifecycle-state interpretation, real
  provider payload schema interpretation, behavior-affecting provider field
  semantics, or authenticity / signature / API / SDK boundary), `L3 = REQUIRED`
  was reported and NOT downgraded to `UNKNOWN`; unexecuted evidence was reported
  as `L3 = REQUIRED — NOT RUN / EVIDENCE MISSING`, never as `L3 = UNKNOWN`, and
  the response did not say "REQUIRED unless argued internal" or "resolve
  REQUIRED vs. UNKNOWN" after that fact was established (CASE A vs. CASE B);
- uncertainty affecting Level 4 reuse failed closed;
- evidence substitution was reported `NOT RELEVANT` whenever the Level 4
  decision stopped before Step C (evidence substitution) was reached, and was
  reported `APPLIES` / `DOES NOT APPLY` only when Step B = YES;
- `Level 4 REQUIRED` (including via a hard exception such as first paid launch)
  was not used to infer or invent specific provider-owned real-time scenarios
  (dunning / retry cadence, provider-driven recovery timing, provider-owned grace
  expiry, renewal timing, cancellation effective-date, or any other); where the
  exact scenario set was not established from authoritative architecture /
  provider evidence it was reported `Level 4 scenario scope = UNKNOWN / TO BE
  DEFINED` and no `REAL SANDBOX PASS` was issued;
- no missing fact was filled by implication from a compatible validation state —
  in particular an active L4 freeze was not treated as proof of `RC = YES`, and
  RC status / hard-exception identity not explicitly established were reported
  `UNKNOWN / NOT ESTABLISHED`;
- an aggregate conclusion (`no hard exception applies`) was reported as
  `hard exception applicability = NONE APPLICABLE` and NOT decomposed into
  unsupported component facts (`first paid launch = NO`, `provider migration =
  NO`, `prior contamination restart = NO`); each such predicate not independently
  established was left `UNKNOWN / NOT ESTABLISHED`, and an applicable hard
  exception was not treated as identifying which one applied;
- `zero blast radius = NOT ESTABLISHED` was kept distinct from `zero blast radius
  = NO` and was not transformed into `affected / in blast-radius path = YES` or
  any assertion of positive impact; it led to `Evidence substitution = DOES NOT
  APPLY` / `fresh L4 = REQUIRED` by failing closed, not by asserting demonstrated
  impact; structural relevance (a provider-side configuration participates in /
  governs / configures the covered behavior) was not converted into
  configuration-delta impact, and where positive impact was not established the
  covered behavior and its L4 scenario were not described as "affected by",
  "impacted by", "touched by", or "changed by" the configuration change — neutral
  wording ("associated with the changed configuration", "governed in part by that
  provider-side configuration") was used instead;
- the zero-blast-radius requirement was described as a result to be
  authoritatively demonstrated, not as one exclusive review mechanism; an
  independent blast-radius review was offered as a possible method only, unless
  authoritative policy mandated it;
- for a mid-freeze decision, proposed vs. deployed change, contamination, and
  restart requirement were not collapsed: `fresh L4 restart = REQUIRED`,
  `CONTAMINATED`, and `Evidence substitution = DOES NOT APPLY` were not reported
  before an actual freeze-integrity-breaking event occurred; before any such
  event the run was reported `current L4 run status = ACTIVE / IN PROGRESS`
  (operationally `BLOCKED` if established) with
  `fresh L4 restart = CONDITIONAL / NOT YET TRIGGERED` and
  `Evidence substitution = NOT RELEVANT`; once an actual contamination event had
  occurred (including provider-side scenario state changed outside the validation
  plan) the affected run was reported `CONTAMINATED` with
  `contamination-restart hard exception = ACTIVE`, `fresh L4 restart = REQUIRED`,
  and `Evidence substitution = DOES NOT APPLY` — never `NOT RELEVANT`;
- for an already-active L4 freeze, a generic `L4 = REQUIRED` was not used to
  collapse the current active run with a fresh-restart requirement: the current
  L4 run status (`ACTIVE / IN PROGRESS`, operationally `BLOCKED` if established,
  before any contamination event) and the fresh L4 restart status
  (`CONDITIONAL / NOT YET TRIGGERED` before contamination; `REQUIRED` only after
  an actual contamination event) were reported as separate fields;
- an undeployed proposed fix's L1 / L2 / L3 evidence state was not attached to
  the current frozen run's observed evidence or verdict: the current run's
  inability to issue `REAL SANDBOX PASS` was stated as caused by the required
  scenario result alone, not "and L3 is UNKNOWN" where that L3 belongs only to
  the proposed fix — see "Proposed-fix evidence isolation (strict)";
- a restart after code/config contamination did not state that a fresh
  disposable entity is automatically mandatory: the affected scenario's entity
  was handled as "fresh disposable entity OR authoritative restoration procedure
  that proves the required starting state and all integrity checks; fail closed
  to a fresh entity only if restoration validity is not established";
- a `CONTAMINATED` verdict was not used to infer restart mechanics: a new commit,
  a different repository revision, a code change, a config change, or a redeploy
  were required only when the actual contamination cause (code/config deployed
  into the frozen environment) required them; for provider-side-only
  contamination with an unchanged, still-authoritative revision that was reported
  explicitly rather than inventing a new-revision requirement; and for the
  affected scenario's test entity, no contamination cause (code/config included)
  was treated as automatically mandating a fresh entity — the fresh-disposable-
  entity-OR-authoritative-restoration rule was applied, failing closed to a fresh
  entity only when restoration validity is not established;
- even for an established code/config contamination cause, the authoritative
  baseline revision/config, revision-integrity verification, and deployment-
  integrity verification were kept separate from the operational actions (new
  commit, new repository revision, redeployment): `deployment verification
  required` was not read as `redeployment required`, a new repository revision
  and a redeploy action were reported `YES` only when the actual correction /
  current deployed state required them (else `NO` / `UNKNOWN / NOT ESTABLISHED`),
  and no conditional or hypothetical next-action branch was allowed to say
  "code/config contamination → redeploy" — it said "use the authoritative
  corrected revision/config as the baseline, verify the deployed environment
  against it, and redeploy only if necessary to establish that state";
- a validation result (`Evidence substitution = APPLIES`, `fresh L4 = NOT
  REQUIRED`, `L1/L2/L3 PASS`) was not turned into a release / promotion /
  deployment authorization; where release authorization was not established the
  answer stopped at the validation conclusion and no "Promote the RC" style
  imperative was written;
- a successful Step C substitution result was scoped to the current decision /
  revision / state and NOT stated as a permanent Level 4 exemption; future
  relevant RC or hard-exception decisions were told to re-enter at Step A, with
  no closed "only if X / Y / Z" future-trigger list;
- a stated present-tense provider fact ("provider is unchanged") was reported as
  stated and NOT expanded into the historical `provider migration`
  hard-exception predicate = `NO`; where the task said `no hard exception
  applies` the answer reported `hard exception applicability = NONE APPLICABLE`
  without decomposing component predicates unless one was independently needed
  and established;
- for an active-L4 execution / status review, established run-state facts were
  not expanded into unestablished execution details: proposed-fix
  existence/absence and test-entity provisioning/preparation state were reported
  `UNKNOWN / NOT ESTABLISHED` (or omitted when not needed), never inferred from
  the clean active freeze; `fresh L4 restart = NOT TRIGGERED` was justified only
  by "no contamination event has occurred", with no unrelated negative facts
  appended; and the pending-scenario next action did not name a "prepared
  disposable test entity" (or any other specific execution resource) unless its
  provisioning was explicitly established;
- the next required action is explicit.