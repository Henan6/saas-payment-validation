# Contributing

Thanks for your interest. This project is a validation framework and an AI-agent
skill, so the bar for changing **behavioral rules** is deliberately higher than
for documentation.

## What is welcome

- Documentation-only improvements: clarity, examples, typos, broken links,
  better explanations of an existing rule.
- New worked examples under `examples/` that demonstrate an existing rule.
- Questions and discussion — open an Issue or a Discussion.

## Changing behavioral rules

The behavioral rules live in `skill/SKILL.md` and `skill/references/`. If you
propose a change to any of them:

- **Explain the rationale.** Describe the concrete situation the current rule
  handles wrong, or the gap it leaves.
- **Provide evidence.** A failing decision, a real provider behavior, an
  architecture pattern the current rules cannot express.
- **Do not weaken anti-inference behavior.** Rules that force `UNKNOWN / NOT
  ESTABLISHED` when a fact is not established, and that forbid filling a gap by
  implication from a compatible state, are core to the framework. Changes that
  make the skill guess more will be declined.
- **Do not silently convert `UNKNOWN` into an assumption.** If a change makes an
  unestablished fact resolve to a default, call that out explicitly and justify
  it.
- **Keep fail-closed behavior.** Uncertainty that affects Level 4 reuse must
  still force a fresh Level 4 run.
- **Include targeted tests.** A behavioral change should come with the decision
  scenarios that show the new behavior is correct and that existing behavior did
  not regress.

## Regression review

The current skill revision is accepted as `CLOSED / PASS` for that revision only
(see [`docs/DEVELOPMENT_RECORD.md`](docs/DEVELOPMENT_RECORD.md)). Any behavioral change requires a
targeted regression review of the affected rules and their interactions across
the four skill files before it can be accepted.

## Keeping the repository public-safe

This is a public repository. Do not add secrets, credentials, private
configuration, real customer data, private identifiers, or references to private
or commercial projects. See [`SECURITY.md`](SECURITY.md).

## Style

- Markdown, wrapped at a reasonable width, consistent with the existing files.
- Prefer compact tables and clear headings over long prose.
- Keep the core engineering idea easy to find.
