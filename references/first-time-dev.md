# First-Time App Store Submission Checklist

Load this file when the user mentions "first app", "first time submitting", or "new developer."
Walk through each section before running the main audit.

---

## Before you can submit — the basics

These are things that block your upload entirely, before Apple even reviews your app.
Most first-time developers hit at least one of these.

### Apple Developer Program
- [ ] Enrolled in Apple Developer Program ($99/year at developer.apple.com)
- [ ] Accepted the latest Apple Developer Program License Agreement
      → Check at developer.apple.com — a banner appears if you need to accept
- [ ] If selling paid apps or IAP: tax and banking info completed in App Store Connect
      → App Store Connect → Agreements, Tax, and Banking

### App Store Connect setup
- [ ] New app record created in App Store Connect (appstoreconnect.apple.com)
- [ ] Bundle ID in App Store Connect matches your Xcode project exactly
      → Common mistake: `com.yourname.AppName` vs `com.yourname.appname` — case matters
- [ ] Primary language set
- [ ] Pricing and availability configured (even for free apps)

### Xcode setup
- [ ] Distribution certificate created in developer.apple.com → Certificates
- [ ] App Store provisioning profile created and downloaded
- [ ] Correct team selected in Xcode → Signing & Capabilities
- [ ] Bundle identifier in Xcode matches App Store Connect exactly
- [ ] Built with current Xcode GM release (Xcode 16 as of 2025)

---

## Common first-time mistakes

Present this as a table. Flag any that might apply based on what's visible in the project.

| Mistake | What happens | Fix |
|---|---|---|
| Bundle ID mismatch between Xcode and ASC | Upload rejected immediately | Match exactly, including case |
| Distribution cert missing or expired | Archive fails to upload | Create new cert in Developer Portal |
| License agreement not accepted | Upload blocked with no clear message | Accept at developer.apple.com |
| Tax forms incomplete | Paid app won't distribute | Complete in ASC → Agreements, Tax, Banking |
| Wrong team selected in Xcode | Signing errors on archive | Signing & Capabilities → correct team |
| App ID not registered | Upload fails | Register App ID in developer.apple.com first |
| Using personal/development cert instead of distribution | Archive shows wrong cert type | Create distribution certificate specifically |

---

## Vibe coder version

If in vibe coder mode, present this as a plain English checklist:

**Before you can submit your first app, you need to set up a few things:**

1. **Apple Developer account** — go to developer.apple.com and pay the $99/year fee.
   No account = can't submit, full stop.

2. **Accept the license agreement** — Apple updates this every year. Log in and look
   for a banner asking you to accept. Easy to miss, blocks everything if you don't.

3. **App Store Connect record** — go to appstoreconnect.apple.com and create a new app.
   The bundle ID here must match your Xcode project exactly — copy-paste it, don't type it.

4. **Distribution certificate** — in your Apple Developer account, go to Certificates and
   create an "Apple Distribution" certificate. Download it and double-click to install.
   Tell Claude Code: "Set up my Xcode project for App Store distribution with my Apple
   Developer account [your email]"

5. **Tax info (if charging money)** — in App Store Connect go to Agreements, Tax, and Banking.
   Fill this in before submitting or your app won't appear in the store even after approval.

---

## After setup — continue with the main audit

Once the user confirms setup is done (or these items are already in place),
continue with the full code scan from the main SKILL.md.

Tell the user:
> "Setup looks good — now let me scan your code for anything Apple would reject."
> Then proceed with Steps 3-5 of the main audit.
