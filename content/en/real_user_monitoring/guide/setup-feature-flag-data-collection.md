---
title: Getting Started with Feature Flag Data in RUM

beta: true
description: Learn how to set up RUM to capture feature flag data and analyze the performance in Datadog
aliases:
- /real_user_monitoring/guide/getting-started-feature-flags/
further_reading:
- link: '/real_user_monitoring/feature_flag_tracking'
  tag: 'Documentation'
  text: 'Analyze your feature flag data with Feature Flag Tracking'
- link: '/real_user_monitoring/explorer'
  tag: 'Documentation'
  text: 'Visualize your RUM data in the RUM Explorer'
---

## Overview
Feature flag data gives you greater visibility into your user experience and performance monitoring by allowing you to determine which users are being shown a specific feature and if any change you introduce is impacting your user experience or negatively affecting performance.

By enriching your RUM data with feature flag data, you can be confident that your feature successfully launches without unintentionally causing a bug or performance regression. With this additional layer of insight, you can correlate feature releases with performance, pinpoint issues to specific releases, and troubleshoot faster.

## Setup

{{< tabs >}}
{{% tab "Browser" %}}

Feature flag tracking is available in the RUM Browser SDK. To start, set up [RUM browser monitoring][1]. You need the Browser RUM SDK version >= 4.25.0.

<details>
  <summary>Before <code>v5.17.0</code></summary>

If you are using a version previous to 5.17.0, initialize the RUM SDK and configure the `enableExperimentalFeatures` initialization parameter with `["feature_flags"]` to start collecting feature flag data.

{{% collapse-content title="NPM" level="h4" %}}
```javascript
  import { datadogRum } from '@datadog/browser-rum';

  // Initialize Datadog Browser SDK
  datadogRum.init({
    ...
    enableExperimentalFeatures: ["feature_flags"],
    ...
});
```
{{% /collapse-content %}}

{{% collapse-content title="CDN async" level="h4" %}}
```javascript
window.DD_RUM.onReady(function() {
    window.DD_RUM.init({
      ...
      enableExperimentalFeatures: ["feature_flags"],
      ...
    })
})
```
{{% /collapse-content %}}

{{% collapse-content title="CDN sync" level="h4" %}}
```javascript
window.DD_RUM &&
    window.DD_RUM.init({
      ...
      enableExperimentalFeatures: ["feature_flags"],
      ...
    })
```
{{% /collapse-content %}}

</details>
<br/>

[1]: /real_user_monitoring/browser/setup/
{{% /tab %}}
{{% tab "Android" %}}

Feature flag tracking is available in the RUM Android SDK. To start, set up [RUM Android monitoring][1]. You need the Android RUM SDK version >= 1.18.0.

[1]: /real_user_monitoring/mobile_and_tv_monitoring/android/setup/
{{% /tab %}}
{{% tab "Flutter" %}}

Feature flag tracking is available for your Flutter applications. To start, set up [RUM Flutter monitoring][1]. You need the Flutter Plugin version >= 1.3.2.

[1]: /real_user_monitoring/mobile_and_tv_monitoring/flutter/setup/
{{% /tab %}}
{{% tab "iOS" %}}

Feature flag tracking is available in the RUM iOS SDK. To start, set up [RUM iOS monitoring][1]. You need the iOS RUM SDK version >= 1.16.0.

[1]: /real_user_monitoring/mobile_and_tv_monitoring/ios/setup
{{% /tab %}}
{{% tab "Kotlin Multiplatform" %}}

Feature flag tracking is available for your Kotlin Multiplatform applications. To start, set up [RUM Kotlin Multiplatform monitoring][1].

[1]: /real_user_monitoring/mobile_and_tv_monitoring/kotlin_multiplatform
{{% /tab %}}
{{% tab "React Native" %}}

Feature flag tracking is available for your React Native applications. To start, set up [RUM React Native monitoring][1]. You need the React Native RUM SDK version >= 1.7.0.

[1]: /real_user_monitoring/mobile_and_tv_monitoring/react_native/setup
{{% /tab %}}
{{% tab "Unity" %}}

Feature flag tracking is available for your Unity applications. To start, set up [RUM Unity monitoring][1].

[1]: /real_user_monitoring/mobile_and_tv_monitoring/unity/setup
{{% /tab %}}
{{< /tabs >}}

## Integrations

You can start collecting feature flag data with [custom feature flag management solutions](#custom-feature-flag-management), or by using one of Datadog's integration partners.

Datadog supports integrations with:
{{< partial name="rum/rum-feature-flag-tracking.html" >}}


</br>

### Amplitude integration

{{< tabs >}}
{{% tab "Browser" %}}

Initialize Amplitude's SDK and create an exposure listener reporting feature flag evaluations to Datadog using the following snippet of code:

For more information about initializing Amplitude's SDK, see Amplitude's [JavaScript SDK documentation][1].

```javascript
  const experiment = Experiment.initialize("CLIENT_DEPLOYMENT_KEY", {
    exposureTrackingProvider: {
      track(exposure: Exposure)  {
        // Send the feature flag when Amplitude reports the exposure
        datadogRum.addFeatureFlagEvaluation(exposure.flag_key, exposure.variant);
      }
    }
  })
```


[1]: https://www.docs.developers.amplitude.com/experiment/sdks/javascript-sdk/

{{% /tab %}}
{{% tab "iOS" %}}

Initialize Amplitude's SDK and create an inspector reporting feature flag evaluations to Datadog using the snippet of code below.

For more information about initializing Amplitude's SDK, see Amplitude's [iOS SDK documentation][1].

```swift
  class DatadogExposureTrackingProvider : ExposureTrackingProvider {
    func track(exposure: Exposure) {
      // Send the feature flag when Amplitude reports the exposure
      if let variant = exposure.variant {
        RUMMonitor.shared().addFeatureFlagEvaluation(name: exposure.flagKey, value: variant)
      }
    }
  }

  // In initialization:
  ExperimentConfig config = ExperimentConfigBuilder()
    .exposureTrackingProvider(DatadogExposureTrackingProvider(analytics))
    .build()
```

[1]: https://www.docs.developers.amplitude.com/experiment/sdks/ios-sdk/


{{% /tab %}}
{{% tab "Android" %}}

Initialize Amplitude's SDK and create an inspector reporting feature flag evaluations to Datadog using the snippet of code below.

For more information about initializing Amplitude's SDK, see Amplitude's [Android SDK documentation][1].

```kotlin
  internal class DatadogExposureTrackingProvider : ExposureTrackingProvider {
    override fun track(exposure: Exposure) {
        // Send the feature flag when Amplitude reports the exposure
        GlobalRumMonitor.get().addFeatureFlagEvaluation(
            exposure.flagKey,
            exposure.variant.orEmpty()
        )
    }
  }

  // In initialization:
  val config = ExperimentConfig.Builder()
      .exposureTrackingProvider(DatadogExposureTrackingProvider())
      .build()
```

[1]: https://www.docs.developers.amplitude.com/experiment/sdks/android-sdk/


{{% /tab %}}
{{% tab "Flutter" %}}

Amplitude does not support this integration. Create a ticket with Amplitude to request this feature.


{{% /tab %}}
{{< /tabs >}}

### ConfigCat integration

{{< tabs >}}
{{% tab "Browser" %}}

When initializing the ConfigCat Javascript SDK, subscribe to the `flagEvaluated` event and report feature flag evaluations to Datadog:

```javascript
const configCatClient = configcat.getClient(
  '#YOUR-SDK-KEY#',
  configcat.PollingMode.AutoPoll,
  {
    setupHooks: (hooks) =>
      hooks.on('flagEvaluated', (details) => {
        datadogRum.addFeatureFlagEvaluation(details.key, details.value);
      })
  }
);
```

For more information about initializing the ConfigCat Javascript SDK, see ConfigCat's [JavaScript SDK documentation][1].

[1]: https://configcat.com/docs/sdk-reference/js


{{% /tab %}}
{{% tab "iOS" %}}

When initializing the ConfigCat Swift iOS SDK, subscribe to the `flagEvaluated` event and report feature flag evaluations to Datadog:

```swift
  let client = ConfigCatClient.get(sdkKey: "#YOUR-SDK-KEY#") { options in
    options.hooks.addOnFlagEvaluated { details in
        RUMMonitor.shared().addFeatureFlagEvaluation(featureFlag: details.key, variation: details.value)
    }
  }
```

For more information about initializing the ConfigCat Swift (iOS) SDK, see ConfigCat's[Swift iOS SDK documentation][1].

[1]: https://configcat.com/docs/sdk-reference/ios


{{% /tab %}}
{{% tab "Android" %}}

When initializing the ConfigCat Android SDK, subscribe to the `flagEvaluated` event and report feature flag evaluations to Datadog:

```java
  ConfigCatClient client = ConfigCatClient.get("#YOUR-SDK-KEY#", options -> {
    options.hooks().addOnFlagEvaluated(details -> {
        GlobalRumMonitor.get().addFeatureFlagEvaluation(details.key, details.value);
    });
  });
```

For more information about initializing the ConfigCat Android SDK, see ConfigCat's [Android SDK documentation][1].

[1]: https://configcat.com/docs/sdk-reference/android


{{% /tab %}}
{{% tab "Flutter" %}}

When initializing the ConfigCat Dart SDK, subscribe to the `flagEvaluated` event and report feature flag evaluations to Datadog:

```dart
  final client = ConfigCatClient.get(
    sdkKey: '#YOUR-SDK-KEY#',
    options: ConfigCatOptions(
        pollingMode: PollingMode.autoPoll(),
        hooks: Hooks(
            onFlagEvaluated: (details) => {
              DatadogSdk.instance.rum?.addFeatureFlagEvaluation(details.key, details.value);
            }
        )
    )
  );
```

For more information about initializing the ConfigCat Dart (Flutter) SDK, see ConfigCat's [Dart SDK documentation][1].

[1]: https://configcat.com/docs/sdk-reference/dart


{{% /tab %}}


{{% tab "React Native" %}}

When initializing the ConfigCat React SDK, subscribe to the `flagEvaluated` event and report feature flag evaluations to Datadog:

```typescript
<ConfigCatProvider
  sdkKey="YOUR_SDK_KEY"
  pollingMode={PollingMode.AutoPoll}
  options={{
    setupHooks: (hooks) =>
      hooks.on('flagEvaluated', (details) => {
        DdRum.addFeatureFlagEvaluation(details.key, details.value);
      }),
  }}
>
  ...
</ConfigCatProvider>
```

For more information about initializing the ConfigCat React SDK, see ConfigCat's [React SDK documentation][1].

[1]: https://configcat.com/docs/sdk-reference/react

{{% /tab %}}
{{< /tabs >}}

### Custom feature flag management

{{< tabs >}}
{{% tab "Browser" %}}

Each time a feature flag is evaluated, add the following function to send the feature flag information to RUM:

```javascript
datadogRum.addFeatureFlagEvaluation(key, value);
```

{{% /tab %}}
{{% tab "iOS" %}}

Each time a feature flag is evaluated, add the following function to send the feature flag information to RUM:

   ```swift
   RUMMonitor.shared().addFeatureFlagEvaluation(key, value);
   ```

{{% /tab %}}
{{% tab "Android" %}}

Each time a feature flag is evaluated, add the following function to send the feature flag information to RUM:

   ```kotlin
   GlobalRumMonitor.get().addFeatureFlagEvaluation(key, value);
   ```

{{% /tab %}}
{{% tab "Flutter" %}}

Each time a feature flag is evaluated, add the following function to send the feature flag information to RUM:

   ```dart
   DatadogSdk.instance.rum?.addFeatureFlagEvaluation(key, value);
   ```
{{% /tab %}}
{{% tab "React Native" %}}

Each time a feature flag is evaluated, add the following function to send the feature flag information to RUM:

   ```javascript
   DdRum.addFeatureFlagEvaluation(key, value);
   ```

{{% /tab %}}
{{< /tabs >}}

### DevCycle integration

{{< tabs >}}
{{% tab "Browser" %}}

Initialize DevCycle's SDK and subscribe to the `variableEvaluated` event, choosing to subscribe to all variable evaluations `variableEvaluated:*` or particular variable evaluations `variableEvaluated:my-variable-key`.

For more information about initializing DevCycle's SDK, see [DevCycle's JavaScript SDK documentation][5] and for more information about DevCycle's event system, see [DevCycle's SDK Event Documentation][6].

```javascript
const user = { user_id: "<USER_ID>" };
const dvcOptions = { ... };
const dvcClient = initialize("<DVC_CLIENT_SDK_KEY>", user, dvcOptions);
...
dvcClient.subscribe(
    "variableEvaluated:*",
    (key, variable) => {
        // track all variable evaluations
        datadogRum.addFeatureFlagEvaluation(key, variable.value);
    }
)
...
dvcClient.subscribe(
    "variableEvaluated:my-variable-key",
    (key, variable) => {
        // track a particular variable evaluation
        datadogRum.addFeatureFlagEvaluation(key, variable.value);
    }
)
```


[5]: https://docs.devcycle.com/sdk/client-side-sdks/javascript/javascript-install
[6]: https://docs.devcycle.com/sdk/client-side-sdks/javascript/javascript-usage#subscribing-to-sdk-events
{{% /tab %}}
{{% tab "iOS" %}}

DevCycle does not support this integration. Create a ticket with DevCycle to request this feature.


{{% /tab %}}
{{% tab "Android" %}}

DevCycle does not support this integration. Create a ticket with DevCycle to request this feature.


{{% /tab %}}
{{% tab "Flutter" %}}

DevCycle does not support this integration. Create a ticket with DevCycle to request this feature.


{{% /tab %}}
{{% tab "React Native" %}}

DevCycle does not support this integration. Create a ticket with DevCycle to request this feature.


{{% /tab %}}
{{< /tabs >}}

### Eppo integration

{{< tabs >}}
{{% tab "Browser" %}}

Initialize Eppo's SDK and create an assignment logger that additionally reports feature flag evaluations to Datadog using the snippet of code shown below.

For more information about initializing Eppo's SDK, see [Eppo's JavaScript SDK documentation][1].

```typescript
const assignmentLogger: IAssignmentLogger = {
  logAssignment(assignment) {
    datadogRum.addFeatureFlagEvaluation(assignment.featureFlag, assignment.variation);
  },
};

await eppoInit({
  apiKey: "<API_KEY>",
  assignmentLogger,
});
```

[1]: https://docs.geteppo.com/sdks/client-sdks/javascript
{{% /tab %}}
{{% tab "iOS" %}}

Initialize Eppo's SDK and create an assignment logger that additionally reports feature flag evaluations to Datadog using the snippet of code shown below.

For more information about initializing Eppo's SDK, see [Eppo's iOS SDK documentation][1].

```swift
func IAssignmentLogger(assignment: Assignment) {
  RUMMonitor.shared().addFeatureFlagEvaluation(featureFlag: assignment.featureFlag, variation: assignment.variation)
}

let eppoClient = EppoClient(apiKey: "mock-api-key", assignmentLogger: IAssignmentLogger)
```

[1]: https://docs.geteppo.com/sdks/client-sdks/ios

{{% /tab %}}
{{% tab "Android" %}}

Initialize Eppo's SDK and create an assignment logger that additionally reports feature flag evaluations to Datadog using the snippet of code shown below.

For more information about initializing Eppo's SDK, see [Eppo's Android SDK documentation][1].

```java
AssignmentLogger logger = new AssignmentLogger() {
    @Override
    public void logAssignment(Assignment assignment) {
      GlobalRumMonitor.get().addFeatureFlagEvaluation(assignment.getFeatureFlag(), assignment.getVariation());
    }
};

EppoClient eppoClient = new EppoClient.Builder()
    .apiKey("YOUR_API_KEY")
    .assignmentLogger(logger)
    .application(application)
    .buildAndInit();
```


[1]: https://docs.geteppo.com/sdks/client-sdks/android

{{% /tab %}}
{{% tab "Flutter" %}}

Eppo does not support this integration. [Contact Eppo][1] to request this feature.

[1]: mailto:support@geteppo.com

{{% /tab %}}
{{% tab "React Native" %}}

Initialize Eppo's SDK and create an assignment logger that additionally reports feature flag evaluations to Datadog using the snippet of code shown below.

For more information about initializing Eppo's SDK, see [Eppo's React native SDK documentation][1].

```typescript
const assignmentLogger: IAssignmentLogger = {
  logAssignment(assignment) {
    DdRum.addFeatureFlagEvaluation(assignment.featureFlag, assignment.variation);
  },
};

await eppoInit({
  apiKey: "<API_KEY>",
  assignmentLogger,
});
```

[1]: https://docs.geteppo.com/sdks/client-sdks/react-native

{{% /tab %}}
{{< /tabs >}}

### Flagsmith Integration

{{< tabs >}}
{{% tab "Browser" %}}

Initialize Flagsmith's SDK with the `datadogRum` option, which reports feature flag evaluations to Datadog using the snippet of code shown below.

   Optionally, you can configure the client so that Flagsmith traits are sent to Datadog via `datadogRum.setUser()`. For more information about initializing Flagsmith's SDK, check out [Flagsmith's JavaScript SDK documentation][1].

   ```javascript
    // Initialize the Flagsmith SDK
    flagsmith.init({
        datadogRum: {
            client: datadogRum,
            trackTraits: true,
        },
        ...
    })
   ```


[1]: https://docs.flagsmith.com/clients/javascript
{{% /tab %}}
{{% tab "iOS" %}}

Flagsmith does not support this integration. Create a ticket with Flagsmith to request this feature.


{{% /tab %}}
{{% tab "Android" %}}

Flagsmith does not support this integration. Create a ticket with Flagsmith to request this feature.

{{% /tab %}}
{{% tab "Flutter" %}}

Flagsmith does not support this integration. Create a ticket with Flagsmith to request this feature.

{{% /tab %}}
{{% tab "React Native" %}}

Flagsmith does not currently support this integration. Create a ticket with Flagsmith to request this feature.

{{% /tab %}}
{{< /tabs >}}

### GrowthBook integration

{{< tabs >}}
{{% tab "Browser" %}}

When initializing the GrowthBook SDK, report feature flag evaluations to Datadog by using the `onFeatureUsage` callback.

For more information about initializing GrowthBook's SDK, see [GrowthBook's JavaScript SDK documentation][1].

```javascript
const gb = new GrowthBook({
  ...,
  onFeatureUsage: (featureKey, result) => {
    datadogRum.addFeatureFlagEvaluation(featureKey, result.value);
  },
});

gb.init();
```

[1]: https://docs.growthbook.io/lib/js#step-1-configure-your-app

{{% /tab %}}
{{% tab "iOS" %}}

GrowthBook does not support this integration. Contact GrowthBook to request this feature.

{{% /tab %}}
{{% tab "Android" %}}

When initializing the GrowthBook SDK, report feature flag evaluations to Datadog by calling `setFeatureUsageCallback`.

For more information about initializing GrowthBook's SDK, see [GrowthBook's Android SDK documentation][1].

```kotlin
val gbBuilder = GBSDKBuilder(...)

gbBuilder.setFeatureUsageCallback { featureKey, result ->
  GlobalRumMonitor.get().addFeatureFlagEvaluation(featureKey, result.value);
}

val gb = gbBuilder.initialize()
```

[1]: https://docs.growthbook.io/lib/kotlin#quick-usage

{{% /tab %}}
{{% tab "Flutter" %}}

When initializing the GrowthBook SDK, report feature flag evaluations to Datadog by calling `setFeatureUsageCallback`.

For more information about initializing GrowthBook's SDK, see [GrowthBook's Flutter SDK documentation][1].

```dart
final gbBuilder = GBSDKBuilderApp(...);
gbBuilder.setFeatureUsageCallback((featureKey, result) {
  DatadogSdk.instance.rum?.addFeatureFlagEvaluation(featureKey, result.value);
});
final gb = await gbBuilder.initialize();
```

[1]: https://docs.growthbook.io/lib/flutter#quick-usage

{{% /tab %}}
{{% tab "React Native" %}}

When initializing the GrowthBook SDK, report feature flag evaluations to Datadog by using the `onFeatureUsage` callback.

For more information about initializing GrowthBook's SDK, see [GrowthBook's React Native SDK documentation][1].

```javascript
const gb = new GrowthBook({
  ...,
  onFeatureUsage: (featureKey, result) => {
    datadogRum.addFeatureFlagEvaluation(featureKey, result.value);
  },
});

gb.init();
```

[1]: https://docs.growthbook.io/lib/react-native#step-1-configure-your-app

{{% /tab %}}
{{< /tabs >}}

### Kameleoon integration

{{< tabs >}}
{{% tab "Browser" %}}

After creating and initializing the Kameleoon SDK, subscribe to the `Evaluation` event using the `onEvent` handler.

For more information about the SDK, see [Kameleoon JavaScript SDK documentation][1].

```javascript
client.onEvent(EventType.Evaluation, ({ featureKey, variation }) => {
  datadogRum.addFeatureFlagEvaluation(featureKey, variation.key);
});
```


[1]: https://developers.kameleoon.com/feature-management-and-experimentation/web-sdks/js-sdk
{{% /tab %}}
{{% tab "iOS" %}}

Kameleoon does not support this integration. Contact product@kameleoon.com to request this feature.

{{% /tab %}}
{{% tab "Android" %}}

Kameleoon does not support this integration. Contact product@kameleoon.com to request this feature.


{{% /tab %}}
{{% tab "Flutter" %}}

Kameleoon does not support this integration. Contact product@kameleoon.com to request this feature.


{{% /tab %}}
{{% tab "React Native" %}}

After creating and initializing the Kameleoon SDK, subscribe to the `Evaluation` event using the `onEvent` handler.

Learn more about SDK initialization in the [Kameleoon React Native SDK documentation][1].

```javascript
const { onEvent } = useInitialize();

onEvent(EventType.Evaluation, ({ featureKey, variation }) => {
  datadogRum.addFeatureFlagEvaluation(featureKey, variation.key);
});
```


[1]: https://developers.kameleoon.com/feature-management-and-experimentation/web-sdks/react-js-sdk
{{% /tab %}}
{{< /tabs >}}


### LaunchDarkly integration

{{< tabs >}}
{{% tab "Browser" %}}

Initialize LaunchDarkly's SDK and create an inspector reporting feature flags evaluations to Datadog using the snippet of code shown below.

 For more information about initializing LaunchDarkly's SDK, see [LaunchDarkly's JavaScript SDK documentation][1].

```javascript
const client = LDClient.initialize("<CLIENT_SIDE_ID>", "<CONTEXT>", {
  inspectors: [
    {
      type: "flag-used",
      name: "dd-inspector",
      method: (key: string, detail: LDClient.LDEvaluationDetail) => {
        datadogRum.addFeatureFlagEvaluation(key, detail.value);
      },
    },
  ],
});
```


[1]: https://docs.launchdarkly.com/sdk/client-side/javascript#initializing-the-client
{{% /tab %}}
{{% tab "iOS" %}}

LaunchDarkly does not support this integration. Create a ticket with LaunchDarkly to request this feature.


{{% /tab %}}
{{% tab "Android" %}}

LaunchDarkly does not support this integration. Create a ticket with LaunchDarkly to request this feature.


{{% /tab %}}
{{% tab "Flutter" %}}

LaunchDarkly does not support this integration. Create a ticket with LaunchDarkly to request this feature.


{{% /tab %}}
{{% tab "React Native" %}}

LaunchDarkly does not currently support this integration. Create a ticket with LaunchDarkly to request this feature.


{{% /tab %}}
{{< /tabs >}}


### Split Integration

{{< tabs >}}
{{% tab "Browser" %}}

Initialize Split's SDK and create an impression listener reporting feature flag evaluations to Datadog using the following snippet of code:

For more information about initializing Split's SDK, see Split's [JavaScript SDK documentation][1].

```javascript
const factory = SplitFactory({
    core: {
      authorizationKey: "<APP_KEY>",
      key: "<USER_ID>",
    },
    impressionListener: {
      logImpression(impressionData) {
          datadogRum
              .addFeatureFlagEvaluation(
                  impressionData.impression.feature,
                  impressionData.impression.treatment
              );
    },
  },
});

const client = factory.client();
```


[1]: https://help.split.io/hc/en-us/articles/360020448791-JavaScript-SDK#2-instantiate-the-sdk-and-create-a-new-split-client
{{% /tab %}}
{{% tab "iOS" %}}

Initialize Split's SDK and create an inspector reporting feature flag evaluations to Datadog using the snippet of code below.

For more information about initializing Split's SDK, see Split's [iOS SDK documentation][1].

```swift
  let config = SplitClientConfig()
  // Send the feature flag when Split reports the impression
  config.impressionListener = { impression in
      if let feature = impression.feature,
          let treatment = impression.treatment {
          RUMMonitor.shared().addFeatureFlagEvaluation(name: feature, value: treatment)
      }
  }
```


[1]: https://help.split.io/hc/en-us/articles/360020401491-iOS-SDK
{{% /tab %}}
{{% tab "Android" %}}

Initialize Split's SDK and create an inspector reporting feature flag evaluations to Datadog using the snippet of code below.

For more information about initializing Split's SDK, see Split's [Android SDK documentation][1].

```kotlin
  internal class DatadogSplitImpressionListener : ImpressionListener {
    override fun log(impression: Impression) {
        // Send the feature flag when Split reports the impression
        GlobalRumMonitor.get().addFeatureFlagEvaluation(
            impression.split(),
            impression.treatment()
        )
    }
    override fun close() {
    }
  }

  // In initialization:
  val apikey = BuildConfig.SPLIT_API_KEY
  val config = SplitClientConfig.builder()
      .impressionListener(DatadogSplitImpressionListener())
      .build()
```


[1]: https://help.split.io/hc/en-us/articles/360020343291-Android-SDK
{{% /tab %}}
{{% tab "Flutter" %}}

Initialize Split's SDK and create an inspector reporting feature flag evaluations to Datadog using the snippet of code below.

For more information about initializing Split's SDK, see Split's [Flutter plugin documentation][1].

```dart
  StreamSubscription<Impression> impressionsStream = _split.impressionsStream().listen((impression) {
    // Send the feature flag when Split reports the impression
    final split = impression.split;
    final treatment = impression.treatment;
    if (split != null && treatment != null) {
      DatadogSdk.instance.rum?.addFeatureFlagEvaluation(split, treatment);
    }
  });
```


[1]: https://help.split.io/hc/en-us/articles/8096158017165-Flutter-plugin
{{% /tab %}}
{{% tab "React Native" %}}

Initialize Split's SDK and create an impression listener reporting feature flag evaluations to Datadog using the following snippet of code:

For more information about initializing Split's SDK, see Split's [React Native SDK documentation][1].

```javascript
const factory = SplitFactory({
    core: {
      authorizationKey: "<APP_KEY>",
      key: "<USER_ID>",
    },
    impressionListener: {
      logImpression(impressionData) {
          DdRum
              .addFeatureFlagEvaluation(
                  impressionData.impression.feature,
                  impressionData.impression.treatment
              );
    },
  },
});

const client = factory.client();
```


[1]: https://help.split.io/hc/en-us/articles/4406066357901-React-Native-SDK#2-instantiate-the-sdk-and-create-a-new-split-client
{{% /tab %}}
{{< /tabs >}}

### Statsig Integration

{{< tabs >}}
{{% tab "Browser" %}}

Initialize Statsig's SDK with `statsig.initialize`.

1. Update your Browser RUM SDK version 4.25.0 or above.
2. Initialize the RUM SDK and configure the `enableExperimentalFeatures` initialization parameter with `["feature_flags"]`.
3. Initialize Statsig's SDK (`>= v4.34.0`) and implement the `gateEvaluationCallback` option as shown below:

   ```javascript
    await statsig.initialize('client-<STATSIG CLIENT KEY>',
    {userID: '<USER ID>'},
    {
        gateEvaluationCallback: (key, value) => {
            datadogRum.addFeatureFlagEvaluation(key, value);
        }
    }
    );
   ```

[1]: https://docs.statsig.com/client/jsClientSDK
{{% /tab %}}
{{% tab "iOS" %}}

Statsig does not support this integration. Contact support@statsig.com to request this feature.

{{% /tab %}}
{{% tab "Android" %}}

Statsig does not support this integration. Contact support@statsig.com to request this feature.

{{% /tab %}}
{{% tab "Flutter" %}}

Statsig does not support this integration. Contact support@statsig.com to request this feature.

{{% /tab %}}
{{% tab "React Native" %}}

Statsig does not currently support this integration. Contact support@statsig.com to request this feature.

{{% /tab %}}
{{< /tabs >}}

## Analyze your Feature Flag performance in RUM

Feature flags appear in the context of your RUM Sessions, Views, and Errors as a list.

{{< img src="real_user_monitoring/guide/setup-feature-flag-data-collection/feature-flag-list-rum-event.png" alt="Feature Flag list of attributes in RUM Explorer" style="width:75%;">}}

### Search feature flags using the RUM Explorer
Search through all the data collected by RUM in the [RUM Explorer][2] to surface trends on feature flags, analyze patterns with greater context, or export them into [dashboards][3] and [monitors][4]. You can search your Sessions, Views, or Errors in the RUM Explorer, with the `@feature_flags.{flag_name}` attribute.

#### Sessions
Filtering your **Sessions** with the `@feature_flags.{flag_name}` attribute, you can find all sessions in the given time frame where your feature flag was evaluated.

{{< img src="real_user_monitoring/guide/setup-feature-flag-data-collection/rum-explorer-session-feature-flag-search.png" alt="Search Sessions for Feature Flags in the RUM Explorer" style="width:75%;">}}

#### Views
Filtering your **Views** with the `@feature_flags.{flag_name}` attribute, you can find the specific views in the given time frame where your feature flag was evaluated.

{{< img src="real_user_monitoring/guide/setup-feature-flag-data-collection/rum-explorer-view-feature-flag-search.png" alt="Search Views for Feature Flags in the RUM Explorer" style="width:75%;">}}

#### Errors
Filtering your **Errors** with the `@feature_flags.{flag_name}` attribute, you can find all the errors in the given time frame that occurred on the View where your feature flag was evaluated

{{< img src="real_user_monitoring/guide/setup-feature-flag-data-collection/rum-explorer-error-feature-flag-search.png" alt="Search Errors for Feature Flags in the RUM Explorer" style="width:75%;">}}

## Troubleshooting

### My feature flag data doesn't reflect what I expect to see
Feature flags show up in the context of events where they are evaluated, meaning they should show up on the views that the feature flag code logic is run on.

Depending on how you've structured your code and set up your feature flags, you may see unexpected feature flags appear in the context of some events.

For example, to see what **Views** your feature flag is being evaluated on, you can use the RUM Explorer to make a similar query:

{{< img src="real_user_monitoring/guide/setup-feature-flag-data-collection/feature_flag_view_query.png" alt="Search Views for Feature Flags in the RUM Explorer" style="width:75%;">}}

Here are a few examples of reasons why your feature flag is being evaluated on unrelated Views that can help with your investigations:

- A common react component that appears on multiple pages which evaluates feature flags whenever they run.
- A routing issue where components with a feature flag evaluation are rendered before/after URL changes.

When performing your investigations, you can also scope your data for `View Name`'s that are relevant to your feature flag.


## Further Reading
{{< partial name="whats-next/whats-next.html" >}}

[1]: /real_user_monitoring/browser/setup/
[2]: https://app.datadoghq.com/rum/explorer
[3]: /dashboards/
[4]: /monitors/#create-monitors
[5]: /real_user_monitoring/feature_flag_tracking
