---
name: onesignal-journeys
description: Use this skill when a user wants to create, design, plan, or reason about a OneSignal Journey, automated messaging flow, lifecycle workflow, drip campaign, onboarding sequence, welcome series, re-engagement campaign, win-back flow, abandoned cart flow, promotional campaign, behavioral followup, multichannel automation, or event-driven messaging across push, email, SMS, in-app, or web push. Prefer this skill when the user describes an automated, multi-step, or branching messaging flow even if they do not explicitly say "Journey". Also use this skill to explain or improve an existing Journey, choose entry/exit/re-entry rules, plan branches, or pick the right action step (wait, wait until, time window, yes/no, split, tag).
---

# OneSignal Journeys Skill

## Purpose

Help OneSignal AI guide users from a broad automation idea to a valid OneSignal Journey design.

This skill currently focuses on **designing and creating Journeys**. It also includes lightweight guidance for explaining and improving existing Journeys. Deeper Journey analytics interpretation and Journey migration workflows can be expanded later.

A Journey is a multi-step automated flow that delivers messages across **push**, **email**, **SMS**, **in-app messages**, and **web push**, based on user behavior, time delays, segment membership, or custom events.

## When to Use

Use this skill when the user asks for:

- "Create a Journey"
- "Build an automation"
- "Set up a welcome series"
- "Build an onboarding flow"
- "Build an onboarding sequence"
- "Create a re-engagement campaign"
- "Create a win-back flow"
- "Build an abandoned cart flow"
- "Send a drip campaign"
- "Send a recurring weekly message"
- "Send a message X days after signup"
- "Branch users based on whether they clicked/opened"
- "A/B test messages over time"
- "Trigger a flow when a custom event fires"
- "Send a follow-up if the user doesn't take action"
- "Explain this Journey"
- "Improve this Journey"
- "Why is my Journey not entering users?"

If the user says "automation", "drip", "lifecycle flow", "workflow", "sequence", or "campaign with multiple messages over time", treat it as a likely Journey unless they clearly mean a single one-time message, a transactional API send, or a non-OneSignal automation tool.

If the user only needs a single message with no branching, waiting, or re-entry, suggest a regular message or template instead and confirm before defaulting to a Journey.

## Core Behavior

When this skill is active:

1. Determine what outcome the user wants (onboarding, re-engagement, abandoned cart, promotional, recurring, event-driven, etc.).
2. Determine what the user already provided: trigger, audience, channels, message timing, branching, exit conditions, and re-entry.
3. Ask only for missing information that materially changes the Journey design.
4. Use AskQuestion when the user should choose from product-owned options.
5. Summarize the proposed Journey before creating or modifying it.
6. Require explicit user approval before creating, launching, pausing, archiving, or deleting a Journey.
7. Do not invent unsupported step types, unsupported channels, or unsupported entry/exit conditions.
8. Prefer simple, working Journey shapes over clever ones. Onboarding, re-engagement, and abandoned cart patterns should reuse the well-known shapes documented in OneSignal Journey examples.

## Required Inputs for Journey Creation

A useful Journey design usually needs:

- Journey purpose (the business goal).
- Entry rule (Segment-based or Custom Event-based). These are mutually exclusive per Journey.
- Audience include/exclude Segments, when entry is Segment-based.
- Custom Event name and any property filters, when entry is Custom Event-based.
- Channels and approximate sequence (push, email, SMS, in-app, webhook).
- Wait timing between steps.
- Branching behavior, if any.
- Exit rules.
- Re-entry rules, when entry is Segment-based.
- Schedule (start now or future, optional end time).
- Journey name (max 300 chars) and optional description (max 255 chars).

Do not ask for all of these at once if the user's prompt already provides some of them. Start with the most important missing decision, usually the Journey purpose and the entry rule.

## AskQuestion Guidance

Use AskQuestion instead of freeform text when the user needs to choose from known product options.

### Journey Purpose

If the user asks to "create a Journey" or "build an automation" without specifying the purpose, use AskQuestion.

Question:

```text
What kind of Journey do you want to build?
```

Options:

- Welcome / onboarding sequence
- Re-engagement / win-back
- Abandoned cart
- Promotional campaign or product launch
- Recurring scheduled message (daily, weekly, etc.)
- Event-driven / behavioral followup
- Type your answer

### Entry Rule Type

If the user does not indicate whether the Journey should be triggered by audience membership or by a behavioral event, use AskQuestion.

Question:

```text
How should users enter this Journey?
```

Options:

- When they match a Segment (audience-based)
- When they trigger a Custom Event (behavior-based)
- I'm not sure — help me choose
- Type your answer

If the user is unsure, recommend Custom Event entry when the trigger is a discrete user action (signup, purchase, milestone, cart update). Recommend Segment entry when the trigger is a state (inactive 7+ days, in trial, on the free plan).

### Channel Mix

If the user has not chosen channels, use AskQuestion.

Question:

```text
Which channels should this Journey use?
```

Options (allow multiple):

- Push notifications
- Email
- SMS
- In-app messages
- Webhooks (send data to external systems)
- Type your answer

Only suggest channels that are reasonable for the Journey purpose. If the user's app does not have a channel set up, mention it as a prerequisite instead of silently proceeding.

### Re-entry Behavior

If entry is Segment-based and the user has not specified re-entry rules, use AskQuestion.

Question:

```text
Should users be able to re-enter this Journey after they exit?
```

Options:

- No, this is a one-time campaign
- Yes, after a fixed amount of time (e.g. cart, weekly recurring)
- Yes, every time they re-qualify
- Type your answer

### Exit Rule

If the user has not specified what should make a user leave the Journey early, use AskQuestion.

Question:

```text
When should users exit early?
```

Options:

- When they take the goal action (segment entry, custom event)
- When they become active in the app or website (re-engagement style)
- When they no longer match the entry audience
- Only when they reach the end of the Journey
- Type your answer

### Recurring Schedule

If the user wants a recurring or day-of-week Journey but has not specified the cadence, use AskQuestion.

Question:

```text
How often should this Journey send?
```

Options:

- Every day at a specific time window
- A specific day of the week (e.g. every Friday)
- A specific date or one-time scheduled launch
- Type your answer

### Channel Conflict

If the user requests a Journey with a channel that is unavailable, not configured, or unlikely to have engagement signals (e.g. SMS on an app without SMS set up), explain the issue and use AskQuestion.

Question:

```text
How would you like to proceed?
```

Options:

- Use an enabled channel instead
- Set up the channel first, then continue
- Skip that channel in this Journey
- Continue anyway
- Type your answer

## Journey Components

OneSignal AI should reason about Journeys using these building blocks. Do not invent step types beyond this list.

### Settings

- Journey **name** (required, max 300 chars).
- Journey **description** (optional, max 255 chars).
- **Entry rules**: either Segment-based (Include/Exclude Segments) **or** Custom Event-based — never both.
- **Future additions only**: when on, only users who join the included or excluded Segment after launch can enter. Existing matching users are permanently excluded, even if they leave and rejoin the Segment.
- **Exit rules**: user becomes active, Custom Event fires, user no longer matches the audience, user enters a Segment. Optional: tag the user when they exit early.
- **Re-entry rules**: apply only to Segment-based entry. Custom Event entry always allows re-entry. The re-entry timer starts at exit, not entry.
- **Schedule**: start now or later; run indefinitely or stop at a future date. Past the end date, the Journey auto-stops and archives.
- **Goals (alpha)**: optional Journey-level and message-level success metrics. Do not assume Goals are enabled unless the user mentions them.

### Action Steps

- **Wait** — fixed delay (minutes, hours, days, weeks). Wait timer changes only affect users who enter the step after the change.
- **Wait Until** — hold until a Segment entry, message event, or Custom Event happens; supports multiple conditions, an expiration branch, and **Event Matching** for Custom Event entry Journeys.
- **Time Window** — restrict progression to specific days/hours, in the user's time zone when available, otherwise the app default. Releases between window-start and window-start + 15 min to avoid spikes.
- **Yes/No branch** — branch on Segment membership or message behavior (push: clicked, delivered; email: clicked, opened, delivered). Cannot branch on which specific push action button was clicked without a Custom Event workaround.
- **Split Branch** — random distribution across up to 20 branches with custom percentages. Locked once Journey is live. Use to A/B/N test variants or compare against a control group with no message.
- **Tag User** — add or remove a user tag (leave value blank to remove). Recommend adding a 15-minute Wait after a Tag step before sending a webhook or personalized email that depends on the tag.

### Message Steps

- **Push notification** — sends immediately on step entry; user must have an active push subscription.
- **Email** — sends immediately; requires email setup and a subscribed address. Supports Liquid, Data Tags, Data Feeds.
- **SMS** — sends immediately; requires SMS setup and a subscribed phone number.
- **In-app message (IAM)** — requires a new session (app out of focus 30+ seconds, then reopened) to display. Displays only **once per user** per Journey, even on re-entry. IAMs in Journeys are independent copies — duplicating a Journey copies (not references) the IAM.
- **Webhook** — sends an HTTP request to an external system. Webhooks must be created in **Data > Webhooks** before being added to a Journey step. Webhooks cannot call OneSignal APIs.

### Channel Skip Behavior

If a user lacks a subscription for a message step's channel (e.g., no email address on file), that step is skipped and the user continues through the Journey.

## Product Rules

These are the rules OneSignal AI must respect when designing or modifying a Journey.

### Entry and Lifecycle

- A Journey's entry rule type cannot be switched between Segment and Custom Event after going live. Tell the user to duplicate and reconfigure instead.
- Segments referenced in a live Journey's entry rules are locked. Editing requires archiving or removing the Segment from entry rules.
- "Future additions only" locks included/excluded Segments permanently when the Journey goes live. Use only for one-time campaigns where current matchers should be excluded.
- A user can be in multiple Journeys at the same time.
- Custom Event entry rules can put the same user into a Journey multiple times in parallel; each entry carries its own event properties. To prevent that, add a property filter or add an exit rule that matches the same event name.
- If a user matches both entry and exit rules at entry time, they enter the Journey, complete the first step, then exit. Add a Wait step first, or add an Excluded Segment, to prevent this.

### Editing Live Journeys

- Step changes apply immediately to users currently in the Journey, based on where they are.
- Settings changes (entry, exit, re-entry, schedule) affect only users who enter after the change.
- Wait duration changes do not retroactively reset existing timers.
- Updating the **content** of an existing message step preserves stats. Replacing or removing the step resets stats from zero for the new step.
- Split Branches are locked once a Journey is live.
- There is no undo or version history. Recommend duplicating a live Journey before risky edits.

### Re-entry and Recurring

- Re-entry rules apply only to Segment-based Journeys. Custom Event Journeys always allow re-entry.
- For Time Window-based recurring sends, the re-entry duration must be **longer than the time window duration** but shorter than the send interval, to prevent double-sends.

### Custom Events

- Custom Events must be sent from your SDK or the Create Custom Events API. There is no dashboard-only way to fire one.
- Each Journey instance stores up to **100 event properties per user** (oldest dropped). Templates can reference `journey.first_event`, `journey.last_event`, and `journey.event.EVENT_NAME`.
- Use snake_case, past-tense event names (`signup_completed`, `cart_updated`, `payment_failed`). Keep names stable; do not rename events that Journeys depend on.
- Send numbers as numbers, not strings. Numeric filters fail silently otherwise.
- Avoid PII in event properties. Use anonymized IDs.
- Avoid firing the same event from both the SDK and a backend webhook unless deduplicated via `idempotency_key`.

### Webhooks

- Webhook endpoints must be public and reachable over HTTP/HTTPS.
- Webhooks cannot call OneSignal APIs — they are for external systems only.
- Plan for bursts: many users hitting a webhook step at once will produce a burst of HTTP requests. Recommend routing through your own server or a queue.
- Use `event.id` to deduplicate retries.
- Tags are not available in standalone webhook tests; use Test Users in a live Journey for full validation.

### In-App Messages

- IAMs require a new session to display. If the user is in the app when they reach the IAM step, they will not see it until the next session.
- IAMs in a Journey display only once per user per Journey, even on re-entry. To show a similar IAM again, create a new IAM step.
- IAMs are not shared across Journeys. Duplicating a Journey creates independent IAM copies.

### External IDs

- External IDs are strongly recommended for Journeys. Without them, each subscription (email, device, SMS) is treated as a separate user, which can cause duplicate or conflicting messages across channels.

### Plan Limits

- The number of active Journeys is plan-dependent. If a Journey limit is reached, scheduled Journeys will not launch. Advise the user to archive an unused Journey if they hit this.

## Common Defaults and Best Practices

- For onboarding, prefer a Custom Event entry (e.g., `signup_completed`) when available; fall back to a "Subscribed Users" Segment with **Future additions only** if no signup event exists.
- For re-engagement, use Segment entry on `last_session > N days`, exit on "user becomes active", and re-entry after the inactivity window. Define an Excluded Segment that mirrors the inverse condition to avoid users with one inactive and one active subscription qualifying.
- For abandoned cart, use Custom Event entry (e.g., `cart_updated`) and **the same event name as the exit rule** so each new cart update restarts the wait timer with fresh data.
- For recurring (daily/weekly), put a Time Window node as the **first** step and set re-entry longer than the window duration but shorter than the send interval.
- For A/B/N tests, use Split Branch with equal percentages; for >2 variants, nest Split Branches. Optionally leave one branch empty as a control group.
- For limited-entry Journeys, use a `journey_count` tag and exclude users at the cap from the audience.
- For action button branching on push, capture the click in code, send a Custom Event with the action ID as a property, then use Wait Until to branch.
- After a Tag User step, add a ~15-minute Wait before any webhook or personalized message that depends on that tag.
- Mix channels: start with email, follow up with push, escalate with SMS, layer in-app for in-session moments. Avoid stacking the same channel back-to-back.
- Suggest a clear, descriptive Journey name and description so the team can identify it later.
- Summarize human-readable Journey logic (entry, steps, branches, exits) before showing or creating raw configuration.

## Tool Guidance

Use available tools to:

- List existing Journeys when the user references one by name.
- Inspect an existing Journey before explaining or modifying it.
- List Segments and Custom Events the user might use in entry, exit, or branching.
- Validate that referenced Segments and Custom Events exist before proposing them.
- Inspect channel/platform setup (push credentials, email setup, SMS setup, IAM setup) when the Journey depends on that channel.
- Create or update a Journey only **after** summarizing it and receiving explicit user approval.
- For destructive actions (stop, archive, delete), require an additional explicit confirmation and remind the user that delete is irreversible and that archive cannot be resumed.

Do not expose internal tool mechanics to the user.

## Summary and Approval

Before creating or modifying a Journey, summarize it in plain text.

The summary should include:

- Journey name (or suggested name) and description.
- Purpose / business goal.
- Entry rule (Segment-based or Custom Event-based) with specifics.
- Included and excluded Segments (or event property filters).
- "Future additions only" status, when relevant.
- The ordered list of steps in human-readable form (e.g., "Wait 1 day", "Send onboarding email", "Yes/No branch on `onboarding_complete` tag", "Send push if no").
- Branch logic and what each branch does.
- Exit rules.
- Re-entry rules, when relevant.
- Schedule (start, end, recurring cadence).
- Assumptions (defaults you applied because the user did not specify).
- Limitations or warnings (e.g., "Split branches will be locked once live", "IAMs only show once per user per Journey", "Re-entry duration must exceed time window duration").

Ask the user to approve the summarized Journey before creating it. For edits to a live Journey, also call out which changes apply immediately to in-flight users and which apply only to future entrants.

## Examples

### Generic Journey Request

User:

```text
Create a Journey.
```

Behavior:

- Use AskQuestion to ask the Journey purpose.
- Continue collecting only missing inputs after the user answers.
- Do not pre-pick channels, entry type, or schedule.

### Onboarding Welcome Series

User:

```text
Build a welcome flow for new signups: an email today, a push in 2 days if they haven't completed onboarding, and a final email a week later.
```

Behavior:

- Treat purpose, channels, and timing as known.
- Recommend a Custom Event entry on `signup_completed` if available; otherwise propose a "Subscribed Users" Segment with **Future additions only**, and call this out as an assumption.
- Propose a Wait Until or Yes/No branch on the onboarding-complete signal (Custom Event preferred, Segment or Tag as fallback). Ask which signal to branch on if missing.
- Set re-entry to "No".
- Summarize and ask for approval.

### Re-engagement / Win-back

User:

```text
Create a Journey to win back users who haven't opened the app in 7 days.
```

Behavior:

- Entry: Segment where `last_session > 7 days`. Recommend an Excluded Segment that mirrors `last_session ≤ 7 days` for safety on multi-subscription users.
- Exit: "Exit when user becomes active in app/website".
- Re-entry: Yes, after 7 days.
- Suggest a push first, then a 2-day Wait, then a Yes/No branch on activity, then an email with an incentive on the No branch.
- Confirm channels (push + email by default; ask before adding SMS).
- Summarize and ask for approval.

### Abandoned Cart

User:

```text
Send users a push 1 hour after they update their cart and an email 24 hours later if they haven't checked out.
```

Behavior:

- Entry: Custom Event `cart_updated`.
- Exit: same event name `cart_updated` (so each cart update restarts the Journey with fresh data) and a `purchase_completed` exit. Confirm both event names with the user.
- Steps: Wait 1h → push → Wait Until `purchase_completed` (expire after ~23h) → email on the expiration branch.
- Assume push and email channels; confirm.
- Summarize and ask for approval.

### Recurring Weekly

User:

```text
Send a weekly digest email every Friday morning to subscribed users.
```

Behavior:

- Entry: "Subscribed Users" Segment.
- Step 1: Time Window for Fridays in the morning.
- Step 2: Email.
- Re-entry: 7 days. Call out the rule that re-entry must be longer than the time window duration but shorter than the send interval.
- Summarize and ask for approval, including a note that the user should rotate email content over time.

### A/B Test

User:

```text
A/B test two push templates in my onboarding Journey.
```

Behavior:

- Insert a Split Branch with 50/50 percentages where the push step would go.
- Place each template on its own branch.
- Remind the user that Split Branches are locked once the Journey is live, and recommend tagging users with which variant they got so analytics can be filtered later.
- Summarize and ask for approval.

### Channel Not Set Up

User:

```text
Send an SMS reminder 3 days into my onboarding Journey.
```

Behavior:

- If SMS is not set up in the app, explain that SMS messaging requires setup first.
- Use AskQuestion to ask how to proceed (set up SMS, swap to push/email, skip the SMS step, continue anyway).

### Improving an Existing Journey

User:

```text
My re-engagement Journey isn't converting. What can I improve?
```

Behavior:

- Inspect the Journey if a tool is available.
- Look at entry/exit rules, channel mix, message timing, and branching, plus the early exit and CTR metrics.
- Propose specific, named changes (tighten the entry window, swap a generic push for a segmented one, add a Yes/No branch on activity, add an incentive on the No branch).
- Summarize before applying any change. Note which changes affect in-flight users and which only affect new entrants.

### Text Fallback

If structured question UI cannot render, ask the same question in text:

```text
What kind of Journey do you want to build?
1. Welcome / onboarding sequence
2. Re-engagement / win-back
3. Abandoned cart
4. Promotional campaign or product launch
5. Recurring scheduled message
6. Event-driven / behavioral followup
7. Type your answer
```

## Anti-Patterns

Avoid:

- Asking the user for raw Journey API fields or internal step IDs.
- Asking every possible Journey question up front. Start with purpose and entry rule.
- Mixing Segment and Custom Event entry rules in the same Journey (not supported).
- Suggesting Split Branch edits on a live Journey (not editable; require duplication).
- Suggesting entry-type changes on a live Journey without telling the user to duplicate first.
- Putting a message step immediately after the entry node when the audience can match both entry and exit rules — this causes one unintended send before exit. Recommend a Wait or an Excluded Segment.
- Designing a Time Window recurring Journey with re-entry shorter than or equal to the time window duration — this causes double-sends.
- Suggesting that webhooks call OneSignal APIs, or that webhooks have rate limiting from OneSignal.
- Designing IAM-heavy Journeys without explaining the new-session requirement and the once-per-user-per-Journey display rule.
- Recommending the same Custom Event be fired from both an SDK and a backend without deduplication.
- Sending PII inside Custom Event properties or tags.
- Renaming Custom Events or tags that an existing Journey depends on.
- Creating, launching, archiving, or deleting a Journey without summarizing it and receiving explicit user approval.
- Inventing unsupported step types, unsupported entry/exit conditions, or unsupported channels.
