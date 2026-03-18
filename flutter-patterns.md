# Flutter-Specific App Store Patterns

Reference file for the auditor. Load this when the user's stack is Flutter, React Native, or other cross-platform frameworks.

---

## Location (Flutter)

**Package**: `geolocator`, `location`, or `permission_handler`

### ❌ Rejection pattern — permission requested after access
```dart
// BAD: accessing location before requesting permission
Position position = await Geolocator.getCurrentPosition();

// BAD: calling permission check and immediately using location
await Geolocator.checkPermission();
Position position = await Geolocator.getCurrentPosition(); // no await for request
```

### ✅ Correct pattern
```dart
// GOOD: request first, check result, then access
LocationPermission permission = await Geolocator.requestPermission();
if (permission == LocationPermission.denied) return;
Position position = await Geolocator.getCurrentPosition();
```

### Info.plist strings (still required even in Flutter)
Flutter apps still require `NSLocationWhenInUseUsageDescription` in `ios/Runner/Info.plist`.
Check `ios/Runner/Info.plist` — NOT `pubspec.yaml`.

### ❌ Bad string
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>This app requires location access.</string>
```

### ✅ Good string
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>Your location shows listings near you and displays your city on your profile. Never shared without your consent.</string>
```

---

## Account Deletion (Flutter)

### ❌ Rejection pattern — email-based deletion
```dart
// BAD: opening email client
final Uri emailUri = Uri(scheme: 'mailto', path: 'support@app.com', ...);
launchUrl(emailUri);

// BAD: just navigating to a web form
launchUrl(Uri.parse('https://myapp.com/delete-account'));
```

### ✅ Correct pattern
```dart
// GOOD: direct API call from within the app
Future<void> deleteAccount() async {
  final confirmed = await showConfirmationDialog(context);
  if (!confirmed) return;
  
  await authService.deleteAccount(); // triggers server-side deletion
  await FirebaseAuth.instance.currentUser?.delete();
  // then sign out and navigate to onboarding
}
```

**AWS Cognito + Firebase note:** Both need to be deleted — the Cognito user pool entry AND the Firebase Auth user. Deleting only one will leave orphaned data and may cause issues on re-registration.

---

## Block User (Flutter)

### ❌ Rejection pattern — report only, no block
```dart
// BAD: only report available
void reportUser(String userId) {
  firestoreService.reportUser(userId);
  showSnackbar('User reported');
}
// No block functionality anywhere
```

### ✅ Correct pattern
```dart
// GOOD: separate block action
Future<void> blockUser(String targetUserId) async {
  await firestoreService.addToBlockList(
    currentUserId: currentUser.id,
    blockedUserId: targetUserId,
  );
  // Update local state immediately so UI reflects block
}

// Block must be surfaced in BOTH places:
// 1. User profile screen — context menu or button
// 2. Chat screen — message long-press or header menu
```

**Firestore security rules must enforce block:**
```javascript
// Blocked users must not be able to read or write to each other's data
allow read: if !isBlockedBy(request.auth.uid, resource.data.userId);
```

---

## In-App Purchases / Subscriptions (Flutter)

**Package**: `in_app_purchase`, `purchases_flutter` (RevenueCat), `qonversion`

### ❌ Rejection pattern — wrong payment method for digital goods
```dart
// BAD: Stripe for a subscription
final result = await Stripe.instance.confirmPayment(...);

// BAD: custom credits system that bypasses IAP for digital features
await creditsService.purchaseCredits(amount: 100); // if credits unlock digital content
```

### ✅ Correct pattern
```dart
// GOOD: StoreKit via in_app_purchase or RevenueCat
final CustomerInfo customerInfo = await Purchases.purchasePackage(package);
```

**Credits system note:** If credits are used to unlock digital features (messaging, boosts, premium listings), they must go through Apple IAP. Credits for physical goods (like escrow for swaps) are generally fine outside IAP.

### ❌ Missing restore purchases
```dart
// Must exist somewhere in your UI — usually Settings
await InAppPurchase.instance.restorePurchases();
```

---

## Privacy / Permission Strings (Flutter → Info.plist)

All permission strings live in `ios/Runner/Info.plist`. Flutter's `pubspec.yaml` permissions config does NOT replace these.

Common keys to check:
```xml
NSLocationWhenInUseUsageDescription   <!-- if using location -->
NSCameraUsageDescription              <!-- if using camera -->
NSPhotoLibraryUsageDescription        <!-- if accessing photo library -->
NSMicrophoneUsageDescription          <!-- if using microphone -->
NSContactsUsageDescription            <!-- if accessing contacts -->
NSUserTrackingUsageDescription        <!-- if using ATT/IDFA -->
```

Each must have a specific, honest description. Empty strings or generic text = rejection.

---

## IPv6 / Network (Flutter)

**Package**: `dio`, `http`

### ❌ Rejection pattern — hardcoded IPs
```dart
// BAD: hardcoded IPv4
static const String baseUrl = 'http://192.168.1.100:8080';
static const String apiUrl = 'http://54.201.12.34/api/v1';

// BAD: http instead of https in production
static const String baseUrl = 'http://myapp.com/api'; // should be https
```

### ✅ Correct pattern
```dart
// GOOD: domain name + https
static const String baseUrl = 'https://api.myapp.com/v1';
```

**Firebase + AWS note:** Firebase SDK and AWS Amplify connect via domain names internally — these are fine. Only custom API base URLs are at risk.

---

## Metadata / App Store Connect (Flutter)

- Screenshots must be taken from the actual running app — not Figma mockups placed on device frames
- Flutter's hot reload can cause visual glitches in screenshots — take them from a clean cold launch
- iPad screenshots: Flutter apps need iPad screenshots even if not iPad-optimized. Use a simulator.

---

## TestFlight vs App Store (Flutter)

Flutter-specific gotcha: Firebase App Distribution ≠ TestFlight. Apple only recognizes TestFlight for beta distribution. Passing Firebase distribution does not mean you'll pass App Store review.

Also: Flutter's `--release` build can behave differently from `--debug` or `--profile`. Always test the release build on a physical device before submitting.
