# Example — evidence substitution

> Fictional scenario. No real company, provider, or system is described.

Evidence substitution is the question of whether a **prior valid `REAL SANDBOX
PASS`** can stand in for a fresh Level 4 run on a new Release Candidate. It is
evaluated only inside Step C of the Level 4 classification sequence — i.e. only
when Step A (RC gate) is met and Step B has classified genuinely real-time-only
behavior.

The four conditions, all of which must hold:

1. current L2 evidence covers the timing-sensitive application behavior changed
   since the previous valid L4 run;
2. current provider-boundary evidence is valid, including fresh L3 where
   required;
3. a previous `REAL SANDBOX PASS` exists;
4. the difference between the previously validated revision and the new Release
   Candidate has **zero blast radius** into the real-time-sensitive behavior that
   the previous L4 evidence covered.

---

## Case 1 — zero blast radius NOT ESTABLISHED → substitution DOES NOT APPLY

### Situation

- A payment-capable, production-intended Release Candidate is being promoted.
- The changed behavior under test is a **provider-owned** dunning-retry schedule
  (the provider runs retries on its own real schedule; that schedule is the thing
  under test) → Step B = YES, real-time-only.
- A previous valid `REAL SANDBOX PASS` exists for an earlier revision that
  covered this dunning behavior.
- The RC changes a provider-side retry configuration value. That configuration
  **participates in** the provider's retry scheduling, but the effect of the
  changed value on the covered real-time behavior has **not** been
  authoritatively established.

### Classification

| Field | Value |
|---|---|
| RC promotion status | YES |
| Step A | RC gate met → continue |
| Step B | Real-time-only = YES (provider-owned dunning schedule) → continue to Step C |
| Condition 3 (prior `REAL SANDBOX PASS`) | Satisfied |
| Zero blast radius | **NOT ESTABLISHED** — the configuration participates in the behavior, but the delta's impact is not demonstrated either way |
| **Evidence substitution** | **DOES NOT APPLY** |
| **Fresh L4** | **REQUIRED** (fail closed) |

### Notes

- `zero blast radius = NOT ESTABLISHED` is **not** `zero blast radius = NO`. No
  positive blast radius has been demonstrated. The reuse condition fails for
  *absence of the required reuse evidence*, not for demonstrated impact.
- Do not describe the covered dunning behavior as "affected by", "impacted by",
  or "touched by" the configuration change. Neutral wording: "the provider-owned
  behavior associated with the changed retry configuration".
- Structural relevance ("the configuration governs the retry schedule") does not
  convert into demonstrated delta impact.

---

## Case 2 — zero blast radius YES, all prerequisites hold → substitution APPLIES

### Situation

- A payment-capable, production-intended Release Candidate is being promoted.
- Step B = YES for the same provider-owned dunning behavior.
- A previous valid `REAL SANDBOX PASS` covering that behavior exists.
- Current L2 evidence covers all application-owned timing changed since that run.
- Fresh L3 provider-boundary evidence for the RC revision is valid.
- An authoritative analysis demonstrably establishes that the difference between
  the previously validated revision and this RC has **zero blast radius** into
  the provider-owned dunning behavior the prior L4 evidence covered — the RC's
  changes are confined to an unrelated reporting surface.

### Classification

| Field | Value |
|---|---|
| RC promotion status | YES |
| Step A | RC gate met → continue |
| Step B | Real-time-only = YES → continue to Step C |
| Condition 1 (L2 covers changed app-owned timing) | Satisfied |
| Condition 2 (provider-boundary evidence valid, fresh L3) | Satisfied |
| Condition 3 (prior `REAL SANDBOX PASS`) | Satisfied |
| Condition 4 (zero blast radius) | **YES** — authoritatively demonstrated |
| **Evidence substitution** | **APPLIES** |
| **Fresh L4** | **NOT REQUIRED FOR THIS DECISION** |

### Notes

- This result is **decision / revision / state scoped**. It resolves *this*
  decision for *this* RC revision only. It is not a standing Level 4 exemption.
- Any future payment-capable, production-intended Release Candidate, or any future
  applicable hard-exception decision, re-enters the classification sequence at
  Step A. Today's zero-blast-radius demonstration may be cited as historical
  evidence, but reuse eligibility must be re-established each time.

---

## Both cases — no release authorization

Neither case authorizes a release. The correct closing form for Case 2 is:

> For the payment-validation question presented, the Level 4 requirement is
> satisfied through valid evidence substitution, so a fresh L4 run is not
> required for this decision.

`Evidence substitution = APPLIES`, `fresh L4 = NOT REQUIRED`, and
`L1 / L2 / L3 PASS` do not, alone or together, establish "promote the RC",
"release approved", or "deploy". A release disposition requires its own scope and
every authoritative release gate. **Validation verdict ≠ release authorization.**
