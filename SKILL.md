---
name: app-store-submission-auditor
description: >
  Scans an iOS app's project folder for App Store rejection risks. Reads source code directly —
  no back-and-forth. Auto-detects whether the user is a vibe coder or technical developer and
  adapts language accordingly. Detects mid-build apps and asks before switching modes. Outputs
  a risk register, detailed findings with dynamic copy-paste fixes, reviewer experience
  checklist, draft App Store Connect reviewer notes, and a manual checklist. Use this skill
  whenever a user wants to check if their iOS app is ready to submit, says "audit my app",
  "is my app ready for the App Store", wants a pre-submission scan, mentions App Store review,
  wants to know what to fix before submitting, or is preparing to launch an iOS app. Also
  trigger proactively when a user has been building an iOS app and says they're getting close
  to done or ready to ship.
---

# App Store Submission Auditor

Scans a project folder for App Store rejection risks. No back-and-forth — read the code,
find the problems, report them. Adapts to the user's level and build state.

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

> "Your app looks like it might still be in progress — I'm seeing [specific signals found,
> e.g. 'placeholder screens and stub functions']. Which would be more useful right now?
>
> **A) Submission audit** — I'll flag everything Apple would reject (assumes app is complete)
> **B) Build checklist** — I'll tell you what to finish before worrying about submission
>
> Which one?"

- If **A**: continue with full audit below, note mid-build state in executive summary
- If **B**: run Build Checklist mode (see end of skill)

---

## Step 2 — Announce mode

Say this before scanning anything:

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

For each area: collect issues silently. Output everything at once in Step 5.

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
- Tracking SDKs present but `PrivacyInfo.xcprivacy` missing or incomplete → ⚠️

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
codebase → ⚠️ · external purchase links for digital features → 🚨

---

### Network config
Flag: hardcoded IPv4 literals → 🚨 · `http://` in production config → 🚨 ·
staging URLs in production build → ⚠️ · `NSAllowsArbitraryLoads: true` in
Info.plist → ⚠️

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
Scan user-visible strings. Flag medical claims ("treats", "cures", "clinically proven") → ⚠️ ·
personalized financial advice → ⚠️ · safety/emergency features without disclaimers → ⚠️

---

## Step 5 — Output

Output all sections at once after the full scan.

---

### Language rules by mode

Write every finding, fix, and recommendation in the correct voice:

**Vibe coder:** Plain English. No jargon. Explain what the thing is before saying it's wrong.
Claude Code prompts must be specific to the actual file and actual code found — not generic.

Format: `Tell Claude Code: "[specific instruction based on actual file/function found]"`

Example — if `mailto:` deletion found in `lib/screens/settings_screen.dart`:
> Tell Claude Code: "In settings_screen.dart, remove the mailto deletion button and replace
> it with an in-app confirmation dialog. When the user confirms, call deleteUser() on both
> Firebase Auth and Cognito, then navigate to the onboarding screen."

Example — if vague location string found in `ios/Runner/Info.plist`:
> Tell Claude Code: "In ios/Runner/Info.plist, update NSLocationWhenInUseUsageDescription
> to say: 'Your location shows listings near you and displays your city on your profile.
> Never shared without your consent.'"

**Technical:** Dev terminology fine. Show exact code diffs referencing the actual file and
line/function found. Guideline numbers where known.

---

### Section 1 — Executive summary

```
[VIBE CODER]
YOUR APP AUDIT
══════════════
What your app does: [inferred from code — 1 sentence]
Issues found: [X] — [Y] will get you rejected, [Z] are risky

The 3 biggest things to fix:
• [top issue in plain English]
• [second issue]
• [third issue]

3 quick wins (easy fixes):
• [easiest fix]
• [second easiest]
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

Use this table for both modes. Adapt language inside cells to detected mode.
Omit rows with no issues. If no evidence found, write `Assumption — verify manually` in evidence cell.

```
| Priority | Area | Finding | Evidence | Fix | Effort | Confidence |
|----------|------|---------|----------|-----|--------|------------|
| P0 🚨 | Permissions | NSLocationWhenInUseUsageDescription is generic | ios/Runner/Info.plist line 12 | [plain English or code diff] | S | High |
```

Effort: S = <1hr · M = half day · L = multi-day

---

### Section 3 — Detailed findings

Group by area. Only include areas with findings — skip clean areas entirely.

Each finding:
- **What I found** — specific file, function, or string (not general)
- **Why Apple rejects it** — plain English (vibe) or guideline reference (technical)
- **Fix** — vibe: plain English + specific Claude Code prompt · technical: code diff

---

### Section 4 — Reviewer experience

Simulate what an Apple reviewer does. Mark each:
✅ likely passes · ⚠️ risky · ❌ will fail · ⬜ can't determine from code

```
Install & launch
  [?] App installs and opens without crashing

First run
  [?] Clear what the app does on first screen
  [?] Permissions explained before requested
  [?] No dead ends or blank screens

Core feature access
  [?] Reviewer can reach main features without special setup
  [?] Test account path exists (if login required)

Purchases (if applicable)
  [?] Paywall shows price, billing cycle, and terms
  [?] Restore purchases visible
  [?] No external payment links for digital features

Account (if applicable)
  [?] Account creation works
  [?] Account deletion findable and works in-app

Links & legal
  [?] Support URL loads
  [?] Privacy policy loads
  [?] Terms/EULA accessible

Edge cases
  [?] No internet → graceful error (not crash or blank)
  [?] Empty states have UI
```

---

### Section 5 — Draft reviewer notes

Generate ready-to-paste text for App Store Connect → App Information → Notes for App Review.
Base every section on what was actually found in the code — no generic placeholders where
the actual value can be inferred. Only use `[fill in]` where Claude genuinely can't know.

```
--- NOTES FOR APP REVIEW ---

ABOUT THIS APP
[Inferred from code — what the app does, who it's for]

TEST ACCOUNT
Email:    [fill in before submitting]
Password: [fill in before submitting]

HOW TO REACH KEY FEATURES
[For each major flow inferred from code:]
1. [Feature name]: [specific steps based on navigation structure found]
2. [Feature name]: [specific steps]
3. [If purchases]: Tap [inferred trigger] to reach the paywall.
   Use sandbox environment to test purchases.

PERMISSIONS
[For each NS*UsageDescription found:]
- [Permission]: Used for [feature name from usage string or inferred from code]
[If location not requested on launch, note it explicitly]

ACCOUNT DELETION
[If found]: Settings → [inferred path] → Delete Account.
Confirmation required. Account removed from servers immediately.
[If not found]: [omit this section]

[Add any section explaining unusual capabilities, credits systems,
or flows a reviewer might find confusing — based on what's in the code]
--- END NOTES ---
```

*Vibe coder intro:* "Here's what to paste into the 'Notes for App Review' box in App Store
Connect. It helps Apple's reviewer understand your app before they open it — fill in the
test account and submit."

*Technical intro:* "Draft App Review Notes — paste into App Store Connect → App Information
→ Notes for App Review. Fill in test credentials before submitting."

---

### Section 6 — Manual checklist

```
[VIBE CODER]
THINGS TO CHECK YOURSELF
(I can't see these from your code)
══════════════════════════════════
In App Store Connect:
  □ Fill in the App Privacy section — what data your app collects
  □ Age rating: 12+ if your app has chat, UGC, or location
  □ All subscription plans in ONE group (not separate groups)
  □ Add this link to your App Store description:
    apple.com/legal/internet-services/itunes/dev/stdeula/

Your listing:
  □ Description matches what's in the app right now
  □ Screenshots show the current version (not old designs or Figma)
  □ Support link opens and loads
  □ Privacy policy link opens and loads

Before submitting:
  □ Test on a real iPhone with the final build — not simulator
  □ Open on an iPad — shouldn't crash
  □ Paste your test account into the Review Notes above

[TECHNICAL]
MANUAL CHECKS — NOT IN CODE
════════════════════════════
App Store Connect
  □ App Privacy nutrition label completed
  □ Age rating 12+ (if UGC, chat, or location)
  □ All subscription tiers in single subscription group
  □ Apple EULA in App Store description
    → apple.com/legal/internet-services/itunes/dev/stdeula/
Metadata
  □ Description matches current build
  □ Screenshots match current build
  □ Support URL resolves · Privacy policy URL live
Device testing
  □ Physical device, release build · iPad doesn't crash
Review Notes
  □ Demo credentials added · Reviewer notes draft pasted
```

---

### Closing line

**Vibe coder:**
> "[X] things to fix. Start with the 🚨 ones — guaranteed rejections.
> Fill in your test account in the reviewer notes draft above, then paste it into
> App Store Connect before you submit.
> *(Say "fix [anything]" and I'll write exactly what to tell Claude Code.)*"

**Technical:**
> "[X] issues. P0s first.
> Reviewer notes draft above — fill credentials before submitting.
> *(Say "fix [issue]" for the corrected code.)*"

**If zero issues:**
> "No code-level issues found. Complete the reviewer notes draft and manual
> checklist — you're good to submit."

---

## Build checklist mode

Run this instead of the full audit if the user chose option B after mid-build detection.

Scan the codebase and output:

```
YOUR BUILD CHECKLIST
════════════════════
[What the app does so far — inferred from code]

FINISH THESE FIRST (before worrying about submission)
[List only the genuinely unfinished things found — stubs, TODOs,
missing flows, placeholder screens. Specific file references.]

ALREADY DONE ✅
[List what's clearly implemented and working]

WHEN YOU'RE DONE BUILDING
Come back and say "audit my app" — I'll check it for App Store submission then.
```

Tone: encouraging, not alarming. No rejection warnings. Focus on what to build next.

---

## Re-audit

If user says "I fixed [X]" or shares updated code:
- Re-scan only that area
- Use actual file/function found in the new code
- Confirm ✅ or explain specifically what's still wrong
- Update that item in the risk register only
