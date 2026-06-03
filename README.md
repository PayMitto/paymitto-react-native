# PayMitto SDK - React Native

Distribution of the PayMitto SDK wrapper for React Native.

**Repository:** https://github.com/PayMitto/paymitto-react-native

## Requirements

- Node.js 20+
- React Native 0.76+
- iOS 16.0+ / Android API 26+
- Xcode 26.0+ (iOS)
- JDK 17+ (Android)

## Developer Documentation

- [React Native](https://docs.paymitto.com/docs/Getting%20started/c2c-via-sdk/mobile-sdk-react-native/)

## Installation

```bash
npm install react-native-paymitto-sdk-11.0.0.tgz
```

Or with Yarn:

```bash
yarn add file:./react-native-paymitto-sdk-11.0.0.tgz
```

### iOS Setup

After installing, run CocoaPods:

```bash
cd ios && pod install
```

In your `Podfile`, add the embed script to ensure the binary framework is bundled correctly:

```ruby
require_relative '../node_modules/react-native-paymitto-sdk/ios/embed_paymitto'

# Inside your target's post_install block:
post_install do |installer|
  # ... existing post_install steps ...
  embed_paymitto_framework!(installer)
end
```

### Expo

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
          "targetSdkVersion": 35,
          "minSdkVersion": 26
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

## Usage

```typescript
import {
  startSDK,
  PayMittoEnvironment,
  PayMittoLocalization,
  PayMittoSupportedAppearance,
  type PayMittoConfiguration,
} from 'react-native-paymitto-sdk';

const configuration: PayMittoConfiguration = {
  environment: PayMittoEnvironment.Sandbox,
  supportedAppearance: PayMittoSupportedAppearance.Device,
  localization: PayMittoLocalization.EN_US,
};

startSDK({
  configuration,
  fetchAccessTokenDetails,
  verifyFundsAndCreateTransfer,
});
```

See developer documentation above for full integration details.

## License

Proprietary - PayMitto, LLC
