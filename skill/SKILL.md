---
name: saas-payment-validation
description: Plan, review, or audit engineering validation for behavior-affecting SaaS payment code and payment-capable releases. Use whenever a change touches checkout, billing state, subscriptions, entitlements, payment webhooks, reconciliation, provider APIs, payment persistence/concurrency, or release-candidate payment lifecycle behavior — even if the user does not use the word "validation". Use it to choose L1-L4 evidence, decide whether Level 4 is actually required, and keep real-time validation evidence trustworthy. Do not use for billing copy/CSS, pricing strategy, provider selection, or general payment questions with no engineering-validation decision in them.
---

# SaaS Payment Validation

Choose the **minimum sufficient** validation for a payment change without weakening correctness.

L1-L3 are the normal engineering loop. L4 is a release-candidate mechanism, not a stronger test you reach for when a change feels scary. Higher levels are not automatically better.

## The ladder

| Level | Proves | Needs |
| --- | --- | --- |
| **L1** | State machines, lifecycle mapping, entitlement decisions, parsing, duplicate-event handling, timestamp boundaries under controlled time | Nothing external. No DB, provider, network, or real time |
| **L2** | Persistence, transactions, concurrency, ordering, conflict resolution, reconciliation races | Real isolated test DB; provider boundary still faked |
| **L3** | Provider SDK/API calls, request shape, webhook authenticity, payload schema interpretation, provider-event → internal-lifecycle mapping | A real provider sandbox; a pinned revision |
| **L4** | Behavior only real elapsed time against a real provider can demonstrate | A pinned revision, a deployed frozen environment, disposable test entities, an intact freeze |

**Before routing anything to L4, check whether the provider offers a time-simulation facility** — a sandbox feature that advances test-mode time to trigger renewals, trial ends, dunning cadence, and retry schedules (Stripe Test Clocks is one example). Where such a facility exists and covers the behavior under test, that behavior is **L3-provable and not real-time-only**. Route to L4 only for behavior that no available simulation facility can reach.

## Fast routing

| Situation | Route |
| --- | --- |
| Deterministic application behavior | L1 |
| Real DB, ordering, concurrency, transactions | L1 + L2 |
| Real provider API/webhook wire boundary | L1 + L3 (+ L2 if applicable) |
| Provider event/payload → internal lifecycle interpretation explicitly changed | L3 = REQUIRED. If not yet executed: `REQUIRED — NOT RUN / EVIDENCE MISSING`. Do not reopen as UNKNOWN |
| Downstream internal state → entitlement mapping only, provider parsing unchanged | L1 (L3 NOT REQUIRED) |
| Lifecycle-mapping change, provider-payload interpretation impact not established | L3 = UNKNOWN. Classify the provider boundary first |
| Timing-sensitive, application-owned clock (injectable `now`) | L1-L3 only |
| Timing-sensitive, provider-owned schedule, provider time simulation available | L1-L3 only |
| RC + genuine provider-owned real-elapsed-time gap, no simulation available | L1-L3 prerequisites + L4 |
| Billing CSS/copy only | Skill not required |

## Workflow

1. Classify the change: what payment behavior changed, what the blast radius may reach, whether the provider wire boundary changed, whether correctness depends on real persistence/concurrency, and what evidence has actually been executed.
2. Read `references/validation-ladder.md` and select the required L1-L3 evidence.
3. Run the Level 4 classification sequence below. If Step A resolves it, stop — do not read the freeze protocol.
4. If Level 4 is genuinely required, read `references/freeze-protocol.md`.
5. Report using the output contract at the end of this file.

Read `references/reporting-rules.md` when a decision is mid-freeze, involves evidence reuse, or when you are unsure how to report a partially-established fact. That file holds the detailed state distinctions; the rules below are the ones that apply to every decision.

## Core rules

**Fail closed.** If a fact is not established by the prompt, the repository, authoritative evidence, or explicit task context, report `UNKNOWN / NOT ESTABLISHED`. Never invent test results, release status, provider behavior, or prior evidence. Uncertainty that affects L4 reuse forces a fresh L4 run.

**Do not infer from a compatible state.** A state that would be consistent with a fact does not establish that fact. An active L4 freeze does not prove `RC = YES`. `L4 = REQUIRED` does not identify which hard exception applies. A prior sandbox PASS does not establish chronology. Repository presence of reconciliation code does not prove L2 is required for this change.

**Explicit facts outrank uncertainty.** `UNKNOWN` is for genuinely missing information, not for hedging. If the task explicitly says no RC promotion is being made, report `RC promotion status = NO` — do not downgrade an explicit NO to UNKNOWN because adjacent history is unknown. Keep the current decision's facts separate from historical facts, which may independently stay UNKNOWN.

**Aggregates do not establish their components.** "No hard exception applies" is reported as `hard exception applicability = NONE APPLICABLE`. It does not establish `first paid launch = NO` or `provider migration = NO` individually. Report each component `UNKNOWN / NOT ESTABLISHED` unless independently established.

**Required evidence is not observed evidence.** Never issue a PASS because tests exist, code looks correct, a checklist exists, a command was mentioned, docs say tests are required, or simulated time passed where real elapsed time is the behavior under test. Track what should run separately from what actually ran.

**Some facts have three states, not two.** Zero blast radius is `YES` (demonstrated), `NO` (positive impact demonstrated), or `NOT ESTABLISHED` (insufficient evidence). Both `NO` and `NOT ESTABLISHED` fail the reuse condition closed, but they are different facts. Never convert `NOT ESTABLISHED` into an assertion of actual impact. Where impact is not established, use neutral wording — "behavior associated with the changed configuration", not "behavior affected by the change".

**A validation verdict is not release authorization.** `fresh L4 = NOT REQUIRED` or `L1/L2/L3 PASS` never implies "promote", "ship", or "deploy". State a release disposition only when release scope is explicit and every authoritative release gate is established. Otherwise stop at the validation conclusion.

## Level 4 classification sequence

Apply in order. Stop as soon as a step resolves the decision.

**Step A — RC gate.** Is a payment-capable, production-intended Release Candidate actually being promoted, or does a hard exception apply (`references/evidence-model.md`)?

- No → `fresh L4 = NOT REQUIRED`; `real-time-only classification = NOT REACHED`. Report those as two separate facts. Do not report `L4 = UNKNOWN`. Stop.
- Yes → continue. The RC is only the gate that lets L4 be *considered*. It does not make anything real-time-only.

**Step B — real-time-only classification.** For each behavior needing validation, classify by ownership, not by the phrase "timing-sensitive":

- Application-owned timing with an injectable or deterministically controllable clock → not real-time-only. Stays L1/L2-testable even at RC time.
- Provider-owned timing that an available provider time-simulation facility can reproduce → not real-time-only. Route to L3.
- Provider-owned timing whose real schedule is the thing under test and cannot be simulated → real-time-only.
- Ownership not established → `real-time-only = UNKNOWN`. Resolve ownership first; fail closed until resolved. Do not assume provider-driven.

All changed behavior not real-time-only → L4 NOT REQUIRED. Stop. Otherwise continue.

**Step C — prior evidence reuse.** Can valid prior L4 evidence cover the real-time-only behavior (`references/evidence-model.md`)?

- Yes → `Evidence substitution = APPLIES`, fresh L4 NOT REQUIRED **for this decision, this revision, this state only**. This is not a standing exemption; re-enter at Step A for every future RC or hard-exception decision.
- No, or zero blast radius not established → `DOES NOT APPLY`, fresh L4 REQUIRED.

**Scenario scope is a separate question.** "Is L4 required" and "which L4 scenarios are required" have different answers from different sources. Never infer specific provider-owned behavior (dunning cadence, recovery timing, grace expiry, renewal or cancellation boundaries) from the mere fact that L4 is required. If the scenario set is not established from authoritative architecture or provider evidence, report `Level 4 scenario scope = UNKNOWN / TO BE DEFINED` and issue no `REAL SANDBOX PASS`.

## Mid-freeze decisions

When a freeze is already active, "L4" is not one field. Report two:

- **Current L4 run status** — `ACTIVE / IN PROGRESS` / `BLOCKED` (only if continuation is established as prevented) / `CONTAMINATED` / `COMPLETED CLEANLY`
- **Fresh L4 restart** — `NOT TRIGGERED` / `CONDITIONAL / NOT YET TRIGGERED` / `REQUIRED`

Three facts stay separate: scenario outcome, freeze integrity, and operational run state. A scenario FAIL under an intact freeze is valid L4 evidence of a failure — it is not contamination and does not by itself block the run. Contamination requires a separate actual integrity-breaking event, and a proposed change is not a deployed one.

An undeployed proposed fix's L1/L2/L3 states belong to that fix, not to the frozen run's verdict.

For contamination causes, restart mechanics, entity handling, and the `deployment verification ≠ redeployment` distinction, read `references/freeze-protocol.md` and `references/reporting-rules.md`.

## Verdict vocabulary

Use only for evidence that actually executed:

- `IMPLEMENTATION PASS` — required L1/L2 evidence passed
- `PROVIDER INTEGRATION PASS` — a real-provider run demonstrated current wire compatibility
- `REAL SANDBOX PASS` — required real-time scenarios completed under a valid L4 freeze
- `CONTAMINATED` — L4 integrity was lost; the observation cannot be represented as a clean pass

For evidence not established: `REQUIRED` / `NOT REQUIRED` / `UNKNOWN` / `NOT RUN` / `EVIDENCE MISSING` / `BLOCKED`.

Never treat L3 as L4. Never call simulated time "real-time validation" — but do not require real time where simulation is a valid substitute for the behavior under test.

## Output

Report these, concisely:

1. **Payment surface** — what changed, and the blast radius that is established
2. **L1 / L2 / L3** — REQUIRED / NOT REQUIRED / UNKNOWN, with `NOT RUN / EVIDENCE MISSING` where the classification is settled but execution is not
3. **L4** — a single `fresh L4` status when no freeze is active; **both** current run status and fresh restart status when one is
4. **RC promotion status** — YES / NO / UNKNOWN
5. **Evidence substitution** — APPLIES / DOES NOT APPLY / NOT RELEVANT. Use `NOT RELEVANT` whenever the decision stopped before Step C
6. **Level 4 scenario scope** — the required set, or `UNKNOWN / TO BE DEFINED`
7. **Observed evidence and verdict** — what actually ran, and what it showed
8. **Next validation actions** — explicit, and validation-only

Omit fields that are not needed for the decision rather than padding them with UNKNOWN. Where a field is needed but unestablished, `UNKNOWN / NOT ESTABLISHED` is the answer.
