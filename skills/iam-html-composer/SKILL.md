---
name: onesignal-iam-html-composer
description: Use this skill when a user wants to create, refine, restyle, or generate HTML for a OneSignal mobile in-app message (IAM). Triggers on phrases like "create an in-app message", "make an IAM", "compose an in-app", "build an HTML IAM", "design an in-app for X", "soft push prompt", "permission prompt", "feature announcement modal", "onboarding modal", "promo modal", "in-app popup", or any request to generate or edit IAM HTML. Prefer this skill even when the user says "popup", "modal", or "splash" if the target surface is a OneSignal mobile in-app.
allowed-tools:
  - ask_user
  - apply_to_iam_composer
  - generate_brand_kit_from_website
---

# OneSignal HTML In-App Message Composer

## Purpose

Turn a plain-language request into production-ready, mobile-safe, brand-aware HTML that can be applied directly to the OneSignal HTML IAM editor.

Every output must be anchored to a known IAM use case, a goal, and a CTA action that maps to a OneSignal IAM JavaScript API method. Do not generate unconstrained "anything" HTML.

## When to Use

Use this skill when the user asks to:

- Create a new HTML in-app message.
- Refine, restyle, or fix an existing IAM.
- Build a soft push or location permission prompt.
- Build a feature announcement, onboarding step, or promo modal.
- Translate a campaign brief, marketing prompt, or website style into an IAM.
- Adapt brand inputs (URL, image, description) into an IAM design.

Treat "in-app", "popup", "modal", "splash", "permission prompt", "rating prompt", and "promo modal" as likely IAM requests unless the user clearly means push, email, SMS, or a web-only surface.

## Core Behavior

When this skill is active:

1. Anchor every output to a known use case and a primary CTA — these are the only two hard anchors. Without them, do not generate HTML.
2. Apply this context precedence when gathering style and content:
   1. The user's request.
   2. Existing campaign or IAM context, if attached or pasted.
   3. Brand Kit — existing brand context the conversation or page already provides, or `generate_brand_kit_from_website` when the user gives a website URL and the tool is offered (it is feature-flag gated).
   4. User-provided fallback brand inputs (colors, description, image).
   5. Neutral defaults, surfaced in the summary.
3. Infer aggressively before asking. Mine the prompt, any brand context already in the conversation, and any pasted IAM HTML for use case, CTA, copy, and goal. Skip questions whose answers are already implied.
4. Spend the single `ask_user` call (up to 3 question blocks) on items that materially improve the output. Two kinds of questions count:
   - **Blocking** — use case anchor and primary CTA, when not implied by the prompt.
   - **Quality-lifting** — the use-case-specific follow-ups that turn a generic IAM into a relevant one (specific value prop, lead benefit, offer specifics, urgency framing, secondary action, personalization). See ask_user Guidance for the patterns per use case.
5. Never ask standing brand-detail questions about colors, fonts, tone, copy direction, goal, or layout. Brand context comes from the Brand Kit or user-provided inputs (see Brand Source); if it is missing, use safe defaults and surface every default explicitly so the user can correct it.
6. Infer presentation style (density, illustration, layout) from the use case — soft push prompts stay compact; onboarding leans illustrative; promo modals lean bold.
7. Summarize the proposed IAM before generating HTML.
8. Require explicit user approval before emitting HTML.
9. Self-check the HTML for rendering soundness, brand fidelity, instruction following, and IAM API correctness before returning it.
10. After approval, show the complete HTML document to the user FIRST in exactly one fenced `html` code block, THEN call `apply_to_iam_composer` with `{ "html": "..." }` containing the same content. The dashboard renders a fixed "Apply to editor" button; the content replaces whatever is currently in the active editor.
11. For iterative edits, regenerate the FULL document — preserve unchanged sections from the previous version, fold in the requested change, then show the updated HTML and call `apply_to_iam_composer` again. Consolidate all of a response's changes into one tool call so the user sees a single Apply button.
12. After the tool call, the backend returns a synthetic result confirming the Apply button is visible and the turn continues — keep any follow-up text brief. Never claim the IAM is applied, saved, or published: the user must click "Apply to editor", and nothing is published. Never print raw JSON action blobs; the tool call itself creates the dashboard button. Do not say "Applying it now" unless the tool was actually called or a full HTML document is visible in the response.
13. `apply_to_iam_composer` is offered only while the user has the IAM composer open (the frontend reports the `iam_composer` UI target). If the tool is absent, show the HTML in one fenced code block and tell the user to open the in-app message HTML editor to apply it.

## Required Inputs

Only two inputs are hard requirements. Without them, the output cannot be valid and the skill must clarify:

- **Use case anchor** — a known IAM category (soft push prompt, feature announcement, onboarding, promo, rating prompt, etc.). The skill must never produce unconstrained "anything" HTML.
- **Primary CTA** — must map to a specific `OneSignalIamApi` method.

Everything else has a safe default and must not block generation:

- **Goal** — infer from the use case (push prompts → opt-in, announcements → adoption, onboarding → activation). Confirm in the summary.
- **Brand** — default to a neutral mobile-first center modal if no URL, image, or description is given.
- **Copy** — write it if not supplied. Surface a sample headline in the summary.
- **Layout and constraints** — apply standard mobile defaults (dimensions, padding, dismiss control, accessibility) unless the user specifies otherwise.

## Brand Source

Apply brand context in this order:

1. **Existing Brand Kit or brand context** — if the conversation or page already provides brand colors, fonts, or tone, use them directly.
2. **Website URL via `generate_brand_kit_from_website`** — if the user gives a website URL and the tool is offered (it is feature-flag gated), call it with `{ "url": "https://..." }`. It returns `{ suggestions, current }`: prefer `current` (the saved Brand Kit) when populated; otherwise style from `suggestions` (colors, fonts, logos, tone sliders). The tool saves nothing — treat the result as proposals to style from. Never call `save_brand_kit` from this skill (it is not in this skill's tool list); if the user wants to save the result to their Brand Kit, direct them to Brand Kit settings.
3. **User-provided colors or description** — hex codes, brand adjectives, or a described image.
4. **Neutral defaults** — surface every defaulted style in the pre-generation summary so the user can correct it.

If the brand tool is unavailable or gated and no colors were given, one `ask_user` question for brand colors is allowed; otherwise use neutral defaults and surface that in the summary.

## ask_user Guidance

`ask_user` presents 1–3 questions in a single multi-step panel; all answers come back at once. Each question carries up to 4 predefined options, each with a short label (2–5 words) and an optional one-line description. By default (`allow_free_text: true`) the panel renders a built-in "Something else" free-text input below the options — never spend an option slot on "Type your answer" or "Other / Custom". Set `allow_multiple: true` for multi-select questions. Use the optional `title` (short task header, e.g. "In-App Message Setup") and `subtitle` (one sentence on what you will do with the answers).

Spend the single call on questions that either unblock generation or measurably improve the output. Do not ask standing brand-detail questions about colors, fonts, tone, copy direction, goal, or layout. When brand context is missing and `generate_brand_kit_from_website` is offered, one brand-source question for a customer website URL is allowed because the tool can replace several brand-detail questions; when the tool is unavailable or gated, ask for brand colors instead (see Brand Source).

### Inference Comes First

Before considering a question, mine these signals:

- **Use case** — keywords like "permission", "opt-in", "push" → Soft Push Permission Prompt; "new feature", "announce", "introducing" → Feature Announcement; "welcome", "setup", "getting started" → Onboarding; "promo", "sale", "deal", "bonus" → Promo; "rate", "review" → Rating; "subscribe", "newsletter" → Capture.
- **Primary CTA** — verbs like "open", "navigate", "view" → `openUrl`; "subscribe", "enable", "allow" → `triggerPushPrompt` or `triggerLocationPrompt` depending on context; "tag", "save preference" → `tagUser`; "claim", "record", "track" → `sendOutcome`; "dismiss", "close" → `close`.
- **Brand** — any URL → `generate_brand_kit_from_website` (when the gated tool is offered); prefer `current` when the saved Brand Kit is populated, otherwise style from `suggestions`.
- **Copy** — quoted text in the prompt is treated as supplied copy. Otherwise plan to author it.
- **Goal** — derive from use case unless the prompt contradicts the obvious mapping.

Only what remains genuinely ambiguous after this pass is a candidate to ask about.

### Two Types of Clarifying Questions

Every question must serve one of these purposes:

1. **Blocking** — without it, the output cannot be valid. The two hard blockers are the **use case anchor** and the **primary CTA**. Ask only when neither the prompt, brand tool result, nor pasted HTML implies them.
2. **Quality-lifting** — without it, the output is valid but generic. These probe the specifics that turn a template into a relevant IAM: the actual value prop, the offer specifics, the lead benefit, urgency framing, audience tone, secondary action style, personalization vectors.

Brand source, copy direction, goal, layout, dismiss presence, and dark mode handling are **never asked**. Default them and surface the defaults in the pre-generation summary so the user can correct them.

### Quality-Lifting Questions Per Use Case

Even when a prompt looks complete, the right one or two follow-ups can take an IAM from generic to specific. Pick from these patterns based on what's missing:

**Soft Push Permission Prompt**
- What specific value will the user get by opting in? ("reminders" — what kind, how often, about what?)
- Is there a "Maybe later" secondary action, or a single dismiss?

**Feature Announcement**
- Which single benefit should lead the headline? (e.g. dark mode → easier on eyes vs. saves battery vs. matches system)
- Does the CTA open the feature directly, deep-link to settings, or open a learn-more page?
- Is the audience all users, new users, or a specific segment (affects tone)?

**Multi-Step Onboarding**
- Which step in the sequence is this, and should a progress indicator be shown?
- What's the user's next step after dismissing this IAM?

**Promo or Sale**
- What is the actual offer (specific amount, %, item, or trial length)?
- Is there urgency to surface (expiration date, "today only", live countdown), or is this evergreen?
- Is the offer universal, or personalized to the recipient?

**Rating Prompt**
- What just happened to earn the prompt (post-purchase, completed task, milestone)?
- Should users who don't love it branch to a feedback flow, or just dismiss?

**Personalization (applies across use cases)**
- Are there user tags worth surfacing (e.g. `{{first_name}}`, plan tier, last action)? Propose 1–2 concrete Liquid options in the question rather than asking abstractly.

Use at most one or two of these in addition to any blocking question. Skip any whose answer is already in the prompt.

### Product-Owned Palettes

Draw from these closed sets when the question fits. The built-in "Something else" free-text input (on by default) covers custom answers — never list it as an option. To make a question strictly closed, set `allow_free_text: false`.

**Use case palette** (for the blocking use-case question):

- Soft Push Permission Prompt
- Feature Announcement
- Multi-Step Onboarding
- Promo or Sale

Rating prompts, capture/newsletter, and anything else arrive via "Something else".

**Primary action palette** (for the blocking CTA question), naming the `OneSignalIamApi` method in each option's description:

- Open a link or deep link (`OneSignalIamApi.openUrl`)
- Trigger the push permission prompt (`OneSignalIamApi.triggerPushPrompt`)
- Tag the user (`OneSignalIamApi.tagUser`)
- Record a custom outcome (`OneSignalIamApi.sendOutcome`)

Location prompts (`OneSignalIamApi.triggerLocationPrompt`) and plain dismiss (`close`) come in via free text or inference.

For quality-lifting questions, build a closed set of 2–4 plausible options tailored to the user's domain (e.g. for a cooking app: "Saved-recipe alerts / Weekly featured recipes / Cooking-step timers"). Free text is always present, so never present an open text box alone.

### Combining

When multiple answers are missing, put up to three question blocks in the one `ask_user` call rather than asking in rounds. Common combinations:

- Use case + CTA, when the prompt is bare.
- CTA destination + lead benefit, when the type is known but the action and headline emphasis are open.
- Offer specifics + urgency framing, for promos with a CTA already named.

### One Call, Hard Cap

One `ask_user` call per distinct user request — a single call carrying up to 3 question blocks, each with at most 4 options. Never ask a second round unless the user pivots to a clearly new task. If even three questions would not be enough, generate with explicit defaults, surface every assumption in the summary, and let the user iterate. Do not call `suggest_reply_to_user` in the same turn as `ask_user`.

## Product Rules

OneSignal HTML IAMs run in a sandboxed WebView. Every output must follow these rules.

### Dashboard Apply Contract

When `apply_to_iam_composer` is available (it is offered only while the user has the IAM composer open), use it instead of telling the user to copy and paste. Do not print the JSON payload.

The order is fixed: show the complete HTML document to the user FIRST, in exactly one fenced `html` code block, THEN call `apply_to_iam_composer` with the same content:

```json
{
  "html": "<!DOCTYPE html>..."
}
```

`html` is the only input field. There is no label parameter — the dashboard renders a fixed "Apply to editor" button. The content replaces whatever is currently in the active editor.

After the tool call, the backend returns a synthetic result ("Content delivered to the user. An 'Apply' button is now visible...") and the turn continues. Keep any follow-up text brief — at most a short closing line. Never re-print the HTML after the call, never claim the IAM is applied or saved (the user must click the button), and never promise post-apply confirmation. Nothing is published; the button only sets editor content client-side after the user clicks.

One apply per response: consolidate all changes into a single `apply_to_iam_composer` call so the user sees exactly one Apply button.

There is no diff tool. All edits, however small, are full-document: regenerate the complete HTML with the requested change — preserving unchanged sections from the previous version — and call `apply_to_iam_composer` again.

If `apply_to_iam_composer` is not in the tool list, the composer is not open. Show the HTML in one fenced code block and tell the user to open the in-app message HTML editor to apply it. That is the legitimate fallback — never fabricate a tool call or print raw JSON action payloads.

### Interaction API

Use only these `OneSignalIamApi` methods for interactivity:

- `OneSignalIamApi.openUrl(e, url)` — open an external URL or deep link; closes the IAM.
- `OneSignalIamApi.triggerPushPrompt(e)` — show the native push permission prompt.
- `OneSignalIamApi.triggerLocationPrompt(e)` — show the native location permission prompt.
- `OneSignalIamApi.tagUser(e, { key: "value" })` — tag the user for segmentation.
- `OneSignalIamApi.sendOutcome(e, "outcome_name")` — record an unattributed custom outcome.
- `OneSignalIamApi.addClickName(e, "click_name")` — set a click identifier readable by the mobile SDK click listener.
- `OneSignalIamApi.trackClick(e)` — manually track a click; required before any custom navigation.
- `OneSignalIamApi.close(e)` — dismiss the IAM.

Do not invent methods that are not in this list.

### Click Handling

- Every clickable element, including CTAs, secondary buttons, close/icon controls, and anything with a click listener, must include `data-onesignal-unique-label` with a short, semantic, unique kebab-case value for that specific action (`allow-notifications`, `maybe-later`,`close`, `claim-offer`). Never use the literal string `data-onesignal-unique-label` as the value; that is the attribute name, not the label.
- Prefer `<button>` over `<a>`. `<a target>` and `window.open()` are not tracked and may not navigate in the sandbox.
- Bind event listeners after the document is ready. The IAM runtime may inject HTML after the host page's `DOMContentLoaded` has already fired, so a bare `document.addEventListener('DOMContentLoaded', init)` can silently never run. Use a `readyState` check, or place the script at the end of `<body>` and call init directly:

  ```js
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
  } else {
    init();
  }
  ```

- Always call `OneSignalIamApi.trackClick(e)` before any custom navigation or call to `openUrl()`.

### Liquid Personalization

- Liquid works in element text, `<style>` blocks, and supported attributes (`href`, `src`, `action`, `data`).
- Liquid does NOT substitute inside `<script>` tags. To use tag values in JavaScript, read from the global `liquidPlayerTags` object, which is available after `DOMContentLoaded`.
- Every Liquid variable must include a default filter, e.g. `{{ first_name | default: 'there' }}`.
- **Personalization is opt-in, not a default.** Do not add Liquid variables to the HTML unless the user explicitly asked for personalization, the prompt contains a tag-like reference (e.g. "greet them by name"), or the user approved a personalization suggestion in the pre-generation summary. If you think personalization would help, surface it as a "would you like to add?" item in the summary — never bake it into the HTML silently.

### Mobile Rendering

- Use safe-area insets: `var(--safe-area-inset-top)`, `--safe-area-inset-right`, `--safe-area-inset-bottom`, `--safe-area-inset-left`.
- Define explicit colors for both light and dark mode. Never rely on system defaults. Use `@media (prefers-color-scheme: dark)`.
- Use responsive CSS; defaults must look correct at common mobile widths.
- For dynamic type on iOS, use `font: -apple-system-body` where appropriate.
- Web fonts: include `font-display: swap`. Keep payload small — avoid heavy base64.
- Older Android (below SDK 4.6.3) falls back to Center Modal — design must degrade gracefully.
- For autoplay video, embed via `<iframe>` with `&mute=1` and `allow="autoplay; encrypted-media"`. Keep videos short.

## Common Defaults and Best Practices

When brand context is missing, default to:

- **Layout:** center modal, ~320–360 px content width, vertical stack.
- **Typography:** system font stack (`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`).
- **Color:** white background in light mode, `#1c1c1e` in dark mode, single accent color on the primary CTA.
- **Actions:** primary CTA + dismiss control.
- **Padding:** generous; safe-area insets respected.
- **Accessibility:** ≥4.5:1 contrast for text, ≥44 px touch targets, descriptive button labels.

Surface every assumption in the pre-generation summary so the user can correct it.

## Tool Guidance

- **generate_brand_kit_from_website:** call when the user provides a website URL and the tool is offered (it is feature-flag gated). Prefer `current` (the saved Brand Kit) when populated; otherwise style from `suggestions` (colors, fonts, logos, tone). It saves nothing — never call `save_brand_kit` from this skill (it is not in this skill's tool list); if the user wants to update their Brand Kit, direct them to Brand Kit settings.
- **ask_user:** one call per distinct request, up to 3 question blocks with at most 4 options each; the built-in "Something else" free text covers custom answers. Do not call `suggest_reply_to_user` in the same turn.
- **apply_to_iam_composer:** call after approval. Show the complete HTML document in exactly one fenced `html` code block first, then call with `{ "html": ... }`. The dashboard renders an "Apply to editor" button; the content fully replaces the editor. Available only while the user has the IAM composer open — when absent, deliver the code block only and point the user to the in-app message editor. Do not print raw JSON or copy/paste instructions when the tool is available.
- **No publish/save actions:** the "Apply to editor" button only updates the local IAM HTML editor after user click. Do not attempt to publish, save, or launch the IAM, and do not claim it was applied.

## Summary and Approval

Before emitting HTML, summarize:

- IAM type, goal, and primary CTA mapped to the exact `OneSignalIamApi` method.
- Brand application (source, confidence, and key inferred styles like primary color, font, tone).
- Copy direction (supplied or AI-written, with a sample headline if generated).
- Liquid variables used and their default fallbacks.
- Any defaults the AI is filling in (layout, dimensions, dismiss control, etc.).

Require explicit approval ("looks good", "generate it") before producing HTML. On a revision request, re-summarize the delta; do not silently regenerate.

## Self-Check Before Returning HTML

Three quick checks before emitting HTML:

- Every clickable element has a unique `data-onesignal-unique-label`.
- Every interaction maps to a real `OneSignalIamApi` method, with `trackClick(e)` before any `openUrl()`.
- Event listeners use the readyState-safe initializer pattern from Click Handling.

## Examples

### Example 1 — Soft push prompt with brand and headline

User prompt:

```text
Make a soft push prompt for our cooking app. Brand is warm orange #FF6B35,
friendly tone, headline "Get recipe reminders".
```

Inferred: type (Soft Push Permission Prompt → `triggerPushPrompt`), brand color, tone, headline.

Quality lifts to consider: the headline "Get recipe reminders" is generic — what kind of reminders meaningfully differs. Secondary action style (Allow + Maybe Later vs. Allow + dismiss only) also shapes the layout.

Behavior: ask one combined `ask_user` call covering both — specific reminder content ("Saved-recipe alerts / Weekly featured recipes / Cooking-step timers") and secondary action ("Include 'Maybe later' button / Single dismiss only"); the built-in "Something else" input covers anything custom. Default copy beyond the headline to AI-written. If personalization would help, surface it in the summary as an optional add (e.g. "Add `{{first_name | default: 'there'}}` greeting? — y/n"); do not include Liquid in the HTML unless the user opts in. Generate on approval: show the full HTML in one code block, then call `apply_to_iam_composer`.

### Example 2 — Feature announcement with brand URL

User prompt:

```text
Create a feature announcement IAM for our new dark mode. Brand: https://acme.com.
```

Inferred: type (Feature Announcement), brand source (URL).

Quality lifts to consider: dark mode is most often pitched on one of three benefits, and the CTA destination is ambiguous (open the feature, deep-link to settings, or open a what's-new page). These shape the entire IAM.

Behavior: call `generate_brand_kit_from_website` with `acme.com` first (if the gated tool is offered) — prefer `current` when the saved Brand Kit is populated, otherwise style from `suggestions` (colors, fonts, logos, tone); if the tool is unavailable, fall back to neutral defaults and surface that. Then ask one combined `ask_user` covering the lead benefit ("Easier on the eyes / Saves battery / Matches your system") and the CTA destination ("Deep-link to settings to enable now / Open the dark-mode feature directly / Open a learn-more page"). Default copy direction; surface a sample headline tied to the chosen benefit in the summary. Generate on approval.

### Example 3 — Vague request

User prompt:

```text
I need an in-app for our app.
```

Inferred: nothing. Both hard anchors are missing.

Behavior: ask one combined `ask_user` call that surfaces both the use case palette and the primary action palette in a single panel — each question carries at most 4 options, with the built-in "Something else" free text covering anything custom. Default brand to neutral, copy to AI-written, goal to whatever the use case implies, layout to center modal. Summarize all defaults and generate on approval. Do not ask follow-ups about brand, copy, goal, or constraints.

### Example 4 — Refining existing HTML

User prompt: pastes IAM HTML and says "Make this match my dark mode brand."

Inferred: type and CTAs from the pasted HTML.

Behavior: parse the pasted HTML to preserve unrelated structure. Regenerate the full document with only the requested change — keep untouched sections intact from the pasted version — then show the complete updated HTML in one fenced `html` code block and call `apply_to_iam_composer` with the same content (it replaces the editor content; there is no partial-diff path). If a brand URL was not given, ask one focused question for the dark-mode brand color or accent (or accept "use neutral"). Re-run the self-check before returning.

### Example 5 — Outcome-focused promo

User prompt:

```text
Build a promo modal that records an outcome called "claimed_bonus" when the user taps Claim.
```

Inferred: type (Promo), CTA (`sendOutcome`), outcome name, button label.

Quality lifts to consider: `claimed_bonus` is the outcome name but does not say what the bonus actually is, which is the whole headline. Urgency framing (live countdown vs. expiration date vs. evergreen) also dramatically changes the design.

Behavior: ask one combined `ask_user` covering the offer specifics ("$5 credit / 20% off / Free item / Extended trial") and urgency framing ("Live countdown / Expires today / Expires this week / Evergreen") — four options each, with "Something else" free text covering the rest. Default brand to neutral, copy to AI-written. If personalization would help, surface it in the summary as an optional add (e.g. "Add `{{first_name | default: 'there'}}` greeting? — y/n"); do not include Liquid in the HTML unless the user opts in. Generate the HTML wiring the Claim button to `OneSignalIamApi.sendOutcome(e, "claimed_bonus")` with a unique label and dismiss control.

## Anti-Patterns

Avoid:

- Generating HTML before the use case and primary CTA are settled.
- Skipping to generation when an obvious quality-lift question would specify a vague value prop, offer, urgency, or audience.
- Asking standing questions about brand source, copy, goal, or layout — those have safe defaults and belong in the summary, not in a question.
- Asking sequential question rounds when one combined `ask_user` call would do.
- Asking any clarifying question when the prompt, brand tool result, or pasted HTML already implies the answer.
- Presenting quality-lifting questions as open text fields instead of closed sets of 2–4 plausible options (the "Something else" free text renders automatically).
- Putting more than 4 options on a question, or wasting an option slot on "Type your answer" / "Other / Custom".
- Calling `ask_user` a second time for the same request, or calling `suggest_reply_to_user` in the same turn.
- Treating `generate_brand_kit_from_website` `suggestions` as saved brand data, or calling `save_brand_kit` at all (it is not in this skill's tool list and replaces the entire Brand Kit — direct users to Brand Kit settings instead).
- Telling dashboard IAM Compose users to copy and paste generated HTML when `apply_to_iam_composer` is available.
- Calling `apply_to_iam_composer` before showing the HTML code block, calling it more than once per response, or re-printing the HTML after the call.
- Namespacing `data-onesignal-unique-label` values verbosely (e.g. `myapp-onboarding-step-1-next`) when a short value like `next-1` is unique within the IAM.
- Relying on system dark mode without explicit color overrides.
- Printing raw `apply_to_iam_composer` JSON input as prose instead of calling the tool.
- Saying "Applying it now" or claiming the IAM is applied/saved without calling `apply_to_iam_composer` — or, when the tool is unavailable, without showing the full HTML. The user must click "Apply to editor"; nothing happens until they do.
- Returning prose alongside the HTML unless the user asked for an explanation.
- Inventing OneSignal methods that are not in the API list above.
- Producing "anything" HTML that is not anchored to a use case and CTA.

## Test Prompts

Suggested prompts to validate the skill behaves correctly:

1. "Create an in-app message." — should ask one combined blocking question covering use case + primary action in a single `ask_user` call; each question carries at most 4 options, with the built-in "Something else" free text covering Other/Custom. Default brand and copy.
2. "Create a soft push prompt for my fitness app. Brand: https://example.com." — should call `generate_brand_kit_from_website` on example.com first (if the gated tool is offered), then ask one combined quality-lifting question covering value-prop specifics and secondary action style. Do not ask about brand, copy, layout, or goal.
3. "Make this in-app match our brand" (with pasted HTML) — should preserve the pasted structure while editing, then show and apply the FULL updated document via `apply_to_iam_composer` — never a diff. May ask one focused question about dark-mode accent or brand specifics. Re-run the self-check.
4. "Build a promo modal that records 'claimed_bonus' when the user taps Claim." — should ask one combined quality-lifting question covering offer specifics and urgency framing. Default brand and copy.
5. "Make a soft push prompt for our cooking app. Brand is warm orange #FF6B35, friendly tone, headline 'Get recipe reminders'." — should NOT skip to generation. Should ask one combined question lifting the reminder specifics and secondary action style.
