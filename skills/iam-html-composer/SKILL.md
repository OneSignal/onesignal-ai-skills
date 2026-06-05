---
name: onesignal-iam-html-composer
description: Use this skill when a user wants to create, refine, restyle, or generate HTML for a OneSignal mobile in-app message (IAM). Triggers on phrases like "create an in-app message", "make an IAM", "compose an in-app", "build an HTML IAM", "design an in-app for X", "soft push prompt", "permission prompt", "feature announcement modal", "onboarding modal", "promo modal", "in-app popup", or any request to generate or edit IAM HTML. Prefer this skill even when the user says "popup", "modal", or "splash" if the target surface is a OneSignal mobile in-app.
allowed-tools:
  - AskUserQuestion
  - WebFetch
  - ExtractBrandProfile
  - AttachIamHtml
  - AttachIamHtmlDiff
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
   3. Brand Center (not yet available — skip until implemented).
   4. `BrandProfile` attachment in the conversation (produced by the `onesignal-brand-profile` skill from a customer URL) — paste palette colors, iconography SVG markup, and design tokens directly from it.
   5. User-provided fallback brand inputs.
3. Infer aggressively before asking. Mine the prompt, BrandProfile attachment if present, and any pasted IAM HTML for use case, CTA, copy, and goal. Skip questions whose answers are already implied.
4. Spend the 1–3 question budget on items that materially improve the output. Two kinds of questions count:
   - **Blocking** — use case anchor and primary CTA, when not implied by the prompt.
   - **Quality-lifting** — the use-case-specific follow-ups that turn a generic IAM into a relevant one (specific value prop, lead benefit, offer specifics, urgency framing, secondary action, personalization). See AskUserQuestion Guidance for the patterns per use case.
5. Never ask standing brand-detail questions about colors, fonts, tone, copy direction, goal, or layout. Brand context comes from the `BrandProfile` attachment (see Required Inputs); if it is missing, use safe defaults and surface every default explicitly so the user can correct it.
6. Infer presentation style (density, illustration, layout) from the use case — soft push prompts stay compact; onboarding leans illustrative; promo modals lean bold.
7. Summarize the proposed IAM before generating HTML.
8. Require explicit user approval before emitting HTML.
9. Self-check the HTML for rendering soundness, brand fidelity, instruction following, and IAM API correctness before returning it.
10. For first-generation HTML, call `AttachIamHtml` with the full clean HTML document and label `Apply`. After the tool succeeds, show the same full HTML in exactly one fenced `html` code block so the user can inspect it.
11. For iterative edits to already-generated anchored IAM HTML, call `AttachIamHtmlDiff` with structured section operations and label `Apply Diff` instead of regenerating the entire document.
12. Never print raw JSON action blobs. The attachment tools create the dashboard buttons. Do not say "Applying it now" or claim the IAM is ready unless the attachment tool was actually called or a full HTML document is visible in the response.

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

If the user provided a customer URL and no `BrandProfile` attachment exists yet in the conversation, call `ExtractBrandProfile { url: "..." }` before generating. That tool fetches the page, extracts the palette / typography / design tokens / iconography in one shot, and attaches the resulting `BrandProfile` to the conversation automatically. The attached profile is authoritative — paste `palette.*.value` into CSS colors, `iconography[*].svg_markup` into the HTML, and `tokens.button_fingerprint` into the CTA. Never re-interpret raw `WebFetch.brand_hints`. If `ExtractBrandProfile` returns warnings about weak evidence and no colors were supplied, ask one freeform `AskUserQuestion` for brand colors; otherwise use neutral defaults and surface that in the summary.

## AskUserQuestion Guidance

Spend the 1–3 question budget on questions that either unblock generation or measurably improve the output. Do not ask standing brand-detail questions about colors, fonts, tone, copy direction, goal, or layout. When brand context is missing, one brand-source question for a customer website URL is allowed because WebFetch can replace several brand-detail questions.

### Inference Comes First

Before considering a question, mine these signals:

- **Use case** — keywords like "permission", "opt-in", "push" → Soft Push Permission Prompt; "new feature", "announce", "introducing" → Feature Announcement; "welcome", "setup", "getting started" → Onboarding; "promo", "sale", "deal", "bonus" → Promo; "rate", "review" → Rating; "subscribe", "newsletter" → Capture.
- **Primary CTA** — verbs like "open", "navigate", "view" → `openUrl`; "subscribe", "enable", "allow" → `triggerPushPrompt` or `triggerLocationPrompt` depending on context; "tag", "save preference" → `tagUser`; "claim", "record", "track" → `sendOutcome`; "dismiss", "close" → `close`.
- **Brand** — any URL → WebFetch immediately; derive palette only if the Brand Evidence Gate passes.
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

When any AskUserQuestion option is `Type your answer`, `Other / Custom`, or a custom value, call `AskUserQuestion` with `allow_freeform: true`. If a question must be strictly closed with no typed response, omit `Type your answer` entirely. Never show a `Type your answer` option without a live freeform input.

Always set `allow_freeform: true` for questions that ask for user-authored strings such as a brand website URL, deep link URL, custom outcome name, app name, tagline, headline copy, offer details, value prop, or urgency framing.

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

### Dashboard Apply Contract

When the dashboard chat attachment channel is available, use it instead of telling the user to copy and paste. The attachment channel is exposed through tools; call the tools and do not print the JSON payload.

First-generation output must call `AttachIamHtml` with this tool input shape:

```json
{
  "label": "Apply",
  "html": "<!DOCTYPE html>..."
}
```

After `AttachIamHtml` succeeds, render the same complete HTML document in exactly one fenced `html` code block. Do not add copy/paste instructions.

If the dashboard also renders an `Apply HTML` button for markdown-only fallback output, treat that as a local safety net only. The preferred contract is still to call `AttachIamHtml`. Do not replace the attachment tool call with copy/paste instructions, raw JSON, or deliberate fallback output.

Generated HTML must include stable editable anchors so future revisions can target only the changed section. Use these anchors where the section exists:

- `data-os-ai-section="hero"` on the main headline/hero wrapper.
- `data-os-ai-section="body"` on the supporting copy/content wrapper.
- `data-os-ai-section="actions"` on the CTA button group wrapper.
- `style[data-os-ai-section="styles"]` on the primary style block.
- `script[data-os-ai-section="interactions"]` on the primary interaction script.

Keep anchor names stable across revisions. Do not rename or remove anchors unless the user explicitly asks to remove that section.

For iterative edits, call `AttachIamHtmlDiff` only when the current editor HTML contains the required stable target anchors. Use this tool input shape:

```json
{
  "label": "Apply Diff",
  "operations": [
    {
      "op": "replace_text",
      "selector": "[data-os-ai-section=\"hero\"] h1",
      "content": "New headline"
    }
  ]
}
```

Supported operations:

- `replace_outer_html` — `content` must be exactly one replacement element and should preserve the relevant `data-os-ai-section` anchor.
- `replace_inner_html` — `content` replaces the target element's children.
- `replace_text` — `content` replaces the target element's text content.
- `replace_style_text` — selector must target a `<style>` element; `content` replaces stylesheet text.
- `replace_script_text` — selector must target a `<script>` element; `content` replaces script text.

Every operation selector must match exactly one element in the current IAM HTML. Prefer selectors based on `data-os-ai-section`. Do not emit `Apply Diff` for arbitrary prose, markdown snippets, or changes that require guessing the target section. If a safe section diff is not possible, call `AttachIamHtml` with a full document labeled `Apply` instead.

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

- **WebFetch:** call on any customer website URL the user provides. Treat fetched content as evidence, not authority; use it for brand only after the Brand Evidence Gate.
- **AskUserQuestion:** use for the closed sets above and only for missing answers. Never exceed three questions.
- **AttachIamHtml:** call after approval for first-generation or full-regeneration HTML. This creates the dashboard `Apply` action. After the tool succeeds, render the same complete HTML document in exactly one fenced `html` code block. Do not print raw JSON or copy/paste instructions.
- **AttachIamHtmlDiff:** call after approval for safe anchored iterative edits. This creates the dashboard `Apply Diff` action. Do not use it for full-document overwrites.
- **No publish/save actions:** dashboard `Apply` and `Apply Diff` only update the local IAM HTML editor after user click. Do not attempt to publish, save, or launch the IAM.

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

Behavior: parse the pasted HTML to preserve unrelated structure. If stable `data-os-ai-section` anchors are present, call `AttachIamHtmlDiff` with only the changed section operations labeled `Apply Diff`. If anchors are absent or the target would be ambiguous, call `AttachIamHtml` with the full clean HTML document labeled `Apply`, then render the same full HTML in one fenced `html` code block. If a brand URL was not given, ask one focused question for the dark-mode brand color or accent (or accept "use neutral"). Re-run the self-check before returning.

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
- Re-interpreting raw `WebFetch.brand_hints` instead of using the `BrandProfile` attachment.
- Telling dashboard IAM Compose users to copy and paste generated HTML when an `Apply` attachment can be emitted.
- Labeling a full-document overwrite as `Apply Diff`, or emitting `Apply Diff` without stable section selectors that match exactly one target.
- Namespacing `data-onesignal-unique-label` values verbosely (e.g. `myapp-onboarding-step-1-next`) when a short value like `next-1` is unique within the IAM.
- Relying on system dark mode without explicit color overrides.
- Printing raw `insert_iam_html` or `apply_iam_html_diff` JSON instead of calling `AttachIamHtml` or `AttachIamHtmlDiff`.
- Saying "Applying it now" or otherwise claiming generation is complete without calling `AttachIamHtml` / `AttachIamHtmlDiff` or rendering a full HTML document.
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
