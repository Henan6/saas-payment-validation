# Example — contamination and restart (provider-side only)

> Fictional scenario. No real company, provider, or system is described.

## Situation

A Level 4 freeze is active. A pinned repository revision is deployed and verified;
disposable test entities are running through the required real-time scenarios.

Partway through the observation window, an operator manually advances one test
subscription's state on the **provider side** — outside the validation plan — to
"move things along". No repository revision, no deployed revision, no application
configuration, and no deployment changed. Only provider-side scenario state was
mutated.

## What this is

**Provider-state integrity** (integrity check 4) can no longer be established for
the affected scenario. That is an actual freeze-integrity-breaking event.

| Field | Value |
|---|---|
| Freeze integrity | **CONTAMINATED** (provider-side scenario state changed outside the validation plan) |
| Current affected L4 run | CONTAMINATED |
| Contamination-restart hard exception | ACTIVE |
| **Fresh L4 restart** | **REQUIRED** |
| **Evidence substitution** | **DOES NOT APPLY** (a hard-exception restart is not eligible for substitution) |
| RC promotion status | UNKNOWN / NOT ESTABLISHED (an active freeze does not establish it; not relevant to the contamination decision) |

A scenario producing an unexpected result would **not** be contamination — that
would be a `scenario FAIL` under intact integrity. Here the cause is an actual
integrity break, not an observed result.

## Restart mechanics — driven by the actual cause

The `CONTAMINATED` verdict by itself determines only the three rows above. The
restart mechanics follow the **actual contamination cause**, which here is
**provider-side only**.

| Restart question | Answer | Why |
|---|---|---|
| Repository revision | **UNCHANGED / STILL AUTHORITATIVE** | No code, config, or deployment changed. The same revision remains authoritative |
| New commit required | NO | Nothing in the code needs correcting |
| New repository revision required | NO | The contamination cause does not require one |
| **Deployment verification** | **REQUIRED** before the new freeze | Every Entry integrity check is re-run before restart — revision, deployment, change, and provider-state integrity must all be re-established |
| **Redeployment action** | **NOT AUTOMATICALLY REQUIRED** | The environment may already be running the authoritative state. `deployment verification required` ≠ `redeployment required` |
| Provider-side scenario state | Must be re-established clean before restart | This is the condition that was broken |
| Affected scenario's test entity | **Fresh disposable entity OR an authoritative restoration procedure** that can prove the required starting state and all integrity checks; fail closed to a fresh entity if restoration validity is not established | The entity can no longer be treated as clean evidence for the affected scenario |

Notes:

- A fresh entity may be the practical default, but it is **not universally
  mandatory** — an authoritative restoration procedure is equally valid when it
  can prove the required starting state and all integrity checks. No "burned
  forever" rule is asserted.
- A restored entity does **not** by itself restore the contaminated run's
  evidence. The affected real-time validation is still restarted from a clean,
  explicitly re-established frozen baseline.
- There is no code/config fix here, so there is **no fix to re-classify** through
  L1–L3 before re-entry. Re-enter once the clean provider-side baseline and all
  four integrity checks are re-established.

## Next actions

1. Record the contamination event and its cause (provider-side scenario-state
   mutation outside the validation plan).
2. Re-establish clean provider-side scenario state for the affected scenario.
3. Re-run all Entry integrity checks: confirm revision integrity (unchanged,
   still authoritative), verify deployment integrity against that revision,
   confirm change integrity, and confirm provider-state integrity.
4. Restart the affected real-time validation from the clean re-established
   baseline, using a fresh disposable entity or a valid authoritative restoration
   procedure.
5. Redeploy only if step 3 shows the deployed environment does not match the
   authoritative revision.
