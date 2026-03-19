# App Store Submission Auditor — AI Skill

An AI skill that scans your iOS app for App Store rejection risks before you submit.
Works with Claude Code, Cursor, GitHub Copilot, Windsurf, and any skills-compatible tool.
Built for vibe coders and developers alike.

## What's new in v1.1

- **Smarter triggering** — fires automatically on 20+ phrases like "submitting soon",
  "got rejected", "about to launch", "TestFlight", and more
- **Config memory** — remembers your stack and app type between sessions,
  shows what you fixed since your last audit
- **False positive fixes** — 7 gotchas prevent common mistakes like flagging
  Stripe for physical goods or WebView help screens as hard rejections
- **First-time dev checklist** — detects first-time submitters and walks
  through Apple Developer Program setup, bundle IDs, certs, and tax forms
- **2025/2026 requirements** — covers new age rating tiers, external AI consent,
  and Xcode 26 SDK deadline (April 28, 2026)

## What makes this different

Most App Store checklists are written for experienced iOS developers. This one
auto-detects whether you're a vibe coder or a developer and changes how it talks
to you — plain English with copy-paste AI prompts, or technical diffs with guideline
references. Same audit, two voices.

It also detects if your app is mid-build and asks if you want a build checklist
instead of a submission audit — no point reviewing for rejection risks if you're
not done yet.

## What it does

- Scans your project folder directly — no questions, no back-and-forth
- Works for Flutter, React Native, and native Swift apps
- Auto-detects vibe coder vs developer and adapts language accordingly
- Remembers your app settings between sessions via config.json
- Detects mid-build apps and offers a build checklist instead
- Detects first-time submitters and walks through setup
- Outputs 6 sections:
  1. Executive summary
  2. Risk register table (Priority / Finding / Evidence / Fix / Effort)
  3. Detailed findings with exact fixes
  4. Reviewer experience checklist (simulates what Apple's reviewer does)
  5. Draft App Store Connect reviewer notes — ready to paste
  6. Manual checklist + 4 post-scan questions

## What it checks

- Permission strings — catches vague or missing `NS*UsageDescription` keys
- Location permission order — flags location access before permission is requested
- Account deletion — catches email flows, missing confirmation, partial deletion
- Block user — checks it exists, is separate from report, enforced at data layer
- Sign in with Apple — required if any third-party social login exists
- In-app purchases — catches Stripe for digital goods, missing restore purchases
- Network config — hardcoded IPs, HTTP URLs, staging URLs in production
- Encryption compliance — `ITSAppUsesNonExemptEncryption` declaration
- Debug artifacts — debug banners, placeholder text, blank screens
- Third-party SDKs — flags undisclosed data-collecting SDKs
- ATT / tracking prompt — catches both missing AND unnecessary prompts
- Web wrapper detection — flags apps that are just websites in a WebView
- Regulated content — medical, financial, safety claims
- 2025/2026 requirements — updated age ratings, AI consent, Xcode SDK deadline

## How to install

### Option 1 — Terminal (works everywhere)

```bash
npx skills add https://github.com/itsncki-design/app-store-submission-auditor
```

Run this in your terminal, use arrow keys + space to select your AI tool(s), hit Enter.

### Option 2 — Download and install

1. Download `app-store-submission-auditor.skill` from the latest release below
2. Install it in your AI tool's settings

For Claude / Claude.ai: Settings → Skills → Install skill → upload the file

## Compatible tools

| Tool | Install method |
|------|----------------|
| Claude Code | Terminal or .skill file |
| Cursor | Terminal |
| GitHub Copilot | Terminal |
| Windsurf | Terminal |
| Any skills-compatible tool | Terminal |

## How to use

Once installed, just say:

> "Audit my app for App Store submission"

Or start working on your iOS app — the skill triggers automatically when it detects
you're close to submitting.

**To re-check a fix:**
> "I fixed my account deletion, re-check it"

**To switch modes:**
> "Switch to developer mode" or "Switch to plain English mode"

**First time submitting:**
> "This is my first app" — it walks you through setup before the audit

## Skill structure

```
app-store-submission-auditor/
├── SKILL.md                        — main skill instructions
├── config.json                     — remembers your app settings between sessions
└── references/
    ├── flutter-patterns.md         — Flutter/RN specific code patterns
    └── first-time-dev.md           — first-time App Store submission setup
```

## Built by

[@itsncki-design](https://github.com/itsncki-design) — if this helped you ship,
give it a ⭐ and share it with someone building with AI.
