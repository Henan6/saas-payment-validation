# SaaS Payment Validation

A reusable validation framework and AI-agent skill for testing payment-capable
SaaS systems across deterministic tests, real integration environments, provider
sandboxes, and real-time release validation.

It gives engineers and coding agents a shared, explicit vocabulary for one hard
question: **what is the minimum sufficient evidence that a payment change is
correct — and when is a real-time release run actually required?**

---

## Why it exists

Payment systems routinely pass ordinary tests while still failing in production
around the parts that ordinary tests cannot reach:

- provider event and payload semantics;
- asynchronous lifecycle transitions;
- provider-owned timing (dunning, retries, renewal and cancellation boundaries);
- stale validation evidence that no longer describes the current revision;
- deployment and configuration drift;
- entitlement state transitions;
- real-time validation integrity — evidence quietly changing underneath a
  long-running observation.

Most of these are not caught by "add more tests." They are caught by being
precise about *which kind of evidence* a given change needs, and by refusing to
let a strong-looking test stand in for evidence it does not actually provide.

This repository packages that discipline as:

1. a **validation ladder** (L1–L4) with explicit routing rules;
2. an **evidence model** that separates *required* evidence from *observed*
   evidence and forbids inference from a compatible-looking state;
3. a **freeze protocol** that protects real-time release evidence from
   contamination;
4. an **AI-agent skill** (`skill/`) that applies all of the above during code
   review, planning, or release classification.

---

## The validation ladder

| Level | Name | Proves | Needs |
|---|---|---|---|
| **L1** | Deterministic / local implementation validation | State machines, lifecycle mapping, entitlement decisions, parsing, duplicate-event handling, timestamp-boundary logic under controlled time | No DB, no provider, no network, no real time |
| **L2** | Real integration validation | Persistence, transactions, concurrency, ordering, conflict resolution, reconciliation races | Real isolated test DB / integration environment; provider boundary still fake |
| **L3** | Real provider sandbox / wire-boundary validation | Provider SDK/API calls, request shape, webhook authenticity, payload schema interpretation, provider-event → internal-lifecycle mapping | A real provider sandbox; a pinned revision for the run |
| **L4** | Frozen real-time validation | Behavior that only real elapsed time against a real provider can demonstrate — provider-owned schedules and lifecycle boundaries | A pinned revision, a deployed frozen environment, disposable test entities, and an intact freeze |

Higher levels are **not** automatically better. L1–L3 are the normal engineering
loop. L4 is a release-candidate mechanism, evaluated only when a payment-capable,
production-intended Release Candidate is actually being promoted — or when a
defined hard exception applies (first paid launch, a provider migration that
changes real lifecycle/timing behavior, or a required restart of a contaminated
L4 run).

---

## Distinctions the framework holds

Most payment-validation mistakes are one of these two things being collapsed into
one. The framework keeps them apart:

| This | is not | that |
|---|---|---|
| Historical evidence | ≠ | current evidence for the revision under review |
| Classification state (`L3 = REQUIRED`) | ≠ | execution state (`NOT RUN / EVIDENCE MISSING`) |
| A scenario `FAIL` under intact integrity | ≠ | freeze contamination |
| `BLOCKED` (cannot continue) | ≠ | `FAIL` (wrong observed result) |
| Deployment *verification* required | ≠ | *redeployment* required |
| A restored test entity | ≠ | restored evidence for a contaminated run |
| An old `PASS` exists | ≠ | known historical coverage of the needed behavior |
| Subset / zero-blast-radius on part of a change | ≠ | zero blast radius for the whole decision |
| A validation verdict | ≠ | release authorization |
| `zero blast radius = NOT ESTABLISHED` | ≠ | `zero blast radius = NO` (demonstrated impact) |
| An active L4 freeze | ≠ | proof that `RC = YES` or that a particular hard exception applies |

When a fact is not established by the prompt, the repository, authoritative
evidence, or explicit task context, the answer is `UNKNOWN / NOT ESTABLISHED` —
never a value filled in by implication.

---

## Why this is useful

**For teams integrating payment providers:**

- a shared checklist for "what does this change actually need before it ships";
- a defensible reason to *not* run an expensive real-time release process when
  the change is application-owned and deterministically testable;
- a defensible reason to *require* one when a provider boundary or a
  provider-owned schedule genuinely changed;
- a protocol that keeps a long-running real-time validation trustworthy instead
  of quietly contaminated.

**For teams using AI coding agents on payment code:**

- the skill in `skill/` gives an agent an explicit, auditable decision procedure
  instead of a vibe;
- it forces the agent to distinguish "a test exists" from "the test ran and
  passed for this revision";
- it makes the agent fail closed — mark things `UNKNOWN`, refuse to invent
  provider behavior or prior results — which is exactly the behavior you want
  around money movement.

**For evaluating agents:** the `docs/` model and the worked `examples/` are a
compact benchmark for whether a tool can hold subtle evidence distinctions under
pressure.

---

## Validation status

- **42 behavioral stress tests** were run against the skill during development.
- The **final integrated test, Test 42 = PASS**.
- The current skill revision is **`CLOSED / PASS`** — the accepted baseline for
  this revision.
- This is **internal engineering validation**, not an external audit or
  certification.

See [`docs/ACCEPTANCE.md`](docs/ACCEPTANCE.md) for the full record and
[`docs/TEST_MATRIX.md`](docs/TEST_MATRIX.md) for the behavioral coverage summary.

This project explicitly does **not** claim to be: production certified, security
certified, formally verified, guaranteed safe, or a production-ready application.
It is a validation framework and a decision aid.

---

## Repository structure

```
.
├── README.md                     # this file
├── LICENSE                       # MIT
├── CONTRIBUTING.md               # how to propose changes to behavioral rules
├── SECURITY.md                   # what not to submit; how to report issues
│
├── skill/                        # the AI-agent skill (accepted, CLOSED / PASS)
│   ├── SKILL.md                  # decision procedure and output contract
│   └── references/
│       ├── validation-ladder.md  # L1–L3 routing and evidence-substitution states
│       ├── evidence-model.md     # RC gate, hard exceptions, Step A/B/C, substitution
│       └── freeze-protocol.md    # L4 entry, integrity checks, contamination, restart
│
├── docs/
│   ├── VALIDATION_MODEL.md       # the model as an engineering design document
│   ├── TEST_MATRIX.md            # behavioral coverage summary (42 tests)
│   └── ACCEPTANCE.md             # public acceptance record
│
└── examples/
    ├── provider-boundary-change.md   # provider payload semantics changed → L3 REQUIRED
    ├── evidence-substitution.md       # when prior L4 evidence may / may not be reused
    ├── contamination-restart.md       # provider-side-only contamination → restart mechanics
    └── active-l4-run.md               # mid-run status review of an active freeze
```

---

## Quick start

Nothing here needs a database, a provider account, a build step, or network
access. It is documentation and a skill definition.

**Read it in this order:**

1. [`docs/VALIDATION_MODEL.md`](docs/VALIDATION_MODEL.md) — the concepts and why
   each distinction exists.
2. [`skill/SKILL.md`](skill/SKILL.md) — the decision procedure and the exact
   output fields it produces.
3. [`skill/references/`](skill/references/) — the detailed rules the skill
   applies.
4. [`examples/`](examples/) — four worked decisions.

**Use it with an AI coding agent:** place `skill/SKILL.md` and
`skill/references/` where your agent loads skills (for example, a
`saas-payment-validation` skill directory), then ask the agent to classify a
payment change. The skill tells the agent what to read and in what order; it
does not require any of the private services a real payment system would use.

**Use it as a human checklist:** for a payment change, walk the "Fast routing"
table in `skill/SKILL.md`, then produce the output fields listed at the end of
that file.

---

## Core principles

1. **Minimum sufficient validation.** Choose the least validation that still
   proves correctness. Higher levels are not automatically better.
2. **L4 is a gate, not a default.** Real-time validation is considered only for a
   real payment-capable Release Candidate, or a defined hard exception. "Important"
   or "timing-sensitive" is never by itself enough.
3. **Ownership decides real-time-only.** Application-owned timing with an
   injectable clock stays L1/L2-testable, even at release time. Only a
   provider-owned schedule whose real elapsed-time behavior is the thing under
   test is real-time-only.
4. **Required evidence ≠ observed evidence.** No PASS from static inspection,
   existing tests, checklists, a mentioned command, or simulated time.
5. **Fail closed.** Unestablished facts stay `UNKNOWN`. Uncertainty that affects
   L4 reuse forces a fresh L4 run.
6. **Don't infer from a compatible state.** An active freeze does not prove why
   L4 was entered. An aggregate ("no hard exception applies") does not establish
   each component predicate.
7. **Protect the evidence.** A real-time run is meaningful only if the observed
   system does not silently change underneath it — hence the freeze protocol and
   its four integrity checks.
8. **Validation verdict ≠ release authorization.** A clean validation result
   never, on its own, authorizes a promotion or a deploy.

---

## Example decisions

| Example | Shows |
|---|---|
| [provider-boundary-change.md](examples/provider-boundary-change.md) | Provider event → internal lifecycle-state mapping changed → `L3 = REQUIRED`; unexecuted → `REQUIRED — NOT RUN / EVIDENCE MISSING`. Classification known ≠ evidence current. |
| [evidence-substitution.md](examples/evidence-substitution.md) | A prior `REAL SANDBOX PASS` exists. When zero blast radius is not established, substitution **does not apply** and a fresh L4 is required; a contrasting case where all prerequisites hold and it does apply. |
| [contamination-restart.md](examples/contamination-restart.md) | Provider-side-only contamination: same repository revision stays authoritative, fresh L4 required, deployment verification required, redeployment **not** automatically required. |
| [active-l4-run.md](examples/active-l4-run.md) | An active freeze with two scenarios PASS and one PENDING: aggregate run `INCOMPLETE`, freeze integrity `NOT CONTAMINATED`, `REAL SANDBOX PASS` not yet eligible. |

---

## Collaboration

I'm interested in collaboration around:

- SaaS product engineering
- AI-native development workflows
- payment infrastructure
- agent evaluation
- developer tooling

If any of that overlaps with what you're working on, please open an Issue or a
Discussion on this repository, or reach out through my GitHub profile
([@Henan6](https://github.com/Henan6)).

Contributions to the framework are welcome — see
[`CONTRIBUTING.md`](CONTRIBUTING.md).

## License

[MIT](LICENSE) © 2026 Henan6
