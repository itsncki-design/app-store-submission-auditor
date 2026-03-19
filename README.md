# App Store Submission Auditor

A Claude skill that scans your iOS app for App Store rejection risks before you submit.
Built for vibe coders and developers alike.

## What makes this different

Most App Store checklists are written for experienced iOS developers. This one auto-detects
whether you're a vibe coder or a developer and changes how it talks to you — plain English
with copy-paste Claude Code prompts, or technical diffs with guideline references. Same
audit, two voices.

It also detects if your app is mid-build and asks if you want a build checklist instead
of a submission audit — no point reviewing for rejection risks if you're not done yet.

## What it does

- Scans your project folder directly — no questions, no back-and-forth
- Works for Flutter, React Native, and native Swift apps
- Auto-detects vibe coder vs developer and adapts language accordingly
- Detects mid-build apps and offers a build checklist instead
- Outputs 6 sections:
  1. Executive summary
  2. Risk register table (Priority / Finding / Evidence / Fix / Effort)
  3. Detailed findings with exact fixes
  4. Reviewer experience checklist (simulates what Apple's reviewer does)
  5. Draft App Store Connect reviewer notes — ready to paste
  6. Manual checklist for things outside the codebase

## What it checks

- Permission strings — catches vague or missing `NS*UsageDescription` keys
- Location permission order — flags location access before permission is requested
- Account deletion — catches email flows, missing confirmation, partial deletion
- Block user — checks it exists, is separate from report, enforced at data layer
- Sign in with Apple — required if any third-party social login exists
- In-app purchases — catches Stripe for digital goods, missing restore purchases
- Network config — hardcoded IPs, HTTP URLs, staging URLs in production
- Debug artifacts — debug banners, placeholder text, blank screens
- Third-party SDKs — flags undisclosed data-collecting SDKs
- Regulated content — medical, financial, safety claims

## How to install

1. Download `app-store-submission-auditor.skill` from the latest release below
2. Go to Claude.ai → Settings → Skills
3. Click **Install skill** and upload the file

## How to use

In any Claude conversation with the skill installed, say:

> "Audit my app for App Store submission"

Then point Claude at your project folder. It reads the code and reports back —
no questions unless it detects your app is mid-build.

**To re-check a fix:**
> "I fixed my account deletion, re-check it"

**To switch modes:**
> "Switch to developer mode" or "Switch to plain English mode"

## Skill structure

```
app-store-submission-auditor/
├── SKILL.md                        — main skill instructions
└── references/
    └── flutter-patterns.md         — Flutter/RN specific code patterns
```

## Built by

[@itsncki-design](https://github.com/itsncki-design) — if this helped you ship,
give it a ⭐ and share it with someone else who's building with AI.
