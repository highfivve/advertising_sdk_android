# Highfivve Advertising Android SDK

The Highfivve Advertising Android SDK enables the integration of Highfivve's advertising solutions
into native Android applications, allowing you to display various ad formats.

## ⚠️ Disclaimer: Official Highfivve GmbH Advertising SDK

This is the official Highfivve GmbH Advertising Software Development Kit (SDK).

**Important Usage Requirements:**

* **Customer Status:** To utilize this SDK and the Highfivve advertising services, you or your
  organization **must be an active and approved customer of Highfivve GmbH.**
* **Authorization:** Access to and use of our advertising platform through this SDK require prior
  authorization and agreement with Highfivve GmbH's terms of service.
* **Contact for Access:** If you are not yet a customer or wish to inquire about using our
  advertising services, please contact us to discuss your needs and begin the onboarding process.

**Contact Information:**

For new customer inquiries, SDK support, or any questions regarding the use of this SDK, please
reach out to us at:

**[team@highfivve.com]**

Using this SDK without being an authorized customer of Highfivve GmbH is a violation of our terms
and may result in a lack of service or functionality.

---

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
  - [Gradle](#gradle)
- [Getting Started](#getting-started)
  - [1. Google AdMob Integration](#1-google-adMob-integration)
  - [2. SDK Initialization](#2-sdk-initialization)
  - [3. Consent / Privacy (GDPR, GPP, CCPA)](#3-consent--privacy-gdpr-gpp-ccpa)
  - [4. Displaying a Banner Ad](#4-displaying-a-banner-ad)
  - [5. Handling Banner Ad Events](#5-handling-banner-ad-events)
  - [6. Displaying an Interstitial Ad](#6-displaying-an-interstitial-ad)
- [Supported Ad Networks](#supported-ad-networks)
- [API Reference (Overview)](#api-reference-overview)
- [Privacy](#privacy)
- [License](#license)

## Features

- Display Banner ad formats.
- Integration with Highfivve's ad serving platform.
- Listener interfaces for ad lifecycle events.
- Customizable ad loading and display options (via parameters like `position` and `pageType`).
- Designed for direct use in native Android projects (Kotlin/Java).

## Requirements

- Android API Level 23 or later (compiled against API 36 - API 37 isn't buildable yet with our
  current Android Gradle Plugin version, see the CHANGELOG)
- Android Gradle Plugin 8.0 or later
- Kotlin 1.7.0 or later / Java 8 or later
- AndroidX libraries

## Installation

### Gradle (Recommended)

1. Ensure you have `mavenCentral()` or your custom Maven repository (where this SDK is published) in
   your project's root `build.gradle(.kts)` or `settings.gradle(.kts)` file:

```gradle 
repositories { 
    google()
    mavenCentral()
 }
```

2. Add the dependency to your module's `build.gradle(.kts)` file (e.g., `app/build.gradle.kts`):

```kotlin
dependencies {
  implementation("com.highfivve:advertising_android:0.0.6")
}
```

2.1 If you need to include specific ad network SDKs, add them as dependencies as well. For example to include InMobi SDK:

```kotlin
dependencies {
    ...
    implementation("com.google.ads.mediation:inmobi:10.8.8.0")
}
```

3. Sync your project with Gradle files.

## Getting Started

### 1. Google AdMob Integration

To enable Google AdMob ads, add your AdMob App ID to your AndroidManifest.xml as shown below. This
is required for Google ad serving to work correctly.
Add the following inside the ```<application>``` tag in your
```android/app/src/main/AndroidManifest.xml```:

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-****************~**********"/>
```

Replace ```ca-app-pub-****************~**********``` with your actual AdMob App ID.

* **Finding your AdMob App ID:** You can find your App ID in the AdMob UI.
* **More Information:
  ** [Google Mobile Ads SDK Android - Get Started](https://developers.google.com/admob/android/quick-start#update_your_androidmanifestxml)

### 2. SDK Initialization

Initialize the `HighfivveAdvertising` singleton once, typically in your `Application` class. This
should be done before any ad requests are made.

```kotlin
import android.app.Application
import com.highfivve.advertising_android.HighfivveAdvertising

class YourApplication : Application() {
    override fun onCreate() {
        super.onCreate()
      HighfivveAdvertising.getInstance().initialize(
            context = this,
            publisherCode = "YOUR_PUBLISHER_CODE", // Provided by Highfivve GmbH
            bundleName = "com.example.your_bundle_name", //your app bundle name
        )
    }
}
```

Remember to register `YourApplication` in your `AndroidManifest.xml`:

```xml
<application android:name=".YourApplication" ... > </application>
```

### 3. Consent / Privacy (GDPR, GPP, CCPA)

The SDK works with **any** Consent Management Platform (CMP) - it doesn't assume a specific one.

**Zero-integration (recommended default):** if your CMP is IAB TCF/GPP-compliant, it already writes
consent to the standard `IABTCF_TCString`/`IABTCF_gdprApplies`/`IABGPP_HDR_GppString`/
`IABGPP_GppSID` keys in `SharedPreferences`. Prebid and Google Ad Manager read these directly - you
don't have to call anything, just make sure your CMP flow has run before ads are requested.

**Explicit:** call `updateConsent` when you want to be explicit, your CMP doesn't write the standard
keys, or to supply signals the standard keys don't cover (e.g. Meta Audience Network's Additional
Consent string):

```kotlin
import com.highfivve.advertising_android.HighfivveAdvertising
import com.highfivve.advertising_android.consent.AdPersonalizationState
import com.highfivve.advertising_android.consent.ConsentInfo

HighfivveAdvertising.getInstance().updateConsent(
  ConsentInfo(
    personalizationState = AdPersonalizationState.PERSONALIZED_ALLOWED, // or NON_PERSONALIZED_ONLY / ADS_DISALLOWED / UNKNOWN
    gdprApplies = true,
    tcString = myCmp.tcString,
    gppString = myCmp.gppString,
    gppApplicableSections = myCmp.gppApplicableSections,
    acString = myCmp.additionalConsentString, // needed for Meta Audience Network
  )
)
```

Call this again whenever consent changes, not just once at startup. Every field is independent and
additive - leave a field unset if you don't have that signal, and the SDK only overrides a given ad
network's own implicit consent detection when the corresponding field is non-null.

### 4. Displaying a Banner Ad

Use the `HighfivveBannerAd` custom view to display banner ads in your layouts.

**XML Layout:**

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <!-- Other UI elements -->

  <com.highfivve.advertising_android.ad.banner.HighfivveBannerAd
        android:id="@+id/highfivve_banner_ad" android:layout_width="wrap_content"
        android:layout_height="wrap_content" android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true" app:position="home_bottom_banner"
          app:pageType="article_list" />
</RelativeLayout>
```

**Kotlin Code (in your Activity/Fragment):**
```kotlin
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import com.highfivve.advertising_android.ad.banner.HighfivveBannerAd
import com.highfivve.advertising_android.listener.AdEventListener
import com.highfivve.advertising_android.util.HighfivveAdEvent

class MyActivity : AppCompatActivity() {
  private lateinit var bannerAd: HighfivveBannerAd

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
      setContentView(R.layout.activity_my) // Your layout with HighfivveBannerAd

      bannerAd = findViewById(R.id.highfivve_banner_ad)

        // Set parameters if not set in XML or to override
      // bannerAd.position = "home_bottom_banner_prog"
      // bannerAd.pageType = "dashboard"

      // Optional: configure automatic reload (see below for defaults)
      // bannerAd.isAutoRefreshEnabled = true
      // bannerAd.refreshIntervalMillis = 45_000L

      // Set listener (see next section)
      bannerAd.adEventListener = object : AdEventListener {
        override fun onAdEvent(event: HighfivveAdEvent, data: Map<String, Any?>?) {
          when (event) {
            HighfivveAdEvent.AD_LOADED -> println("Banner ad loaded: $data")
            HighfivveAdEvent.AD_FAILED_TO_LOAD -> println("Banner ad failed: $data")
            HighfivveAdEvent.AD_CLICKED -> println("Banner ad clicked")
            HighfivveAdEvent.AD_IMPRESSION -> println("Banner ad impression recorded")
            else -> Unit
          }
            }
        }

      // Load the ad
      bannerAd.loadAd()
    }
}
```

The view automatically cancels its own pending reload when detached from the window, so no manual
cleanup is required.

**`HighfivveBannerAd` XML Attributes:**

* `app:position` (String, required): A unique identifier for this ad slot/position.
* `app:pageType` (String, optional): Contextual information about the page where the ad is
  displayed.

**Automatic banner refresh:** the banner reloads itself on an interval by default (a client-side
reload is required so header bidding runs a fresh auction on every reload, rather than relying on
Google Ad Manager's own server-side refresh). Configure this via:

* `isAutoRefreshEnabled: Boolean` - defaults to `true`. Disabling it cancels any pending reload
  immediately.
* `refreshIntervalMillis: Long` - defaults to 30 seconds. Coerced to a minimum of 10 seconds to
  guard against runaway reload loops.

### 5. Handling Banner Ad Events

Implement the `AdEventListener` interface and assign it to your `HighfivveBannerAd`
instance's `adEventListener` property. `HighfivveAdEvent` covers the full list of event types:
`AD_LOADED`, `AD_FAILED_TO_LOAD`, `AD_OPENED`, `AD_CLOSED`, `AD_CLICKED`, `AD_IMPRESSION`,
`AD_NOT_FOUND`, `DISABLED_ALL`, `BLOCKED_BY_CONSENT`.

### 6. Displaying an Interstitial Ad

Interstitial ads are managed with the `HighfivveInterstitialAd` class rather than a view - construct
one per placement, load it ahead of time, and show it when appropriate:

```kotlin
import com.highfivve.advertising_android.ad.interstitial.HighfivveInterstitialAd
import com.highfivve.advertising_android.listener.AdEventListener
import com.highfivve.advertising_android.util.HighfivveAdEvent

val interstitialAd = HighfivveInterstitialAd(context, position = "interstitial_main")
interstitialAd.pageType = "article_list" // optional

// Optional: configure automatic preloading of the next ad after the current one is dismissed
// interstitialAd.isAutoReloadEnabled = true // default
// interstitialAd.reloadDelayMillis = 0 // default: preload immediately after dismissal

interstitialAd.adEventListener = object : AdEventListener {
  override fun onAdEvent(event: HighfivveAdEvent, data: Map<String, Any?>?) {
    if (event == HighfivveAdEvent.AD_LOADED) {
      interstitialAd.show(activity)
    }
    }
}

interstitialAd.loadAd()
```

By default, the SDK automatically preloads the next interstitial as soon as the current one is
dismissed (`isAutoReloadEnabled = true`, `reloadDelayMillis = 0`) - disable it or add a delay if you
want more control over when the next ad request happens.

## Supported Ad Networks

- **Prebid Mobile** - the SDK's real-time header bidding partner, enabled by default via remote
  configuration.
- **InMobi** and **Meta Audience Network** - can be enabled as Google Ad Manager mediation partners
  via remote configuration and their respective (optional) mediation adapter dependencies. Add the
  adapter dependency yourself as shown in [Installation](#installation) step 2.1 if you need one of
  these.

## API Reference (Overview)

Full KDoc-style documentation lives in the source under `src/main/kotlin` - the public entry points
are `HighfivveAdvertising` (SDK initialization and consent), `HighfivveBannerAd` (banner ads),
`HighfivveInterstitialAd` (interstitial ads), and the shared `AdEventListener`/`HighfivveAdEvent`
types. Browse the source on
[GitHub](https://github.com/highfivve/advertising_sdk_android) for full class/method documentation.

## Privacy

- This SDK does not collect personally identifiable information (PII) directly itself, but it
  forwards device/advertising identifiers and consent signals to the ad networks it mediates
  (Google Ad Manager, Prebid, and any optional network you enable) so they can serve and measure
  ads - see [Consent / Privacy](#3-consent--privacy-gdpr-gpp-ccpa) above for how to control this.
- If you enable optional ad network SDKs (InMobi, Meta), ensure you comply with their own data
  safety requirements and declare the relevant data usage in your app's Google Play Data Safety
  section.
- Requires the `INTERNET` permission (declared by this SDK's manifest) and
  `com.google.android.gms:play-services-ads` (declared as an `api` dependency, see
  [Installation](#installation)).

## License

This SDK is released under the Apache 2.0 License. See
the [Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0) file for more details.

