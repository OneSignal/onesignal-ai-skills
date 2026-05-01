---
name: onesignal-journeys
description: Use this skill when a user wants to create or analyze a OneSignal Journey, or describes creating a workflow, automation, lifecycle flow, re-engagement flow, winback flow, or promotional campaign workflow. Prefer this skill when the user says "workflow" in a lifecycle messaging context, because workflows in Customer.io-style language usually map to Journeys in OneSignal.
---

# OneSignal Journeys Skill

## Purpose

Help OneSignal AI guide users through Journey workflows.

This skill currently has detailed guidance for **creating Journeys**. It also reserves a placeholder for **analyzing Journeys**, which should be expanded later.

## When to Use

Use this skill when the user asks for:

- "Create a Journey"
- "Create a workflow"
- "Build an automation"
- "Create a re-engagement workflow"
- "Create a winback flow"
- "Create a lifecycle campaign"
- "Analyze this Journey"
- "Why is this Journey underperforming?"
- "How is this Journey doing?"

If the user says "workflow" or "automation", treat it as a likely OneSignal Journey unless the user clearly means a one-time campaign, newsletter, or transactional message.

## Supported Capabilities

- `Create Journey`: turn lifecycle intent into a valid Journey draft.
- `Analyze Journey`: placeholder for future Journey performance analysis guidance.

## Core Behavior

When this skill is active:

1. Determine whether the user wants to create a Journey or analyze an existing Journey.
2. Determine what the user already provided.
3. Ask only for missing information.
4. Prefer a shared OneSignal AI artifact over freeform text when the user should choose from known options.
5. For creation, summarize the proposed Journey before creating it.
6. For creation, require explicit user approval before creating the Journey.
7. Do not invent unsupported Journey behavior.

## Create Journey

### Required Inputs

A useful Journey draft usually needs:

- Journey type or goal.
- Audience / entry rule.
- Channels.
- Message sequence.
- Wait timing.
- Exit behavior.
- Re-entry behavior.

Do not ask for all of these at once if the user's prompt already provides some of them. Start with the most important missing decision.

### Guided Artifact Behavior

The agent should request shared OneSignal AI artifact primitives.

#### Journey Type Intake

If the user asks to "create a Journey" or "create a workflow" without specifying the type, request a `multi_select` artifact.

The artifact should present these options:

- Re-engagement
- Winback
- Promotional campaign
- Something else

Example artifact intent:

```json
{
  "artifact": "multi_select",
  "title": "What type of Journey would you like to create?",
  "options": [
    {
      "id": "re_engagement",
      "label": "Re-engagement",
      "description": "Bring inactive users back to your app or site."
    },
    {
      "id": "winback",
      "label": "Winback",
      "description": "Recover lapsed users with a stronger offer or message."
    },
    {
      "id": "promotional",
      "label": "Promotional campaign",
      "description": "Promote a sale, launch, announcement, or limited-time offer."
    },
    {
      "id": "something_else",
      "label": "Something else"
    }
  ]
}
```

#### Multi-Select Intake

If the user should choose more than one option from a known set, request a `multi_select` artifact rather than asking many chat questions.

Useful Journey multi-selects include:

- Channels to use: Push, Email, SMS, In-app.
- Journey goals: activate, re-engage, recover purchase, promote, educate.
- Exit behaviors: user becomes active, completes purchase/event, enters segment, Journey completes.

#### Summary Before Creation

When enough information exists to draft a Journey, summarize it in plain text.

The summary should include:

- Journey name or suggested name.
- Journey type / goal.
- Entry rule and audience.
- Channels.
- Message steps.
- Wait timing.
- Exit behavior.
- Re-entry behavior.
- Assumptions.

#### Approval

Before creating a Journey, ask the user to approve the summarized draft in plain text.

Do not create the Journey until the user confirms.

## Analyze Journey

Placeholder for future work.

For now:

- If the user asks to analyze a Journey, explain that Journey analysis guidance is not fully defined in this skill yet.
- Use available Journey details and analytics tools if they exist.
- Prefer a concise summary with obvious findings over inventing unsupported diagnostics.
- Do not make changes to the Journey as part of analysis.

Future guidance should define:

- Required context for analysis.
- Which Journey-level, step-level, and message-level metrics to inspect.
- Which chart or insight artifacts to request.
- How to prioritize findings.
- How to recommend next actions.

## Product Rules

- Journeys are automated multichannel messaging flows across email, push, SMS, in-app messages, and web push.
- Entry rules can be Segment-based or Custom Event-based, but not both in the same Journey.
- Message steps send when the user reaches them; use Wait steps for delays.
- In-app messages require a new app session to display.
- Re-entry rules apply to Segment-based Journeys; Custom Event-based Journeys can re-enter whenever the event occurs.
- Use exit rules to stop messaging when the user converts, becomes active, enters a segment, or no longer qualifies.
- External IDs are recommended for multichannel Journeys so users are unified across subscriptions.

## Common Defaults and Best Practices

- For re-engagement, entry is usually inactivity-based and exit is usually "user becomes active."
- Avoid over-messaging users who do not respond after several touches.
- Use multiple channels thoughtfully: email for richer context, push for nudges, SMS for urgent reminders, and in-app for contextual moments.

## Tool Guidance

Use available tools to:

- List or inspect Segments when choosing audience criteria.
- List Templates when message steps need existing templates.
- Inspect platform/channel setup before recommending channels.
- Validate the Journey draft before creation, if supported.
- Create the Journey only after summarizing it and receiving user approval.

Do not expose internal tool mechanics to the user.

## Examples

### Generic Request

User:

> Create a Journey.

Behavior:

- Request a `multi_select` artifact asking for Journey type.
- Present the common Journey type options.
- Continue collecting missing inputs after the user answers.

### Re-Engagement

User:

> Create a re-engagement workflow for users who have not opened the app in 14 days.

Behavior:

- Treat Journey type and inactivity window as known.
- Ask only for missing details like channels or message strategy.
- Summarize a Journey with inactive-user entry, exit on app activity, wait steps, and re-entry behavior before asking for approval.

### Analyze Journey Placeholder

User:

> Why is this Journey underperforming?

Behavior:

- State that Journey analysis guidance is not fully defined yet.
- Inspect available Journey details and analytics if tools/context are available.
- Provide a concise summary of obvious findings.
- Do not update the Journey.

### Text Fallback

If rich artifacts cannot render, ask the same structured question in text:

```text
What type of Journey would you like to create?
1. Re-engagement
2. Winback
3. Promotional campaign
4. Something else
```

## Anti-Patterns

Avoid:

- Asking the user for raw Journey API fields.
- Asking every possible Journey question up front.
- Guessing Journey type, entry rule, channel, or exit behavior when it materially changes the flow.
- Creating a Journey without summarizing it and receiving explicit approval.
- Inventing unsupported Journey behavior.
