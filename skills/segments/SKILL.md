---
name: onesignal-segments
description: Use this skill when a user wants to create or reason about OneSignal Segments, audiences, cohorts, targeting groups, suppression groups, engaged users, inactive users, VIP users, purchasers, message clickers, custom-event audiences, or users matching behavior, attributes, tags, or subscription properties. Prefer this skill when the user describes building an audience for a campaign, Journey, message, analysis, or exclusion list, even if they do not explicitly say "Segment".
---

# OneSignal Segments Skill

## Purpose

Help OneSignal AI guide users from a broad audience request to a valid OneSignal Segment definition.

This skill currently focuses on **creating Segments**. It also includes lightweight guidance for explaining existing Segments, but deeper Segment analysis/improvement should be expanded later.

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
- "Target users who purchased"
- "Build an audience for this campaign"
- "Build an audience for this Journey"
- "Exclude users from this message"

If the user says "audience", "cohort", "targeting group", or "suppression group", treat it as a likely OneSignal Segment unless they clearly mean a static CSV import, a one-time recipient list, or a dynamic API filter on a single message.

## Core Behavior

When this skill is active:

1. Determine what audience outcome the user wants.
2. Determine what the user already provided: purpose, channel, timeframe, behavior, attributes, tags, match logic, and Segment name.
3. Ask only for missing information that materially changes the Segment.
4. Use AskQuestion when the user should choose from product-owned options.
5. Summarize the proposed Segment before creating it.
6. Require explicit user approval before creating or modifying the Segment.
7. Do not invent unsupported filters or unsupported API behavior.

## Required Inputs for Segment Creation

A useful Segment definition usually needs:

- Segment purpose.
- Segment type or filter family.
- Audience criteria.
- Time window, if behavior/event-based.
- Match logic, if multiple criteria exist.
- Segment name.

Do not ask for all of these at once if the user's prompt already provides some of them. Start with the most important missing decision.

## AskQuestion Guidance

Use AskQuestion instead of freeform text when the user needs to choose from known product options.

### Segment Purpose

If the user asks to "create a Segment" or "build an audience" without specifying the purpose, use AskQuestion.

Question:

```text
What's the goal of this Segment?
```

Options:

- Send a campaign or newsletter
- Suppress or exclude people
- Track or analyze an audience
- Re-engage inactive users
- Type your answer

### Segment Type

If the user describes a broad audience but not the criteria type, use AskQuestion.

Question:

```text
What kind of criteria should define this Segment?
```

Options:

- User behavior or activity
- Message engagement
- User tags or attributes
- Location, language, app version, or device properties
- Type your answer

### Engagement Definition

If the user says "engaged", "most engaged", "high intent", or similar without defining the signal, use AskQuestion.

Question:

```text
What should count as engaged?
```

Options:

- Opened a message
- Clicked a message
- Opened and clicked
- Performed a custom event
- Type your answer

### Inactivity Definition

If the user says "inactive", "lapsed", "dormant", or "churn risk" without defining inactivity, use AskQuestion.

Question:

```text
What should count as inactive?
```

Options:

- No recent app or website session
- No recent message opens or clicks
- No recent purchase or conversion event
- No recent custom event
- Type your answer

### Channel Conflict

If the user requests a Segment based on a channel that is unavailable, not configured, or unlikely to have native engagement data, explain the issue and use AskQuestion.

Question:

```text
How would you like to proceed?
```

Options:

- Use an enabled channel instead
- Use custom events or tags
- Continue anyway
- Type your answer

## Segment Rules

- Segments update automatically as users interact with the app or site.
- Segments can be created in the dashboard, via the Create Segment API, or by CSV import.
- If no filters are selected, a Segment can default to every user of the app. Avoid this unless the user explicitly wants all users.
- Subscription-based Segments use filters on subscription attributes such as device type, language, app version, tags, country, location, and session activity.
- User-based Segments use user-level filters such as message events and custom events.
- Message event filters can target interactions by channel, such as sent, delivered, opened, clicked, bounced, failed, suppressed, reported spam, or received depending on channel.
- Custom event filters target meaningful actions tracked in the app, website, integrations, or API.
- Public Create Segment API filters include tag, last session, first session, session count, session time, language, app version, location, and country.
- Dashboard-created message-event or custom-event Segments may not be fully supported by public Create/Update Segment APIs.
- Filters can be combined with AND/OR logic. AND is the default; OR must be explicit.
- Push, email, and SMS messages are only sent to opted-in Subscriptions when targeting a Segment.
- In-app messages can display to mobile Subscriptions regardless of status.
- When used in Journeys, Segments evaluate Subscriptions to Users, and matching users enter the Journey.
- Audience counts include subscribed and unsubscribed Subscriptions for transparency.
- Audience counts may be exact or estimated. Estimates should be clearly labeled.
- User-based Segment counts may not be available in all contexts.

## Common Defaults and Best Practices

- For campaign targeting, clarify whether the Segment is meant to include or exclude users.
- For behavioral Segments, ask for a timeframe if one is missing.
- For "engaged" audiences, ask which engagement signals count.
- For "inactive" audiences, ask what inactivity means and what timeframe to use.
- For "VIP" audiences, clarify whether VIP means value, tag/tier, purchase behavior, or engagement.
- For multi-condition Segments, clarify whether the user expects AND or OR logic.
- Suggest a clear Segment name if the user does not provide one.
- Summarize human-readable criteria before showing or creating raw filters.
- Avoid PII in tags or tag-derived criteria.

## Tool Guidance

Use available tools to:

- List existing Segments when the user references one by name.
- Inspect an existing Segment before explaining or modifying it.
- Preview or validate audience counts before creation, if supported.
- Inspect channel/platform setup when the Segment depends on channel engagement.
- Create the Segment only after summarizing it and receiving explicit user approval.

Do not expose internal tool mechanics to the user.

## Summary and Approval

Before creating a Segment, summarize it in plain text.

The summary should include:

- Segment name or suggested name.
- Segment purpose.
- Segment type.
- Filters / criteria.
- Time window.
- Match logic.
- Assumptions.
- Limitations or warnings.

Ask the user to approve the summarized Segment before creating it.

## Examples

### Generic Segment Request

User:

```text
Create a Segment.
```

Behavior:

- Use AskQuestion to ask the Segment purpose.
- Present purpose options.
- Continue collecting only missing inputs after the user answers.

### Engaged Email Users

User:

```text
Create a most engaged Segment of my users over the email channel over the last 7 days.
```

Behavior:

- Treat channel and timeframe as known.
- Use AskQuestion to clarify what "most engaged" means.
- Useful options: opened a message, clicked a message, opened and clicked, performed a custom event, type your answer.
- Summarize the Segment before asking for approval.

### Inactive Users

User:

```text
Create a Segment for users who have not opened the app in 30 days.
```

Behavior:

- Treat inactivity type and timeframe as known.
- Suggest a clear Segment name.
- Summarize criteria before asking for approval.

### Channel Conflict

User:

```text
Create a most engaged Segment over SMS in the last 7 days.
```

Behavior:

- If SMS is unavailable or not configured, explain that native SMS engagement may not be available.
- Use AskQuestion to ask how to proceed.
- Useful options: use an enabled channel instead, use custom events or tags, continue anyway, type your answer.

### Text Fallback

If structured question UI cannot render, ask the same question in text:

```text
What's the goal of this Segment?
1. Send a campaign or newsletter
2. Suppress or exclude people
3. Track or analyze an audience
4. Re-engage inactive users
5. Type your answer
```

## Anti-Patterns

Avoid:

- Asking the user for raw Segment API fields.
- Asking every possible Segment question up front.
- Guessing ambiguous terms like "engaged", "inactive", or "VIP" when they materially change the Segment.
- Creating a Segment with no filters unless the user explicitly wants all users.
- Creating a Segment without summarizing it and receiving explicit approval.
- Inventing unsupported Segment filters.
