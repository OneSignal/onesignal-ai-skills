---
name: onesignal-iam-html-composer
description: Use this skill when a user wants to create, refine, restyle, or generate HTML for a OneSignal mobile in-app message (IAM). Triggers on phrases like "create an in-app message", "make an IAM", "compose an in-app", "build an HTML IAM", "design an in-app for X", "soft push prompt", "permission prompt", "feature announcement modal", "onboarding modal", "promo modal", "in-app popup", or any request to generate or edit IAM HTML. Prefer this skill even when the user says "popup", "modal", or "splash" if the target surface is a OneSignal mobile in-app.
allowed-tools:
  - AskUserQuestion
  - WebFetch
---

# OneSignal HTML In-App Message Composer

## Purpose

Turn a plain-language request into production-ready, mobile-safe, brand-aware HTML that can be pasted directly into the OneSignal HTML IAM editor.

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
   3. Brand Center (not yet available — skip until implemented).
   4. Customer website URL fetched with WebFetch — infer colors, fonts, CTA voice, tone, product vocabulary, logo.
   5. User-provided fallback brand inputs.
3. Infer aggressively before asking. Mine the prompt, URL fetch results, and any pasted IAM HTML for use case, CTA, brand, copy, and goal. Skip questions whose answers are already implied.
4. Spend the 1–3 question budget on items that materially improve the output. Two kinds of questions count:
   - **Blocking** — use case anchor and primary CTA, when not implied by the prompt.
   - **Quality-lifting** — the use-case-specific follow-ups that turn a generic IAM into a relevant one (specific value prop, lead benefit, offer specifics, urgency framing, secondary action, personalization). See AskUserQuestion Guidance for the patterns per use case.
5. Never ask standing questions about brand source, copy direction, goal, or layout — those have safe defaults and belong in the summary, not in a question. Surface every default explicitly so the user can correct it.
6. Infer presentation style (density, illustration, layout) from the use case — soft push prompts stay compact; onboarding leans illustrative; promo modals lean bold.
7. Summarize the proposed IAM before generating HTML.
8. Require explicit user approval before emitting HTML.
9. Self-check the HTML for rendering soundness, brand fidelity, instruction following, and IAM API correctness before returning it.
10. Return clean HTML only, no surrounding explanation, unless the user asked for one.

## Required Inputs

Only two inputs are hard requirements. Without them, the output cannot be valid and the skill must clarify:

- **Use case anchor** — a known IAM category (soft push prompt, feature announcement, onboarding, promo, rating prompt, etc.). The skill must never produce unconstrained "anything" HTML.
- **Primary CTA** — must map to a specific `OneSignalIamApi` method.

Everything else has a safe default and must not block generation:

- **Goal** — infer from the use case (push prompts → opt-in, announcements → adoption, onboarding → activation). Confirm in the summary.
- **Brand** — default to a neutral mobile-first center modal if no URL, image, or description is given.
- **Copy** — write it if not supplied. Surface a sample headline in the summary.
- **Layout and constraints** — apply standard mobile defaults (dimensions, padding, dismiss control, accessibility) unless the user specifies otherwise.

If the user provides a website URL, WebFetch it before doing anything else and use it as the brand source. That eliminates the need for brand questions entirely.

## AskUserQuestion Guidance

Spend the 1–3 question budget on questions that either unblock generation or measurably improve the output. Standing questions about brand source, copy direction, goal, or layout waste the budget — those have safe defaults and belong in the summary.

### Inference Comes First

Before considering a question, mine these signals:

- **Use case** — keywords like "permission", "opt-in", "push" → Soft Push Permission Prompt; "new feature", "announce", "introducing" → Feature Announcement; "welcome", "setup", "getting started" → Onboarding; "promo", "sale", "deal", "bonus" → Promo; "rate", "review" → Rating; "subscribe", "newsletter" → Capture.
- **Primary CTA** — verbs like "open", "navigate", "view" → `openUrl`; "subscribe", "enable", "allow" → `triggerPushPrompt` or `triggerLocationPrompt` depending on context; "tag", "save preference" → `tagUser`; "claim", "record", "track" → `sendOutcome`; "dismiss", "close" → `close`.
- **Brand** — any URL → WebFetch immediately and derive palette, fonts, tone. Skip brand questions afterward.
- **Copy** — quoted text in the prompt is treated as supplied copy. Otherwise plan to author it.
- **Goal** — derive from use case unless the prompt contradicts the obvious mapping.

Only what remains genuinely ambiguous after this pass is a candidate to ask about.

### Two Types of Clarifying Questions

Every question must serve one of these purposes:

1. **Blocking** — without it, the output cannot be valid. The two hard blockers are the **use case anchor** and the **primary CTA**. Ask only when neither the prompt, URL fetch, nor pasted HTML implies them.
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

Draw from these closed sets when the question fits, and always include `Type your answer`.

**Use case palette** (for the blocking use-case question):

- Soft Push Permission Prompt
- Feature Announcement
- Multi-Step Onboarding
- Promo or Sale
- Other / Custom
- Type your answer

**Primary action palette** (for the blocking CTA question):

- Open a link or deep link (`OneSignalIamApi.openUrl`)
- Trigger the push permission prompt (`OneSignalIamApi.triggerPushPrompt`)
- Trigger the location permission prompt (`OneSignalIamApi.triggerLocationPrompt`)
- Tag the user (`OneSignalIamApi.tagUser`)
- Record a custom outcome (`OneSignalIamApi.sendOutcome`)
- Other / Custom
- Type your answer

For quality-lifting questions, build a closed set of 3–5 plausible options tailored to the user's domain (e.g. for a cooking app: "saved-recipe alerts / weekly featured recipes / cooking-step timers / Type your answer"). Do not present an open text box alone.

### Combining

When multiple answers are missing, prefer one AskUserQuestion with two question blocks over two sequential rounds. Common combinations:

- Use case + CTA, when the prompt is bare.
- CTA destination + lead benefit, when the type is known but the action and headline emphasis are open.
- Offer specifics + urgency framing, for promos with a CTA already named.

### Hard Cap

Never exceed three clarifying questions across the entire flow. If even three would not be enough, generate with explicit defaults, surface every assumption in the summary, and let the user iterate.

## Product Rules

OneSignal HTML IAMs run in a sandboxed WebView. Every output must follow these rules.

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

- Every clickable element must have a unique `data-onesignal-unique-label`. The attribute name is fixed by OneSignal — the runtime reads it for click tracking. The *value* should be short and descriptive (`next-1`, `view-roadmap`, `close`); unique within the IAM is enough, no need for global namespacing.
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

- **WebFetch:** call on any customer website URL the user provides. Treat fetched content as the brand source of truth and use it before asking fallback brand questions.
- **AskUserQuestion:** use for the closed sets above and only for missing answers. Never exceed three questions.
- **No write actions:** the user pastes the final HTML into the OneSignal HTML IAM editor themselves. Do not attempt to publish or save.

## Summary and Approval

Before emitting HTML, summarize:

- IAM type, goal, and primary CTA mapped to the exact `OneSignalIamApi` method.
- Brand application (source + key inferred styles like primary color, font, tone).
- Copy direction (supplied or AI-written, with a sample headline if generated).
- Liquid variables used and their default fallbacks.
- Any defaults the AI is filling in (layout, dimensions, dismiss control, etc.).

Require explicit approval ("looks good", "generate it") before producing HTML. On a revision request, re-summarize the delta; do not silently regenerate.

## Self-Check Before Returning HTML

Privately verify, before returning the HTML:

- Every clickable element has a unique `data-onesignal-unique-label`.
- All interactions use `OneSignalIamApi` methods. No `window.open`, no untracked `<a target>`.
- `OneSignalIamApi.trackClick(e)` precedes any custom navigation or `openUrl()` call.
- Event listeners are bound inside a `DOMContentLoaded` handler.
- No Liquid appears inside `<script>` tags; JS tag access uses `liquidPlayerTags`.
- Every Liquid variable has a `default:` filter.
- Liquid is only present when the user explicitly requested or approved personalization.
- Explicit light and dark mode colors are defined for text, background, and buttons.
- Safe-area insets are respected.
- HTML is self-contained, mobile-first, and matches the approved summary.

## Examples

### Example 1 — Soft push prompt with brand and headline

User prompt:

```text
Make a soft push prompt for our cooking app. Brand is warm orange #FF6B35,
friendly tone, headline "Get recipe reminders".
```

Inferred: type (Soft Push Permission Prompt → `triggerPushPrompt`), brand color, tone, headline.

Quality lifts to consider: the headline "Get recipe reminders" is generic — what kind of reminders meaningfully differs. Secondary action style (Allow + Maybe Later vs. Allow + dismiss only) also shapes the layout.

Behavior: ask one combined AskUserQuestion covering both — specific reminder content ("Saved-recipe alerts / Weekly featured recipes / Cooking-step timers / Type your answer") and secondary action ("Include 'Maybe later' button / Single dismiss only / Type your answer"). Default copy beyond the headline to AI-written. If personalization would help, surface it in the summary as an optional add (e.g. "Add `{{first_name | default: 'there'}}` greeting? — y/n"); do not include Liquid in the HTML unless the user opts in. Generate on approval.

### Example 2 — Feature announcement with brand URL

User prompt:

```text
Create a feature announcement IAM for our new dark mode. Brand: https://acme.com.
```

Inferred: type (Feature Announcement), brand source (URL).

Quality lifts to consider: dark mode is most often pitched on one of three benefits, and the CTA destination is ambiguous (open the feature, deep-link to settings, or open a what's-new page). These shape the entire IAM.

Behavior: WebFetch `acme.com` first to extract palette, fonts, voice, logo. Then ask one combined AskUserQuestion covering the lead benefit ("Easier on the eyes / Saves battery / Matches your system / Type your answer") and the CTA destination ("Deep-link to settings to enable now / Open the dark-mode feature directly / Open a learn-more page / Type your answer"). Default copy direction; surface a sample headline tied to the chosen benefit in the summary. Generate on approval.

### Example 3 — Vague request

User prompt:

```text
I need an in-app for our app.
```

Inferred: nothing. Both hard anchors are missing.

Behavior: ask one combined question that surfaces both the use case palette and the primary action palette in a single round-trip. Default brand to neutral, copy to AI-written, goal to whatever the use case implies, layout to center modal. Summarize all defaults and generate on approval. Do not ask follow-ups about brand, copy, goal, or constraints.

### Example 4 — Refining existing HTML

User prompt: pastes IAM HTML and says "Make this match my dark mode brand."

Inferred: type and CTAs from the pasted HTML.

Behavior: parse the pasted HTML to preserve unrelated structure. If a brand URL was not given, ask one focused question for the dark-mode brand color or accent (or accept "use neutral"). Re-run the self-check before returning.

### Example 5 — Outcome-focused promo

User prompt:

```text
Build a promo modal that records an outcome called "claimed_bonus" when the user taps Claim.
```

Inferred: type (Promo), CTA (`sendOutcome`), outcome name, button label.

Quality lifts to consider: `claimed_bonus` is the outcome name but does not say what the bonus actually is, which is the whole headline. Urgency framing (live countdown vs. expiration date vs. evergreen) also dramatically changes the design.

Behavior: ask one combined AskUserQuestion covering the offer specifics ("$5 credit / 20% off / Free item / Extended trial / Type your answer") and urgency framing ("Live countdown / Expires today / Expires this week / Evergreen / Type your answer"). Default brand to neutral, copy to AI-written. If personalization would help, surface it in the summary as an optional add (e.g. "Add `{{first_name | default: 'there'}}` greeting? — y/n"); do not include Liquid in the HTML unless the user opts in. Generate the HTML wiring the Claim button to `OneSignalIamApi.sendOutcome(e, "claimed_bonus")` with a unique label and dismiss control.

## Anti-Patterns

Avoid:

- Generating HTML before the use case and primary CTA are settled.
- Skipping to generation when an obvious quality-lift question would specify a vague value prop, offer, urgency, or audience.
- Asking standing questions about brand source, copy, goal, or layout — those have safe defaults and belong in the summary, not in a question.
- Asking sequential questions when one combined question would do.
- Asking any clarifying question when the prompt, URL fetch, or pasted HTML already implies the answer.
- Presenting quality-lifting questions as open text fields instead of closed sets of 3–5 plausible options plus `Type your answer`.
- Asking more than three clarifying questions across the entire flow.
- Using `<a href>` for primary actions or `window.open()` for navigation.
- Calling `openUrl()` without first calling `trackClick(e)` in custom handlers.
- Reusing the same `data-onesignal-unique-label` across elements.
- Placing Liquid variables inside `<script>` tags.
- Omitting `default:` filters on Liquid variables.
- Adding Liquid personalization (e.g. `{{first_name}}`) without explicit user request or approval. Personalization is opt-in.
- Namespacing `data-onesignal-unique-label` values verbosely (e.g. `myapp-onboarding-step-1-next`) when a short value like `next-1` is unique within the IAM and easier to read.
- Wrapping init in `document.addEventListener('DOMContentLoaded', ...)` without a `readyState` check. The IAM may inject HTML after the host page is already loaded, making the listener silently never fire.
- Relying on system dark mode without explicit color overrides.
- Returning prose alongside the HTML unless the user asked for an explanation.
- Inventing OneSignal methods that are not in the API list above.
- Producing "anything" HTML that is not anchored to a use case and CTA.

## Test Prompts

Suggested prompts to validate the skill behaves correctly:

1. "Create an in-app message." — should ask one combined blocking question covering use case + primary action. Default brand and copy.
2. "Create a soft push prompt for my fitness app. Brand: https://example.com." — should WebFetch first, then ask one combined quality-lifting question covering value-prop specifics and secondary action style. Do not ask about brand, copy, layout, or goal.
3. "Make this in-app match our brand" (with pasted HTML) — should refine in place, not rewrite. May ask one focused question about dark-mode accent or brand specifics. Re-run the self-check.
4. "Build a promo modal that records 'claimed_bonus' when the user taps Claim." — should ask one combined quality-lifting question covering offer specifics and urgency framing. Default brand and copy.
5. "Make a soft push prompt for our cooking app. Brand is warm orange #FF6B35, friendly tone, headline 'Get recipe reminders'." — should NOT skip to generation. Should ask one combined question lifting the reminder specifics and secondary action style.
