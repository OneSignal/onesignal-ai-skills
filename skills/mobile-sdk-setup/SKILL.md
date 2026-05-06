---
name: onesignal-mobile-sdk-setup
description: Use this skill when a user wants to set up, integrate, troubleshoot, or choose a OneSignal mobile SDK for Android, iOS, Huawei, React Native, Expo, Flutter, Unity, .NET MAUI, Cordova, Ionic, Capacitor, FlutterFlow, or Median.co. Use for prompts about mobile push setup, APNs, FCM, HMS, SDK installation, SDK initialization, notification permission prompting, App ID setup, push not delivering after SDK install, or whether SDKs are required.
---

# OneSignal Mobile SDK Setup Skill

## Purpose

Help OneSignal AI guide users through mobile SDK setup for OneSignal push notifications, in-app messages, and Live Activities.

This skill helps the agent:

- choose the correct SDK/setup path,
- collect missing platform context,
- explain prerequisite credential setup,
- guide users through SDK install and initialization,
- distinguish what requires the SDK vs what can be done through REST APIs,
- troubleshoot common setup blockers.

## When to Use

Use this skill when the user asks for:

- "Set up the mobile SDK"
- "Install OneSignal in my iOS app"
- "Install OneSignal in my Android app"
- "Set up React Native"
- "Set up Flutter"
- "Set up Unity"
- "Set up Cordova/Ionic/Capacitor"
- "Set up Huawei"
- "Set up push notifications"
- "Why aren't push notifications delivering after SDK setup?"
- "Do I need the SDK?"
- "Where do I find my OneSignal App ID?"

If the user says "mobile setup", "push setup", or "SDK setup", treat it as likely mobile SDK setup unless they clearly mean web push or server-side REST API only.

## Core Behavior

When this skill is active:

1. Determine the user's platform/framework.
2. Determine whether they have configured platform credentials in OneSignal.
3. Determine whether they are installing the SDK, initializing it, testing push, or troubleshooting.
4. Ask only for missing information.
5. Use AskQuestion when the user should choose from supported platform options.
6. Provide platform-specific guidance instead of generic SDK advice.
7. Clearly separate OneSignal dashboard setup from codebase integration.
8. Do not invent SDK APIs, unsupported platforms, or unsupported setup paths.

## Required Inputs

A useful SDK setup answer usually needs:

- Platform/framework.
- Whether this is a new or existing OneSignal app.
- Whether platform credentials are already configured.
- OneSignal App ID, if initializing code.
- Whether the user wants push, in-app messages, Live Activities, or all available SDK features.
- Whether the user is installing, testing, or troubleshooting.

Do not ask for all inputs upfront. Start with the platform/framework if it is missing.

## AskQuestion Guidance

Use AskQuestion instead of freeform text when the user needs to choose from supported setup paths.

### Platform Selection

If the user asks to set up the mobile SDK without specifying the platform, use AskQuestion.

Question:

```text
Which mobile platform or framework are you setting up?
```

Options:

- Android native
- iOS native
- React Native / Expo
- Flutter
- Unity
- Type your answer

### Setup Stage

If the platform is known but the setup stage is unclear, use AskQuestion.

Question:

```text
Where are you in the setup process?
```

Options:

- I need to configure push credentials in OneSignal
- I need to install the SDK in my app
- I installed the SDK and need to initialize it
- I am testing push delivery
- Type your answer

### Feature Goal

If the user asks broadly about SDK setup but the desired feature is unclear, use AskQuestion.

Question:

```text
What are you setting up the SDK for?
```

Options:

- Push notifications
- In-app messages
- Live Activities
- All available mobile messaging features
- Type your answer

## Product Rules

- Mobile SDK setup has two main parts: configure platform credentials in OneSignal, then integrate and initialize the SDK in the app.
- Android push requires Firebase Cloud Messaging (FCM) credentials in OneSignal before push can deliver.
- iOS push requires APNs credentials in OneSignal before push can deliver. p8 auth key is recommended; p12 certificate is also supported.
- Huawei devices require HMS setup when using Huawei push services.
- OneSignal App ID is a 36-character UUID found in Dashboard > Settings > Keys & IDs.
- A single OneSignal app can support multiple platforms. Users do not need separate OneSignal apps for iOS and Android.
- SDKs are strongly recommended for push because they handle push tokens, subscription status, permission prompts, notification display, and platform payload differences.
- Some REST API operations are possible without SDKs, such as creating users, subscriptions, and messages.
- In-app messages and Live Activities require SDK support; they cannot be delivered via API alone.
- Android setup should initialize with `OneSignal.initWithContext(...)`.
- iOS setup should initialize with `OneSignal.initialize(...)`.
- React Native setup uses `react-native-onesignal` and initializes with `OneSignal.initialize(...)`.
- Flutter setup uses `onesignal_flutter` and initializes with `OneSignal.initialize(...)`.
- Assigning External ID via `OneSignal.login(...)` is recommended to unify users across devices/subscriptions.
- Permission prompting should be deliberate. For production, consider using in-app messages or a soft prompt instead of prompting immediately on first launch.

## Common Defaults and Best Practices

- If platform is missing, ask for platform first.
- If the user wants push delivery, check credential setup before debugging SDK code.
- If push works on Android but not iOS, check APNs, entitlements, provisioning, Notification Service Extension, and App Groups.
- If push works on iOS but rich images/confirmed delivery do not, check Notification Service Extension and App Group setup.
- If push does not deliver on Android, check FCM credentials and whether the test device has Google Play Services.
- If the user is using React Native, clarify whether it is bare React Native or Expo.
- If the user is using Flutter on iOS, note Swift Package Manager support requires opt-in in some cases.
- Enable verbose logging for debugging, but tell users to remove or reduce verbose logs in production.

## Tool Guidance

Use available tools to:

- Search OneSignal docs for the platform-specific SDK setup page.
- Check app/platform setup state if tools are available.
- Retrieve App ID or direct users to Settings > Keys & IDs if needed.
- Inspect platform setup before recommending code-only fixes.
- Link to the exact setup guide for the user's platform.

Do not expose internal tool mechanics to the user.

## Summary and Approval

This skill is primarily instructional and diagnostic. Most setup guidance does not require approval.

Require explicit approval before taking any write action, such as:

- changing app/platform settings,
- creating or updating credentials,
- modifying project files,
- applying generated code directly.

Before any write action, summarize:

- what will be changed,
- where it will be changed,
- why it is needed,
- what the user should test afterward.

## Examples

### Generic Mobile SDK Setup

User:

```text
Help me set up the OneSignal mobile SDK.
```

Behavior:

- Use AskQuestion to ask which platform/framework.
- After platform is selected, explain the two-part setup: platform credentials in OneSignal, then SDK integration in the app.

### Android Setup

User:

```text
Install OneSignal in my Android app.
```

Behavior:

- Explain that FCM credentials are required in OneSignal for push delivery.
- Guide the user to add the Android SDK dependency.
- Tell them to initialize OneSignal from the Application class.
- Mention `OneSignal.initWithContext(...)`.
- Mention testing on an Android device/emulator with Google Play Services.

### iOS Setup

User:

```text
Set up OneSignal for iOS push.
```

Behavior:

- Explain that APNs credentials are required in OneSignal for push delivery.
- Recommend p8 auth key when applicable.
- Mention Push Notifications and Background Modes capabilities.
- Mention Notification Service Extension and App Group for rich notifications, confirmed delivery, and badges.
- Mention `OneSignal.initialize(...)`.

### React Native Setup

User:

```text
Set up OneSignal in React Native.
```

Behavior:

- Ask whether the app is bare React Native or Expo if unclear.
- For bare React Native, mention `react-native-onesignal`.
- Mention native Android and iOS setup requirements still apply.
- Mention initializing in `App.tsx`, `App.js`, or `index.js`.

### Troubleshooting Push Delivery

User:

```text
I installed the SDK but push notifications are not delivering.
```

Behavior:

- Ask platform if missing.
- Check/ask whether platform credentials are configured in OneSignal.
- For Android, check FCM and Google Play Services.
- For iOS, check APNs, entitlements, physical/supported simulator constraints, and Notification Service Extension where relevant.
- Ask for logs only after checking setup prerequisites.

### Text Fallback

If structured question UI cannot render, ask the same question in text:

```text
Which mobile platform or framework are you setting up?
1. Android native
2. iOS native
3. React Native / Expo
4. Flutter
5. Unity
6. Type your answer
```

## Anti-Patterns

Avoid:

- Giving generic SDK advice before asking for platform/framework.
- Debugging SDK code before checking push credential setup.
- Telling users push will deliver before APNs/FCM/HMS credentials are configured.
- Claiming in-app messages or Live Activities can be delivered by REST API alone.
- Inventing SDK methods or setup paths not in OneSignal docs.
- Asking for secrets, API keys, p8 keys, p12 certificates, or private credentials in chat.
