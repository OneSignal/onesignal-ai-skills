---
name: onesignal-[domain]
description: Use this skill whenever a user wants to [primary workflow] in [OneSignal domain]. Include the real words users use, including adjacent terms they may say when they do not know the OneSignal product name.
---

# OneSignal [Domain] Skill

## Purpose

Help OneSignal AI guide a user from a broad request to a valid [domain outcome].

This skill focuses on **[specific scope]**.

Examples:

- Creating [object]
- Optimizing [object]
- Analyzing [object]
- Explaining [object]

Keep the skill narrow. If a domain has multiple large workflows, create separate skills or explicitly scope this one.

## When to Use

Use this skill when the user asks for:

- "[Example trigger phrase]"
- "[Example trigger phrase]"
- "[Example trigger phrase]"

Also use this skill when the user uses adjacent language that maps to this OneSignal domain.

Example:

```text
If the user says "[adjacent term]", treat it as likely [OneSignal product term] unless they clearly mean something else.
```

## Core Behavior

When this skill is active:

1. Determine what the user already provided.
2. Identify missing decisions that materially affect the outcome.
3. Ask only for missing information.
4. Prefer a shared OneSignal AI artifact over freeform text when the user should choose from known options.
5. Summarize the proposed output before creating or changing anything.
6. Require explicit user approval before any write action.
7. Do not invent unsupported product behavior.

## Required Inputs

A useful [domain outcome] usually needs:

- [Required input 1]
- [Required input 2]
- [Required input 3]
- [Optional input 1]
- [Optional input 2]

Do not ask for all of these at once if the user's prompt already provides some of them. Start with the most important missing decision.

## Guided Artifact Behavior

The agent should request shared OneSignal AI artifact primitives. This skill does not define the UI schema or renderer.

### Primary Intake

If the user asks to [primary workflow] without specifying [most important missing decision], request a `multi_select` artifact.

The artifact should present these options:

- [Option 1]
- [Option 2]
- [Option 3]
- [Option 4]
- Something else

Example artifact intent:

```json
{
  "artifact": "multi_select",
  "title": "[Question to ask the user]",
  "options": [
    {
      "id": "option_1",
      "label": "[Option 1]",
      "description": "[Short user-facing explanation]"
    },
    {
      "id": "option_2",
      "label": "[Option 2]",
      "description": "[Short user-facing explanation]"
    },
    {
      "id": "something_else",
      "label": "Something else"
    }
  ]
}
```

### Multi-Select Intake

If the user should choose more than one option from a known set, request a `multi_select` artifact rather than asking many chat questions.

Useful option groups:

- [Option 1]
- [Option 2]
- [Option 3]
- [Option 4]

### Summary Before Action

When enough information exists to draft the output, summarize it in plain text.

The summary should include:

- [Summary item 1]
- [Summary item 2]
- [Summary item 3]
- Assumptions
- Warnings or limitations

### Approval

Before taking a write action, ask the user to approve the summarized output in plain text.

Do not execute the write action until the user confirms.

## Product Rules

Add only the rules the AI needs to avoid bad or unsupported output.

- [Rule or constraint 1]
- [Rule or constraint 2]
- [Rule or constraint 3]

Examples:

- "[Use recommended default] when [condition]."
- "Do not [unsupported behavior]."
- "When [ambiguous term] appears, clarify whether the user means [option A] or [option B]."

## Common Defaults and Best Practices

Add product-owned defaults that are safe and useful.

- [Default or best practice 1]
- [Default or best practice 2]
- [Default or best practice 3]

Only use defaults when they are safe and visible to the user. List assumptions in the summary.

## Tool Guidance

Use available tools to:

- Inspect existing objects when needed.
- List related objects when the user references something by name.
- Validate before creating/updating.
- Execute only after user approval when the action writes data.

Do not expose internal tool mechanics to the user.

## Examples

### Generic Request

User:

> [Generic request]

Behavior:

- Request a `multi_select` artifact asking for [most important missing decision].
- Present product-owned options.
- Continue collecting missing inputs after the user answers.

### Specific Request

User:

> [Specific request with some inputs already provided]

Behavior:

- Treat [known input] as supplied.
- Ask only for missing details such as [missing detail].
- Summarize before asking for approval.

### Text Fallback

If rich artifacts cannot render, ask the same structured question in text:

```text
[Question]
1. [Option 1]
2. [Option 2]
3. [Option 3]
4. Something else
```

## Anti-Patterns

Avoid:

- Asking the user for raw API fields when product-friendly choices are available.
- Asking every possible question up front.
- Guessing decisions that materially change the outcome.
- Taking write actions without summary and explicit approval.
- Inventing unsupported product behavior.
