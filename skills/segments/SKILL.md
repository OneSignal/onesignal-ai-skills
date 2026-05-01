---
name: onesignal-segments
description: Use this skill when a user wants to create or reason about a OneSignal Segment, audience, cohort, targeting group, engaged users, inactive users, VIP users, purchasers, message clickers, or users matching behavior or attributes. Prefer this skill when the user describes building an audience for a campaign, Journey, message, or analysis, even if they do not explicitly say "Segment".
---

# OneSignal Segments Skill

## Purpose

Help OneSignal AI guide users from a broad audience request to a valid Segment definition.

This skill currently has detailed guidance for **creating Segments**. It also reserves a placeholder for **analyzing or improving Segments**, which should be expanded later.

## When to Use

Use this skill when the user asks for:

- "Create a Segment"
- "Create an audience"
- "Build a targeting group"
- "Create an engaged users Segment"
- "Create an inactive users Segment"
- "Create a VIP users Segment"
- "Target users who clicked"
- "Target users who opened"
- "Build an audience for this campaign"
- "Build an audience for this Journey"

If the user says "audience", "cohort", or "targeting group", treat it as a likely OneSignal Segment unless they clearly mean a one-time recipient list or CSV import.

## Supported Capabilities

- `Create Segment`: turn audience intent into a valid Segment definition.
- `Analyze Segment`: placeholder for future Segment analysis guidance.
- `Improve Segment`: placeholder for future Segment refinement guidance.

## Core Behavior

When this skill is active:

1. Determine whether the user wants to create, analyze, or improve a Segment.
2. Determine what the user already provided.
3. Ask only for missing information.
4. Prefer a shared OneSignal AI artifact over freeform text when the user should choose from known options.
5. For creation, summarize the proposed Segment before creating it.
6. For creation, require explicit user approval before creating the Segment.
7. Do not invent unsupported Segment filters or unsupported product behavior.

## Create Segment

### Required Inputs

A useful Segment definition usually needs:

- Segment purpose or audience goal.
- Segment type.
- Filters or audience criteria.
- Time window, if behavior/event-based.
- Match logic, if multiple criteria exist.
- Segment name.

Do not ask for all of these at once if the user's prompt already provides some of them. Start with the most important missing decision.

### Guided Artifact Behavior

The agent should request shared OneSignal AI artifact primitives.

#### Segment Purpose Intake

If the user asks to "create a Segment" or "build an audience" without specifying the purpose, request a `multi_select` artifact.

The artifact should present these options:

- Send a campaign or newsletter
- Suppress or exclude people
- Track or analyze an audience
- Re-engage inactive users
- Something else

Example artifact intent:

```json
{
  "artifact": "multi_select",
  "title": "What's the goal of this Segment? What do you want to do with it?",
  "options": [
    {
      "id": "send_campaign",
      "label": "Send a campaign or newsletter",
      "description": "Target a specific audience for a one-time or ongoing message."
    },
    {
      "id": "suppress_or_exclude",
      "label": "Suppress or exclude people",
      "description": "Keep certain people out of a campaign or flow."
    },
    {
      "id": "track_or_analyze",
      "label": "Track or analyze an audience",
      "description": "Monitor a group over time without necessarily messaging them."
    },
    {
      "id": "re_engage_inactive",
      "label": "Re-engage inactive users",
      "description": "Win back people who have gone quiet."
    },
    {
      "id": "something_else",
      "label": "Something else"
    }
  ]
}
```

#### Ambiguous Audience Intake

If the user uses an ambiguous audience term, request a `multi_select` artifact to clarify what it means.

Examples:

- "Most engaged" may mean opened, clicked, received, visited, purchased, or performed a custom event.
- "Inactive" may mean no recent session, no message engagement, no purchase, or no custom event.
- "VIP" may mean high purchase value, high engagement, specific tag/tier, or specific subscription plan.

#### Channel or Setup Conflict

If the user requests a Segment based on a channel that is unavailable or not configured, explain the issue and request a `multi_select` artifact asking how they want to proceed.

Example options:

- Use a different enabled channel.
- Proceed with custom events or tags.
- Something else.

#### Summary Before Creation

When enough information exists to define a Segment, summarize it in plain text.

The summary should include:

- Segment name or suggested name.
- Segment purpose.
- Segment type.
- Filters / criteria.
- Time window.
- Match logic.
- Assumptions.
- Limitations or warnings.

#### Approval

Before creating a Segment, ask the user to approve the summarized definition in plain text.

Do not create the Segment until the user confirms.

## Analyze / Improve Segment

Placeholder for future work.

For now:

- If the user asks to analyze or improve a Segment, explain that detailed analysis/refinement guidance is not fully defined in this skill yet.
- Use available Segment details and audience counts if tools/context are available.
- Prefer a concise summary with obvious findings over inventing unsupported diagnostics.
- Do not modify the Segment as part of analysis.

Future guidance should define:

- Required context for analysis.
- Which audience counts and filter details to inspect.
- How to detect over-broad, too-narrow, stale, or risky Segment definitions.
- Which chart or insight artifacts to request.
- How to recommend next actions.

## Product Rules

- Segments update automatically as users interact with the app or site.
- Segments can be created in the dashboard, via API, or by CSV import.
- If no filters are selected, a Segment can default to every user of the app; avoid this unless the user explicitly wants all users.
- Subscription-based Segments use filters on subscription attributes like device type, language, app version, tags, country, location, and session activity.
- User-based Segments use user-level filters such as message events and custom events.
- Message event filters can target interactions like sent, delivered, opened, clicked, bounced, failed, suppressed, or reported spam depending on channel.
- Custom event filters can target meaningful actions tracked in the app, website, or external systems.
- Message event and custom event Segments may have product/plan/data-retention constraints.
- Push, email, and SMS messages are only sent to subscribed Subscriptions when targeting a Segment. In-app messages can display regardless of subscription status.
- In Journeys, Segments evaluate Subscriptions to Users and all matching Users can enter the Journey.
- Use AND when all filters must match. Use OR when any condition can match.

## Common Defaults and Best Practices

- For "engaged" audiences, clarify which engagement signals count.
- For "inactive" audiences, clarify what inactivity means and what timeframe to use.
- For campaign targeting, clarify whether the Segment should include or exclude users.
- For behavioral Segments, ask for a timeframe if one is not supplied.
- For multi-condition Segments, clarify whether the user expects AND or OR logic.
- Use human-readable criteria before showing raw filters.

## Tool Guidance

Use available tools to:

- List existing Segments when the user references one by name.
- Preview or validate Segment counts before creation, if supported.
- Inspect channel/platform setup if the Segment depends on channel engagement.
- Create the Segment only after summarizing it and receiving user approval.

Do not expose internal tool mechanics to the user.

## Examples

### Generic Request

User:

> Create a Segment.

Behavior:

- Request a `multi_select` artifact asking for Segment purpose.
- Present purpose options.
- Continue collecting missing inputs after the user answers.

### Engaged Email Users

User:

> Create a most engaged Segment of my users over the email channel over the last 7 days.

Behavior:

- Treat channel and timeframe as known.
- Clarify what "most engaged" means.
- Useful options include opened an email, clicked an email, both opened and clicked, and something else.
- Summarize the Segment before asking for approval.

### Disabled Channel Conflict

User:

> Create a most engaged Segment over SMS in the last 7 days.

Behavior:

- If SMS is unavailable or disabled, explain that native SMS engagement may not be available.
- Ask how the user wants to proceed.
- Useful options include use an enabled channel, proceed with custom events/tags, or something else.

### Inactive Users

User:

> Create a Segment for users who have not opened the app in 30 days.

Behavior:

- Treat inactivity type and timeframe as known.
- Suggest a clear Segment name.
- Summarize criteria before asking for approval.

### Analyze / Improve Placeholder

User:

> Is this Segment too broad?

Behavior:

- State that detailed Segment analysis guidance is not fully defined yet.
- Inspect available Segment criteria and counts if tools/context are available.
- Provide a concise summary of obvious findings.
- Do not update the Segment.

### Text Fallback

If rich artifacts cannot render, ask the same structured question in text:

```text
What's the goal of this Segment?
1. Send a campaign or newsletter
2. Suppress or exclude people
3. Track or analyze an audience
4. Re-engage inactive users
5. Something else
```

## Anti-Patterns

Avoid:

- Asking the user for raw Segment API fields.
- Asking every possible Segment question up front.
- Guessing ambiguous terms like "engaged", "inactive", or "VIP" when they materially change the Segment.
- Creating a Segment with no filters unless the user explicitly wants all users.
- Creating a Segment without summarizing it and receiving explicit approval.
- Inventing unsupported Segment filters.
