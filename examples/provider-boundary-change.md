# Example — provider boundary change

> Fictional scenario. No real company, provider, or system is described.

## Situation

A SaaS product consumes subscription lifecycle webhooks from a payment provider
(referred to here as *the provider*). An engineer changes how one provider event
is interpreted:

- **Before:** a `subscription.past_due` provider event maps to the internal
  state `GRACE`.
- **After:** the same provider event, when its payload carries
  `collection_paused = true`, maps to a new internal state `SUSPENDED_HOLD`
  instead.

This is a change to **provider event payload semantics → internal lifecycle-state
mapping**. The downstream entitlement rules were not touched; only the
interpretation of the provider payload changed.

No Release Candidate is being promoted. This is ordinary development.

## Classification

| Field | Value |
|---|---|
| Task mode | Payment change review (normal development) |
| Payment-surface classification | Webhook / provider event → internal lifecycle-state interpretation |
| L1 | REQUIRED — deterministic mapping and the new state's transitions |
| L2 | NOT REQUIRED — no persistence / concurrency / ordering behavior changed |
| **L3** | **REQUIRED** — the semantic interpretation of a real provider payload field changed; this is a provider-wire-boundary change even though no outbound call or request shape changed |
| L4 | NOT REQUIRED — Step A: no payment-capable, production-intended RC is being promoted, and no hard exception applies. Step B (real-time-only classification) = NOT REACHED |
| RC promotion status | NO (explicitly established for this decision) |
| Evidence substitution | NOT RELEVANT (L4 decision stopped at Step A) |
| Real-time-only behavior / L4 scenario scope | NOT REACHED |
| Freeze status | No freeze active |

## Result

**`L3 = REQUIRED`.** This is settled by the L3 explicit-fact rule: once a
provider-boundary change (provider event type / payload field semantics →
internal lifecycle-state interpretation) is explicitly established, there is
nothing left to classify about *whether* L3 is required.

If no fresh provider-sandbox run has been executed for the revision under review:

**`L3 = REQUIRED — NOT RUN / EVIDENCE MISSING`.**

This is **not** `L3 = UNKNOWN`. `UNKNOWN` is only for genuinely unresolved
boundary ownership — where whether the provider boundary changed at all is not
established. Here it *is* established.

## The distinction this shows

> **Classification known ≠ evidence current.**

"We know L3 is required" and "L3 evidence exists for this revision" are two
different facts:

- *Classification state* — `L3 = REQUIRED` — is decided from what changed.
- *Execution state* — `NOT RUN / EVIDENCE MISSING` — is decided from what has
  actually been run against the revision under review.

Missing execution evidence against a `REQUIRED` classification is reported as
`NOT RUN / EVIDENCE MISSING`, never converted back into classification
uncertainty. The only remaining L3 action is to run the provider-sandbox
validation on the current revision and record the evidence. Do not phrase it as
"REQUIRED unless someone argues it is internal" or "resolve REQUIRED vs.
UNKNOWN".

## Next actions

1. Add / update L1 coverage for the new `SUSPENDED_HOLD` state and every valid
   and invalid transition into and out of it.
2. Run a short L3 provider-sandbox validation on the pinned revision under
   review: confirm the real provider `subscription.past_due` payload (including
   the `collection_paused` field) is interpreted into the intended internal
   state.
3. Record the L3 result. Until it is run, the L3 evidence state remains
   `NOT RUN / EVIDENCE MISSING`.
