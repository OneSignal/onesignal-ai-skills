# OneSignal AI Skills

This repository contains example domain skills for OneSignal AI.

Skills are product-owned packages that teach OneSignal AI how to reason about and complete workflows in a specific area of the OneSignal platform. They are intended to be reusable across:

- OneSignal AI chat.
- OneSignal UI-initiated AI experiences.
- OneSignal MCP server workflows.
- Text fallback experiences for clients that cannot render rich UI.

## What Lives Here

Skills may include:

- Domain instructions and best practices.
- Required and optional user inputs.
- Constraints and restrictions.
- Tool/action mappings.
- Tool sequencing guidance.
- Validation expectations.
- Summary and approval expectations.
- Example interactions.

Shared artifact schemas and renderers do **not** live in this repository. Artifact primitives such as `multi_select`, `progress_status`, and `chart` should live with the OneSignal AI runtime. Skills can request those shared primitives and provide domain-specific content for them.

## Current Examples

- `skills/skill-creator/`: helper skill for creating and refining OneSignal AI domain skills.
- `skills/segments/`: example Segments skill for creating OneSignal audience Segments.
- `skills/mobile-sdk-setup/`: example setup skill for guiding mobile SDK installation and troubleshooting.
- `skills/journeys/`: example Journeys skill for designing OneSignal multichannel automation flows.

## Suggested Shape

```text
onesignal-ai-skills/
  README.md
  skills/
    skill-creator/
      SKILL.md
    segments/
      SKILL.md
    mobile-sdk-setup/
      SKILL.md
    journeys/
      SKILL.md
    templates/
      SKILL.md
    messaging/
      SKILL.md
```

Each domain skill should use this shape:

```text
skills/
  domain-name/
    SKILL.md
```

`SKILL.md` teaches the AI how to reason. This intentionally follows the simplest Claude Skills pattern: the only required file is `SKILL.md`. Supporting references can be added later if a skill becomes too large to keep in one file.

To create a new domain skill, copy `example-template/domain-skill/`, rename the folder, and replace placeholders with product-specific guidance.

## Recommended Skill Sections

Use the template's section order for consistency:

1. Purpose
2. When to Use
3. Core Behavior
4. Required Inputs
5. Guided Artifact Behavior
6. Product Rules
7. Common Defaults and Best Practices
8. Tool Guidance
9. Examples
10. Anti-Patterns
