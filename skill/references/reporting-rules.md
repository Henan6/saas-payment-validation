# Reporting and State Rules

This file holds the detailed state-distinction and reporting rules referenced
from `SKILL.md`. `SKILL.md` carries the rules that apply to every
payment-validation decision; the rules here are the ones that only matter in
particular situations, so they were moved out to keep the main file short. Read
this file when a decision is mid-freeze, when it turns on whether prior Level 4
evidence can be reused, or when you need to report a fact that is only partially
established. Nothing here is new: every rule below was previously part of
`SKILL.md`, and its wording is unchanged apart from heading levels and
cross-reference pointers updated to name this file.

## Contents

- [Anti-inference and fact establishment](#anti-inference-and-fact-establishment)
  - [Anti-inference rule (strict)](#anti-inference-rule-strict)
    - [Explicit facts outrank uncertainty](#explicit-facts-outrank-uncertainty)
    - [Aggregate facts do not establish their component facts (strict)](#aggregate-facts-do-not-establish-their-component-facts-strict)
- [Zero blast radius and configuration-delta impact](#zero-blast-radius-and-configuration-delta-impact)
  - [Zero blast radius is a three-state fact (strict)](#zero-blast-radius-is-a-three-state-fact-strict)
    - [Configuration relevance vs. configuration-delta impact (strict)](#configuration-relevance-vs-configuration-delta-impact-strict)
  - [Evidence requirements specify the result, not an exclusive mechanism (strict)](#evidence-requirements-specify-the-result-not-an-exclusive-mechanism-strict)
  - [Scenario-scope wording when reuse failed via `zero blast radius = NOT ESTABLISHED`](#scenario-scope-wording-when-reuse-failed-via-zero-blast-radius--not-established)
- [Evidence substitution scope and status](#evidence-substitution-scope-and-status)
  - [Evidence substitution is decision / revision / state scoped (strict)](#evidence-substitution-is-decision--revision--state-scoped-strict)
  - [Evidence-substitution status rule](#evidence-substitution-status-rule)
- [Mid-freeze state transitions](#mid-freeze-state-transitions)
  - [Mid-freeze state-transition rule (strict)](#mid-freeze-state-transition-rule-strict)
  - [Current Level 4 run status vs. fresh Level 4 restart requirement (strict)](#current-level-4-run-status-vs-fresh-level-4-restart-requirement-strict)
  - [Active-L4 execution / status review: do not expand known run-state into unestablished detail (strict)](#active-l4-execution--status-review-do-not-expand-known-run-state-into-unestablished-detail-strict)
    - [Next-action wording when a required scenario is pending](#next-action-wording-when-a-required-scenario-is-pending)
  - [Proposed-fix evidence isolation (strict)](#proposed-fix-evidence-isolation-strict)
  - [Scenario-result state rule (strict)](#scenario-result-state-rule-strict)
- [Restart mechanics after contamination](#restart-mechanics-after-contamination)
  - [Restart-mechanics anti-inference rule (strict)](#restart-mechanics-anti-inference-rule-strict)
    - [Baseline requirement vs. operational action (strict)](#baseline-requirement-vs-operational-action-strict)
    - [Contaminated / affected test entity handling](#contaminated--affected-test-entity-handling)
- [Validation verdict vs. release authorization](#validation-verdict-vs-release-authorization)
  - [Validation sufficiency vs. release authorization (strict)](#validation-sufficiency-vs-release-authorization-strict)
- [Known redundancies](#known-redundancies)

## Anti-inference and fact establishment

### Anti-inference rule (strict)

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

#### Explicit facts outrank uncertainty

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
resolves Step A of the Level 4 classification sequence (SKILL.md): `fresh L4 = NOT REQUIRED`,
and Step B is `NOT REACHED`. If real-time ownership was never classified because
Step B was not reached, report `real-time-only classification = NOT REACHED`
separately. Do not report `L4 = UNKNOWN`, and do not report a combined
`L4 = UNKNOWN / NOT REQUIRED`, for that decision.

#### Aggregate facts do not establish their component facts (strict)

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

## Zero blast radius and configuration-delta impact

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

## Evidence substitution scope and status

### Evidence substitution is decision / revision / state scoped (strict)

A successful Step C result — `Evidence substitution = APPLIES`,
`fresh L4 = NOT REQUIRED` — resolves **this** decision, for **this** Release
Candidate revision / state only. It does **not** create a standing or permanent
Level 4 exemption for future revisions, future payment-capable RC decisions, or
future hard-exception decisions.

For every future relevant decision — any future payment-capable,
production-intended Release Candidate, or any future applicable hard-exception
decision — re-run the Level 4 classification sequence (SKILL.md) from Step A, using the
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

### Evidence-substitution status rule

Evidence substitution belongs to Step C of the Level 4 classification sequence (SKILL.md).
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
  state-transition rule (strict)" in this file. Before any such event has occurred, the
  mid-freeze decision reports `Evidence substitution = NOT RELEVANT`.

If the substitution prerequisites happen to look satisfied but Step C was never
reached, an optional note is allowed — "Substitution prerequisites may appear
satisfied, but substitution was not needed for this decision." — but the formal
status must remain `NOT RELEVANT`. Do not write that substitution "applies
anyway" or "independently confirms" the decision.

## Mid-freeze state transitions

### Mid-freeze state-transition rule (strict)

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
Level 4 run status vs. fresh Level 4 restart requirement (strict)" in this file.

Do not report `fresh L4 restart = REQUIRED`, `CONTAMINATED`, or
`Evidence substitution = DOES NOT APPLY` before the event that triggers
contamination has actually occurred. Per the anti-inference rule, do not assume a
proposed redeploy — or any other proposed integrity-breaking action — has already
happened. Conversely, once contamination has actually occurred, do not describe
evidence substitution as `NOT RELEVANT`.

### Current Level 4 run status vs. fresh Level 4 restart requirement (strict)

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

### Active-L4 execution / status review: do not expand known run-state into unestablished detail (strict)

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

#### Next-action wording when a required scenario is pending

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

### Proposed-fix evidence isolation (strict)

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

### Scenario-result state rule (strict)

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
  isolation (strict)" in this file;
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

## Restart mechanics after contamination

### Restart-mechanics anti-inference rule (strict)

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

#### Baseline requirement vs. operational action (strict)

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

#### Contaminated / affected test entity handling

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

## Validation verdict vs. release authorization

### Validation sufficiency vs. release authorization (strict)

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

## Known redundancies

These sections state the same rule in different words. They are kept separate on
purpose (this file loads on demand, and merging mid-freeze rules risks changing
their meaning). If you edit one, check the other so they do not drift apart.

1. **"Mid-freeze state-transition rule (strict)"** ↔ **"Current Level 4 run
   status vs. fresh Level 4 restart requirement (strict)"** — both enumerate the
   full before-/after-contamination field set (`ACTIVE / IN PROGRESS`, `BLOCKED`,
   `CONDITIONAL / NOT YET TRIGGERED`, `CONTAMINATED`, `REQUIRED`, hard exception
   `ACTIVE`/`NOT ACTIVE`, `Evidence substitution NOT RELEVANT`/`DOES NOT APPLY`),
   one as a temporal list and one as "fields A/B + rules".
2. **"Current Level 4 run status vs. fresh Level 4 restart requirement (strict)"**
   ↔ **"Scenario-result state rule (strict)"** — both state that a `scenario
   FAIL` alone does not trigger a fresh restart, does not by itself mean
   `BLOCKED`, and that a generic `L4 = REQUIRED` must not collapse "current run"
   with "fresh restart".
3. **"Proposed-fix evidence isolation (strict)"** ↔ **"Scenario-result state
   rule (strict)"** — both state that an undeployed proposed fix's L1/L2/L3
   states do not attach to the current frozen run's verdict, and that the run's
   inability to issue `REAL SANDBOX PASS` is explained by the required scenario
   result alone. Each keeps its own worked example.
4. **"Mid-freeze state-transition rule (strict)"** (after-contamination block) ↔
   **"Evidence-substitution status rule"** (the "Exception — mid-freeze
   contamination transition" bullet) — both give `CONTAMINATED` → `Evidence
   substitution = DOES NOT APPLY` (not `NOT RELEVANT`), and `NOT RELEVANT` before
   the event.
5. **"Anti-inference rule (strict)"** ("`L4 REQUIRED` does NOT prove a particular
   hard exception applies") ↔ **"Aggregate facts do not establish their
   component facts (strict)"** ("an established *applicable* hard exception does
   not automatically identify **which** hard exception applies") — the same
   "applicable ≠ identified" point.

Canonical homes for two rules that are echoed in passing elsewhere: the full
banned / neutral wording list lives in **"Configuration relevance vs.
configuration-delta impact (strict)"**; the "a validation requirement specifies
the result, not one mandated mechanism" rule lives in **"Evidence requirements
specify the result, not an exclusive mechanism (strict)"**. Sections that touch
those points in their own context (for example "Scenario-scope wording when reuse
failed via `zero blast radius = NOT ESTABLISHED`") should defer to those two.
