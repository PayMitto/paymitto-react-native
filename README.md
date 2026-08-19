# PayMitto SDK - React Native

Distribution of the PayMitto SDK wrapper for React Native.

**Repository:** https://github.com/PayMitto/paymitto-react-native

**Current version:** 11.0.1 — bundles PayMitto iOS SDK 11.0.2 and Android SDK 11.0.4

## Requirements

- Node.js 20+
- React Native 0.76+ with the New Architecture enabled (the wrapper is a TurboModule)
- iOS 16.0+ / Android API 26+
- Xcode 26.0+ (iOS)
- JDK 17+ (Android)

## Developer Documentation

- [React Native](https://docs.paymitto.com/docs/Getting%20started/c2c-via-sdk/mobile-sdk-react-native/)

## Installation

Download `react-native-paymitto-sdk-11.0.1.tgz` from the [latest release](https://github.com/PayMitto/paymitto-react-native/releases/latest), then install it:

```bash
npm install react-native-paymitto-sdk-11.0.1.tgz
```

Or with Yarn:

```bash
yarn add file:./react-native-paymitto-sdk-11.0.1.tgz
```

Then follow the setup for your project type below — **Expo** or **bare React Native**.

## Setup — Expo

Add the plugin to your `app.json`:

```json
{
  "expo": {
    "plugins": [
      "react-native-paymitto-sdk/plugin/with-paymitto-sdk",
      ["expo-build-properties", {
        "ios": { "deploymentTarget": "16.0" },
        "android": {
          "compileSdkVersion": 36,
          "targetSdkVersion": 36,
          "minSdkVersion": 26,
          "kotlinVersion": "2.1.20"
        }
      }]
    ]
  }
}
```

Then run:

```bash
npx expo prebuild --clean
```

The plugin applies the iOS Podfile changes and the Android Gradle properties described below, so there is nothing else to configure.

## Setup — Bare React Native

### iOS

In your `Podfile`:

```ruby
platform :ios, '16.0'

require_relative '../node_modules/react-native-paymitto-sdk/ios/embed_paymitto'

target 'YourApp' do
  use_frameworks! :linkage => :dynamic

  # ... existing configuration ...

  post_install do |installer|
    # ... existing post_install steps ...
    embed_paymitto_framework!(installer)
  end
end
```

Then install the pods:

```bash
cd ios && pod install
```

Why each line is needed:

| Line | Reason |
| --- | --- |
| `platform :ios, '16.0'` | The native SDK requires iOS 16.0 or later. |
| `use_frameworks! :linkage => :dynamic` | The native SDK ships as a Swift Package with binary `xcframework`s. Under CocoaPods' default static linking you will get a warning about possible linker errors at install time. |
| `embed_paymitto_framework!(installer)` | CocoaPods links Swift Package dependencies but does not embed their dynamic frameworks into the app bundle. This helper adds a build phase that copies `PayMittoSDK.framework` and `VisaSensoryBranding.framework` into the app. Without it the app compiles but crashes on launch. |

### Android

Make sure `minSdkVersion` is at least **26** in `android/build.gradle`:

```gradle
buildscript {
    ext {
        minSdkVersion = 26
        // ... your existing values ...
    }
}
```

That is the only required change. The wrapper's own Gradle module supplies its Kotlin, Jetpack Compose and Material dependencies, so you do **not** need to add the Compose compiler plugin, `buildFeatures { compose true }`, `composeOptions`, or any Compose dependency to your app.

What the native SDK merges into your app's manifest:

- Permissions: `INTERNET`, `ACCESS_NETWORK_STATE` and `CAMERA` (used for identity document capture)
- `android:allowBackup="false"` on `<application>`, declared with `tools:replace` — this overrides the value in your app's manifest

## Usage

```typescript
import {
  startSDK,
  PayMittoEnvironment,
  PayMittoLocalization,
  PayMittoSupportedAppearance,
  type PayMittoConfiguration,
  type PayMittoTokenResponse,
  type PayMittoTransferRequest,
  type PayMittoTransferResponse,
  type PayMittoError,
} from 'react-native-paymitto-sdk';

const configuration: PayMittoConfiguration = {
  environment: PayMittoEnvironment.Sandbox,
  supportedAppearance: PayMittoSupportedAppearance.Device,
  localization: PayMittoLocalization.EN_US,
};

// Called whenever the SDK needs an access token. Fetch it from your backend —
// never ship PayMitto credentials inside the app.
async function fetchAccessTokenDetails(): Promise<PayMittoTokenResponse | PayMittoError> {
  const response = await fetch('https://your-backend.example.com/paymitto/token');

  if (!response.ok) {
    return { code: 'TOKEN_ERROR', message: 'Could not fetch access token' };
  }

  // { accessToken, expiresIn, scope, tokenType }
  return response.json();
}

// Called when the sender confirms a transfer. Verify the funds on your side,
// create the transfer through your backend, and return its id.
async function verifyFundsAndCreateTransfer(
  request: PayMittoTransferRequest,
): Promise<PayMittoTransferResponse | PayMittoError> {
  const response = await fetch('https://your-backend.example.com/paymitto/transfers', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(request),
  });

  if (!response.ok) {
    return { code: 'TRANSFER_ERROR', message: 'Could not create transfer' };
  }

  // { transferId }
  return response.json();
}

startSDK({
  configuration,
  fetchAccessTokenDetails,
  verifyFundsAndCreateTransfer,
  onLoad: async () => console.log('PayMitto SDK loaded'),
  onClose: async () => console.log('PayMitto SDK closed'),
});
```

Both callbacks may resolve to a `PayMittoError` (`{ code, message }`) instead of a success value — the SDK surfaces the failure to the sender rather than throwing.

### Showing an existing transfer

To open the details of a transfer that already exists, use `startTransferDetails`. It takes the same configuration and token callback, plus the transfer id:

```typescript
import { startTransferDetails } from 'react-native-paymitto-sdk';

startTransferDetails({
  transferId: 'transfer-id-from-your-backend',
  configuration,
  fetchAccessTokenDetails,
});
```

See the developer documentation above for full integration details, including appearance customization and the complete configuration reference.

## License

Proprietary - PayMitto, LLC
