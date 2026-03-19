---
name: app-store-submission-auditor
description: >
  Scans an iOS app project for App Store rejection risks. Reads source code directly —
  no back-and-forth. Auto-detects vibe coder vs developer and adapts language. Detects
  mid-build apps and asks before switching modes. Outputs a risk register, detailed findings
  with dynamic copy-paste fixes, reviewer experience checklist, draft App Store Connect
  reviewer notes, and a post-scan manual checklist.

  TRIGGER THIS SKILL for any of these — even if not explicitly asked:
  "audit my app" / "is my app ready" / "about to submit" / "submitting soon" /
  "about to launch" / "ready to ship" / "App Store review" / "keep getting rejected" /
  "got rejected" / "rejection" / "App Store Connect" / "TestFlight" / "submit for review" /
  "pre-submission" / "app review" / "first app" / "never submitted before" /
  "why did Apple reject" / "what do I need to fix" / "is this ready to launch".
  Also trigger proactively when working on an iOS app and the user seems close to shipping.

  Flutter/RN stack: load references/flutter-patterns.md.
  First-time submitter: load references/first-time-dev.md.
---

# App Store Submission Auditor

Scans a project folder for App Store rejection risks. No back-and-forth — read the code,
find the problems, report them. Adapts to the user's level and build state.

---

## First-time developer

If user mentions "first app", "first time submitting", "never submitted before", or
"new developer" → load `references/first-time-dev.md` and run that checklist first,
then continue with the full audit below.

---

## Config — check before scanning

Look for `config.json` in the skill directory. If it exists, read it for saved preferences
(stack, app type, subscription status, EU distribution, previous audit results).

If it does NOT exist, create it after the first audit with what was detected:

```json
{
  "mode": "vibe_coder",
  "stack": "flutter",
  "has_subscriptions": true,
  "has_social_features": true,
  "has_location": true,
  "eu_distribution": false,
  "first_time_submitter": false,
  "last_audit": "2026-03-19",
  "issues_fixed": []
}
```

On subsequent runs: load config, skip re-detection for anything already known, and note
"Last audit: [date] — you fixed [N] issues since then." if issues_fixed is populated.

---

## Gotchas — common false positives to avoid

These are the things Claude most often gets wrong when running this audit.
Check these before flagging any issue.

**1. Stripe for physical goods is fine**
Only flag Stripe/PayPal as a violation if it's being used for digital goods or features.
Stripe for physical item swaps, marketplace escrow, or real-world services = NOT a violation.
Only flag if credits/payments unlock in-app digital features.

**2. WebView for help/legal is fine**
Only flag WebView as a web wrapper if it's the PRIMARY UI. A WebView loading a terms of
service page, help article, or external link inside an otherwise native app is fine and
expected. Only flag if fewer than ~5 native screens exist and WebView loads the main content.

**3. NSUserTrackingUsageDescription without tracking SDKs**
If the key exists but no known tracking SDKs are found, flag as "unnecessary ATT prompt"
(P1), not as a missing string. Don't flag it as missing — it's present. Flag it as
potentially causing rejection for prompting unnecessarily.

**4. Firebase Analytics ≠ automatic rejection**
Firebase Analytics collecting data is not a rejection by itself. It only becomes a problem
if it's undisclosed in the privacy policy and App Privacy label. Flag as "needs disclosure"
not as a hard rejection.

**5. Staging URL might be intentional**
Flag staging/dev URLs as ⚠️ with "confirm this isn't in your production build" — not as a
guaranteed rejection. The developer may have a staging build and a production build config.
Don't assume it's in the production binary without asking.

**6. Location in startup ≠ always wrong**
If the app's CORE purpose is location-based (e.g. a map app, a nearby listings app) and
location is requested near startup, that's acceptable. Only flag if non-location features
could work without it and location is still requested upfront.

**7. No block feature ≠ reject for apps without chat**
Only flag missing block user feature if the app has actual user-to-user interaction (chat,
messaging, social profiles). A marketplace with listings but no direct user messaging does
not need a block feature.

---

## Step 1 — Three detections before scanning

### A) Detect user mode

**Vibe coder signals:** mentions Claude Code / Cursor / Bolt / "vibe coding" / "built with AI",
casual language, no file/framework specifics, non-developer questions.

**Technical signals:** names files or frameworks unprompted, pastes code, uses dev terminology,
references guideline numbers.

**Default if unclear:** vibe coder mode.

### B) Detect stack (read top-level folder)
- `pubspec.yaml` → Flutter → load `references/flutter-patterns.md`
- `package.json` + `ios/` → React Native → load `references/flutter-patterns.md`
- `*.xcodeproj` only → Native Swift

### C) Detect build state (read during initial folder scan)

**Mid-build signals** — flag if 2+ are true:
- More than 3 `TODO`, `FIXME`, `coming soon`, `placeholder` in user-visible strings
- Core flows missing entirely (no auth screens, no main feature, no navigation)
- Stub/mock function names (`stubAuth()`, `mockPayment()`, `fakeLocation()`)
- Fewer than ~8 meaningful screens or widgets

If mid-build signals detected → **pause and ask before continuing:**

> "Your app looks like it might still be in progress — I'm seeing [specific signals found].
> Which would be more useful right now?
>
> **A) Submission audit** — I'll flag everything Apple would reject
> **B) Build checklist** — I'll tell you what to finish before worrying about submission"

- If **A**: continue with full audit, note mid-build state in executive summary
- If **B**: run Build Checklist mode (see end of skill)

---

## Step 2 — Announce mode

**Vibe coder:**
> "Running in **plain English mode** — no jargon, copy-paste fixes.
> *(Developer? Just say so.)*"

**Technical:**
> "Running in **developer mode** — technical language, exact code diffs.
> *(Want plain English? Just say so.)*"

Switch immediately if corrected.

---

## Step 3 — Scan these files

Read without asking. Note missing files and continue.

**Flutter:** `ios/Runner/Info.plist` · `pubspec.yaml` · `lib/main.dart` ·
`ios/Runner/PrivacyInfo.xcprivacy` · `*.entitlements` ·
files matching: `*auth*` `*account*` `*delete*` `*user*` `*block*` `*report*`
`*location*` `*geo*` `*permission*` `*payment*` `*purchase*` `*subscription*`
`*iap*` `*credits*` `*paywall*` `*api*` `*network*` `*config*` `*env*`
`*constants*` `*privacy*` `*policy*` `*eula*` `*terms*` `*legal*`

**Native Swift:** `*/Info.plist` · `AppDelegate.swift` · `*.entitlements` ·
`PrivacyInfo.xcprivacy` · `*Account*` `*Auth*` `*Delete*` `*Block*` `*Report*`
`*Location*` `*Permission*` `*Payment*` `*Purchase*` `*IAP*` `*Subscription*`
`*Network*` `*API*` `*Config*` `*Constants*`

---

## Step 4 — What to look for

Collect issues silently. Output everything at once in Step 5.

**Severity:**
- 🚨 P0 — Apple will reject for this
- ⚠️ P1 — Actively checked, likely flagged
- 💡 P2 — Won't necessarily reject, worth fixing

---

### Permission strings
Read every `NS*UsageDescription` in Info.plist.

Flag:
- Empty value → 🚨
- Generic value ("needed for app", "required for functionality", "to improve your experience") → 🚨
- Feature exists in code but key is missing → 🚨
- Key exists but no matching feature in code → ⚠️ over-requesting
- `NSLocationAlwaysUsageDescription` without clear background need → ⚠️
- Tracking SDK present but `NSUserTrackingUsageDescription` missing → 🚨
- `PrivacyInfo.xcprivacy` missing or incomplete when tracking SDKs present → ⚠️

---

### Location permission order
Flag if location is accessed before `requestPermission()` is called, or at app startup
when not all features need it. Check Flutter (`Geolocator`) and native (`CLLocationManager`).

---

### Account deletion
Flag: `mailto:` redirect → 🚨 · web URL redirect → 🚨 · no confirmation dialog → 🚨 ·
only one auth provider deleted → ⚠️ · no deletion flow found anywhere → 🚨

---

### Block user
Flag: no block function exists → 🚨 · block and report are same function → ⚠️ ·
block only hides in UI, not enforced at data layer → ⚠️ · block missing from profile
screen OR chat screen → ⚠️

---

### Sign in with Apple
Flag: third-party social login exists (Google, Facebook, Twitter) but no Sign in with
Apple → 🚨 (required when any third-party login is offered)

---

### Payments & IAP
Flag: Stripe/PayPal/custom for digital goods → 🚨 · credits unlocking digital features
outside IAP → 🚨 · no restore purchases call found → ⚠️ · Apple EULA URL not in
codebase → ⚠️ · external purchase links for digital features → 🚨 ·
subscription terms not shown before purchase → ⚠️

---

### Network config
Flag: hardcoded IPv4 literals → 🚨 · `http://` in production config → 🚨 ·
staging URLs in production build → ⚠️ · `NSAllowsArbitraryLoads: true` → ⚠️

---

### Encryption compliance
Read `Info.plist` for `ITSAppUsesNonExemptEncryption`.

Flag:
- Key missing entirely → ⚠️ (Apple will ask during submission, blocks upload)
- Set to `true` with no export compliance documentation → ⚠️

**Vibe coder:** "Your app needs to declare whether it uses encryption. If you only
use standard HTTPS (which most apps do), add this to your Info.plist:
Tell Claude Code: 'Add ITSAppUsesNonExemptEncryption with value NO to Info.plist'"

**Technical:** `ITSAppUsesNonExemptEncryption` missing from Info.plist.
If app uses only HTTPS → set to `NO`. If custom encryption → set to `YES` and
upload export compliance documentation in App Store Connect.

---

### Debug artifacts
Flag: `debugShowCheckedModeBanner: true` → ⚠️ · "beta"/"demo"/"coming soon"/
"placeholder"/"TODO" in user-visible strings → ⚠️ · blank screens with no empty
state UI → ⚠️ · excessive print statements → 💡

---

### SDK inventory
Read `pubspec.yaml` / `package.json`. Flag undisclosed data-collecting SDKs:
`firebase_analytics` `facebook_*` `amplitude_flutter` `mixpanel_flutter`
`appsflyer_sdk` `adjust_sdk` → ⚠️ each (needs disclosure, not a rejection itself)

---

### Regulated content
Scan user-visible strings. Flag: medical claims ("treats", "cures", "clinically proven") → ⚠️ ·
personalized financial advice without disclaimers → ⚠️ · safety/emergency features
without disclaimers → ⚠️

---

### ATT / tracking prompt
Read `Info.plist` and `pubspec.yaml` / `package.json`.

**Flag if:**
- Tracking SDK present but `NSUserTrackingUsageDescription` missing → 🚨
- `NSUserTrackingUsageDescription` present but vague → 🚨
- `PrivacyInfo.xcprivacy` missing when tracking SDKs present → ⚠️
- ATT requested at app launch before user has experienced any value → ⚠️
- **`NSUserTrackingUsageDescription` present but NO tracking SDKs found** → ⚠️
  (unnecessary ATT prompt — Apple rejects apps that ask for tracking without actually tracking)

**Flutter:** ATT must use `app_tracking_transparency` package. Flag if tracking SDKs
exist but this package is absent from `pubspec.yaml`.

---

### Web wrapper detection
Flag: `webview_flutter` / `flutter_inappwebview` as primary UI with fewer than ~5 native
screens → 🚨 · `WKWebView` loading external URL as entire app → 🚨 · all navigation
inside WebView with no native nav → 🚨

Not flagged: WebView used for a single supplemental feature alongside native functionality.

---

### 2025/2026 requirements
Flag:
- New age rating tiers: if app has UGC, chat, or advertising — old 12+ may now need 13+/16+
  under January 2026 updated questionnaire → ⚠️ (note: verify in App Store Connect)
- External AI consent: if app sends personal user data to an external AI service and no
  consent modal found in code → ⚠️
- Swift 6 (native only): ATT and StoreKit calls not marked `@MainActor` → 💡

---

## Step 5 — Output

Output all sections at once after the full scan.

---

### Language rules by mode

**Vibe coder:** Plain English. Explain what the thing is before saying it's wrong.
Claude Code prompts specific to actual file and function found — not generic templates.

Format: `Tell Claude Code: "[specific instruction referencing actual file/function found]"`

**Technical:** Dev terminology fine. Exact code diffs referencing actual file and line.
Guideline numbers where known.

---

### Section 1 — Executive summary

```
[VIBE CODER]
YOUR APP AUDIT
══════════════
What your app does: [inferred — 1 sentence]
Issues found: [X] — [Y] will get you rejected, [Z] are risky

3 biggest things to fix:
• [top issue in plain English]
• [second]
• [third]

3 quick wins:
• [easiest fix]
• [second]
• [third]

[TECHNICAL]
AUDIT SUMMARY
═════════════
Stack: [stack] · Files scanned: [N] · Issues: [X] ([Y] P0, [Z] P1)
App: [inferred purpose]
Top risks: [top 3] · Fast wins: [top 3]
```

---

### Section 2 — Risk register

Both modes get this table. Adapt language in cells to detected mode.
Omit rows with no issues. Use `Assumption — verify manually` when no code evidence.

```
| Priority | Area | Finding | Evidence | Fix | Effort | Confidence |
|----------|------|---------|----------|-----|--------|------------|
| P0 🚨 | [area] | [finding] | [file:line or function] | [fix] | S/M/L | High/Med/Low |
```

Effort: S = <1hr · M = half day · L = multi-day

---

### Section 3 — Detailed findings

Group by area. Skip clean areas entirely.

Each finding: what was found (specific file/function) · why Apple rejects it · exact fix
Vibe: plain English + specific Claude Code prompt
Technical: code diff

---

### Section 4 — Reviewer experience

Simulate what Apple's reviewer does. Mark each:
✅ likely passes · ⚠️ risky · ❌ will fail · ⬜ can't determine from code

```
Install & launch
  [?] App opens without crashing

First run
  [?] Clear what app does on first screen
  [?] Permissions explained before requested
  [?] No dead ends or blank screens

Core features
  [?] Reviewer reaches main features without special setup
  [?] Test account path exists (if login required)

Purchases (if applicable)
  [?] Paywall shows price, billing, terms
  [?] Restore purchases visible
  [?] No external payment links for digital features

Account (if applicable)
  [?] Account creation works
  [?] Account deletion findable and works in-app

Links & legal
  [?] Support URL loads · Privacy policy loads · EULA accessible

Edge cases
  [?] No internet → graceful error (not crash)
  [?] Empty states have UI
```

---

### Section 5 — Draft reviewer notes

Generate ready-to-paste text for App Store Connect → Notes for App Review.
Infer every section from actual code. Only use `[fill in]` for things Claude can't know.

```
--- NOTES FOR APP REVIEW ---

ABOUT THIS APP
[Inferred from code]

TEST ACCOUNT
Email:    [fill in]
Password: [fill in]

HOW TO REACH KEY FEATURES
[Steps inferred from navigation structure]

PERMISSIONS
[Each NS*UsageDescription found — feature it's used for]

ACCOUNT DELETION
[Path inferred from code, or omit if not found]

[Any unusual flows, credits systems, or gated features]
--- END NOTES ---
```

---

### Section 6 — Manual checklist + 4 post-scan questions

After the automated findings, end with this section.

```
[VIBE CODER]
THINGS TO CHECK YOURSELF
════════════════════════
In App Store Connect:
  □ Fill in the App Privacy section
  □ Age rating — updated tiers as of Jan 2026: check if your app
    needs 13+ or 16+ (not just 12+) if it has chat, UGC, or ads
  □ All subscription plans in ONE group
  □ Add Apple's EULA link to your App Store description:
    apple.com/legal/internet-services/itunes/dev/stdeula/

Your listing:
  □ Description matches current build
  □ Screenshots show the current version (not Figma)
  □ Support link loads · Privacy policy link loads

Before submitting:
  □ Test on a real iPhone, final build — not simulator
  □ Open on iPad — shouldn't crash
  □ Paste test account into Review Notes

Good to know:
  □ Phased release — after approval you can roll out to
    1% → 2% → 5% → 10% → 50% → 100% instead of everyone at once.
    Great for catching issues before full launch.
  □ Expedited review — if it's a critical bug fix or urgent,
    you can ask Apple to fast-track it in App Store Connect
    under App Review → Contact Us → Expedite Request.

[TECHNICAL]
MANUAL CHECKS — NOT IN CODE
════════════════════════════
App Store Connect
  □ App Privacy nutrition label completed
  □ Age rating updated (Jan 2026 questionnaire — new 13+/16+/18+ tiers)
  □ All subscription tiers in single subscription group
  □ Apple EULA in App Store description
  □ ITSAppUsesNonExemptEncryption set in Info.plist
  □ EU DSA trader status verified (if distributing paid app in EU)
Metadata
  □ Description matches current build · No prices in description
  □ No "Also available on Android" references
  □ Screenshots match current build (real UI, not marketing renders)
  □ Support URL resolves · Privacy policy URL live
Device testing
  □ Physical device, release build · iPad doesn't crash
  □ IPv6 network test (Apple review network is IPv6-only)
Review Notes
  □ Demo credentials added · Reviewer notes draft pasted
Useful options
  □ Phased release available after approval (1%→2%→5%→10%→50%→100%)
  □ Expedited review available for critical fixes (App Review → Contact Us)
```

---

**After the manual checklist, ask these 4 questions:**

```
[VIBE CODER]
One last thing — a few things I can't check from your code:

  1. Have you tested the app on a real iPhone recently?
     (Not just the simulator)

  2. Do your App Store screenshots show the current
     version of your app?

  3. Have you added a test account to your Review Notes?
     (Skip this if your app doesn't require login)

  4. Is this your first time submitting to the App Store?
     (Say yes and I'll walk you through a setup checklist)

[TECHNICAL]
Four things I can't verify from code:

  1. Tested on physical device with release build?
  2. Screenshots in ASC match current build?
  3. Demo credentials in App Review Notes? (skip if no login)
  4. First submission? (yes → I'll load the first-time setup checklist)
```

If user answers yes to question 4 → load `references/first-time-dev.md`

---

### Closing line

**Vibe coder:**
> "[X] things to fix. Start with the 🚨 ones — guaranteed rejections.
> Answer the 4 questions above and fill in your test account in the reviewer
> notes draft, then you're ready to submit.
> *(Say "fix [anything]" and I'll write exactly what to tell Claude Code.)*"

**Technical:**
> "[X] issues. P0s first. Answer the 4 questions, fill reviewer notes credentials.
> *(Say "fix [issue]" for corrected code.)*"

**If zero code issues:**
> "No code-level issues found. Answer the 4 questions above and run through
> the manual checklist — then you're good to submit."

---

## Re-audit

If user says "I fixed [X]" or shares updated code:
- Re-scan only that area
- Confirm ✅ or explain specifically what's still wrong
- Update that item in the risk register only
- Note: metadata rejection = fix in App Store Connect, no new build needed
       binary rejection = fix code, create new archive, upload new build

---

## Build checklist mode

Run if user chose option B after mid-build detection.

```
YOUR BUILD CHECKLIST
════════════════════
[What the app does so far — inferred from code]

FINISH THESE FIRST
[Unfinished things found — stubs, TODOs, missing flows.
Specific file references.]

ALREADY DONE ✅
[What's clearly implemented and working]

WHEN YOU'RE DONE BUILDING
Come back and say "audit my app" — I'll check for
App Store submission then.
```

Tone: encouraging. No rejection warnings. Focus on what to build next.
