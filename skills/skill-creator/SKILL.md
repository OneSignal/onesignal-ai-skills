---
name: onesignal-skill-creator
description: Use this skill when a OneSignal PM, designer, engineer, or domain owner wants to create, revise, or evaluate a OneSignal AI domain skill. This includes requests like "create a skill", "make a skill for Journeys", "turn this workflow into a skill", "improve this skill", or "help me define the prompts/options for a product domain skill".
---

# OneSignal Skill Creator

## Purpose

Help OneSignal teams create focused, product-owned `SKILL.md` files for OneSignal AI.

This skill should guide a domain owner from a rough product workflow idea to a concise Claude-style skill file that teaches OneSignal AI:

- when the domain skill should trigger,
- what the skill should help users accomplish,
- what information is required,
- when to ask clarifying questions,
- which product-owned options to offer through AskQuestion,
- which tools/actions are relevant,
- what product rules and guardrails apply,
- when to summarize and require user approval.

## When to Use

Use this skill when the user asks to:

- Create a new OneSignal AI skill.
- Turn a domain workflow into a skill.
- Improve or simplify an existing skill.
- Define AskQuestion options for a domain workflow.
- Convert product rules or best practices into AI behavior.
- Create a skill for domains like Journeys, Segments, Templates, Messaging, Analytics, Onboarding, Setup, or Migration.

## Core Behavior

When helping create a skill:

1. Identify the domain and intended workflow.
2. Keep the scope narrow; avoid packing too many unrelated workflows into one skill.
3. Ask clarifying questions only when needed.
4. Prefer AskQuestion when the domain owner needs to choose from known setup options.
5. Draft a single `SKILL.md` with YAML frontmatter and markdown instructions.
6. Keep the skill concise and operational.
7. Avoid adding artifact schemas, manifests, implementation plans, or PRD content into the skill file.

## Interview Checklist

Use these questions to gather enough context before drafting:

1. What OneSignal domain is this skill for?
2. What should the skill help the AI do?
3. What user phrases should trigger the skill?
4. What is the primary workflow for the first version?
5. What required inputs does the AI need to complete the workflow?
6. Which inputs are often ambiguous and should be clarified?
7. Which choices should be presented through AskQuestion?
8. What tools/actions does the AI need?
9. What product rules or constraints must the AI follow?
10. What should the AI summarize before taking action?
11. What requires explicit user approval?
12. What should the AI never do?

If the user already provided this information in the conversation, extract it instead of asking again.

## Recommended Skill Structure

Use this structure unless there is a strong reason not to:

```markdown
---
name: onesignal-[domain]
description: Use this skill when...
---

# OneSignal [Domain] Skill

## Purpose

## When to Use

## Core Behavior

## Required Inputs

## AskQuestion Guidance

## Product Rules

## Common Defaults and Best Practices

## Tool Guidance

## Summary and Approval

## Examples

## Anti-Patterns
```

## Section Guidance

### Frontmatter

`name` should be lowercase, hyphenated, and specific.

Good:

- `onesignal-segments`
- `onesignal-journeys`
- `onesignal-email-templates`

`description` is the primary trigger. Include both:

- what the skill does,
- when it should be used.

Include adjacent user language. Users may say "audience" instead of "Segment" or "workflow" instead of "Journey".

### Purpose

State the skill's job in one or two short paragraphs.

Example:

```text
Help OneSignal AI guide users from a broad audience request to a valid OneSignal Segment definition.
```

### When to Use

List user phrases and adjacent terms that should activate the skill.

Include product-specific synonyms.

### Core Behavior

Define the general decision loop.

Recommended pattern:

```text
1. Determine what outcome the user wants.
2. Determine what the user already provided.
3. Ask only for missing information that materially changes the result.
4. Use AskQuestion when the user should choose from product-owned options.
5. Summarize the proposed output before creating/changing it.
6. Require explicit approval before write actions.
7. Do not invent unsupported product behavior.
```

### Required Inputs

List the minimum information needed to complete the workflow.

Keep it product-facing, not API-facing.

### AskQuestion Guidance

This is where domain owners define guided intake behavior.

Use AskQuestion when:

- a required decision is missing,
- the option set is product-owned,
- guessing would materially change the outcome,
- the user should not have to answer from a blank slate.

For each AskQuestion, define:

- the condition,
- the question,
- the options,
- whether one option should be `Type your answer`.

Example:

```markdown
If the user asks to create a Journey without specifying the type, use AskQuestion.

Question:
"What type of Journey would you like to create?"

Options:
- Re-engagement
- Winback
- Promotional campaign
- Type your answer
```

### Product Rules

Include only rules the AI needs to avoid bad or unsupported output.

Do not paste entire product docs.

### Common Defaults and Best Practices

Include defaults that are safe, useful, and should be visible to users.

The AI should list assumptions in the summary before approval.

### Tool Guidance

Name the kinds of tools/actions the AI should use.

Examples:

- list existing objects,
- inspect existing state,
- validate before creation,
- create/update only after approval.

Do not expose internal tool mechanics to the user.

### Summary and Approval

Define what the AI should summarize before write actions.

Always require explicit approval before creating, updating, deleting, launching, pausing, or applying generated output.

### Examples

Add 3-5 examples that teach the intended behavior.

Each example should include:

- user prompt,
- what is already known,
- what is missing,
- what the AI should do.

### Anti-Patterns

List behavior the AI should avoid.

Common anti-patterns:

- asking for raw API fields,
- asking every possible question upfront,
- guessing material decisions,
- taking write actions without approval,
- inventing unsupported product behavior.

## Output Expectations

When asked to create a skill:

1. Interview the user if required context is missing.
2. Draft the full `SKILL.md`.
3. Keep the skill focused on one domain/workflow.
4. Avoid creating separate manifests or artifact schemas.
5. Suggest 2-3 test prompts for the domain owner to try.

## Example Test Prompts

For a newly created skill, propose realistic prompts like:

- "Create a [domain object]."
- "Create a [specific domain object] with [some details]."
- "Explain or improve this [domain object]."

The goal is to verify:

- the skill triggers correctly,
- the AI asks the right clarifying questions,
- AskQuestion options are product-owned,
- the AI summarizes before write actions,
- the AI does not invent unsupported behavior.

## Anti-Patterns

Avoid:

- creating a huge PRD-like skill,
- including artifact schemas,
- adding `manifest.json`,
- writing implementation tickets inside the skill,
- overloading one skill with too many unrelated workflows,
- copying large docs verbatim,
- making domain teams define UI rendering details.
