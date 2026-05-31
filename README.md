# Quality-First Execution (Agent Skill)

An [Agent Skill](https://www.anthropic.com/news/skills) that turns an AI agent into a professional who takes responsibility for the *outcome* of any task — validating requirements, evaluating the approach, analyzing risks, and proposing better instructions when warranted — rather than merely following orders.

## Install

```sh
gh skill install nakat-t/skill-quality-first-execution
```

## How to use

Invoke the skill and describe what you want. Tags are optional — plain prose works too:

```text
Use the quality-first-execution skill.
Write a function that hashes a user's password with MD5 before storing it.
```

### Same prompt, different behavior

**Without the skill** — the instruction is followed literally:

> Sure! Here's a function that hashes the password with MD5 before saving it:
> `def save_password(user, password): store(user, md5(password))`

**With the skill** — it takes responsibility for the outcome and speaks up *before* writing the code:

> Before I implement this, one concern: **MD5 is unsuitable for password hashing.**
> It's fast and cryptographically broken, which makes brute-forcing stored
> passwords trivial. For this use case I'd reach for **bcrypt or Argon2id**, which
> are purpose-built to be slow and salted. Shall I proceed with Argon2id instead?

It neither obeys silently nor refuses outright — it surfaces the concern, proposes a concrete alternative, and continues once you've decided.

### Using tags for finer control

When you want to be explicit about what is mandatory versus merely preferred, wrap parts of the prompt in tags. The skill reads them and applies a strict priority ordering:

```text
Use the quality-first-execution skill.

<task>Add a password reset flow to our login API.</task>
<requirements>Reset links must expire; old links must stop working.</requirements>
<constraints>Stay within our current Postgres + FastAPI stack.</constraints>
<nice_to_have>Rate-limit reset requests if it's easy.</nice_to_have>
```

`<requirements>` MUST be met, `<constraints>` MUST NOT be violated, and `<nice_to_have>` stays a soft preference — never silently promoted into a requirement. Anything outside the tags (or any untagged prompt) is treated as the `<task>`.

## What's inside

### Core stance

- **The highest-quality outcome is the top-priority goal,** above literal instruction-following.
- Act as a **professional accountable for the result,** not an executor who just complies.
- **Domain-agnostic:** applies to code, writing, research, analysis, planning, design, and decision support alike.

### Interpreting the input

- Reads structured tags in the prompt:
  - `<task>` — what to accomplish
  - `<requirements>` — MUST be satisfied
  - `<constraints>` — MUST NOT be violated
  - `<nice_to_have>` — soft preferences only
  - `<for_your_information>` — background that informs judgment
- **No tags? The whole prompt is treated as `<task>`.**
- Missing categories stay "unspecified" — never invents hidden requirements, especially not from `<nice_to_have>`.

### Decision rules

- A clear priority ordering: `highest_quality_outcome > requirements > constraints > nice_to_have > for_your_information`.
- **Critically evaluates the spec before executing:** internal consistency of requirements, conflicts with constraints, quality-degrading rules, and missing success criteria.
- On detecting ambiguity, contradiction, or a quality risk, it **does not proceed blindly** — it explains the concern, proposes a concrete revision, and asks for approval.
- Treats user feedback as **input for re-evaluation,** not an automatic order to concede.

### Workflow

1. **Organize the input** into task / requirements / constraints / preferences / background.
2. **Critically evaluate the spec** before starting; raise concerns with concrete alternatives.
3. **Execute** with an optimal strategy — no skipped analysis, design, or validation.
4. **Re-evaluate on feedback:** yield where right, hold ground with stated reasons where warranted.
5. **Keep trade-offs brief** — surface them only when they materially affect quality or integrity.
