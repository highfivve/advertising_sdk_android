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
  - [3. Displaying a Banner Ad](#3-displaying-a-banner-ad)
  - [4. Handling Banner Ad Events](#4-handling-banner-ad-events)
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

- Android API Level 23 or later
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
    implementation("com.highfivve:advertising_android:0.0.3")
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

Initialize the `HighfivveAdManager` once, typically in your `Application` class. This should be done
before any ad requests are made.

```kotlin
import android.app.Application
import com.highfivve.advertising_android.manager.HighfivveAdManager

class YourApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        HighfivveAdManager.initialize(
            context = this,
            publisherCode = "YOUR_PUBLISHER_CODE", // Provided by Highfivve GmbH
            bundleName = "com.example.your_bundle_name", //your app bundle name
        )
    }
```

Remember to register `YourApplication` in your `AndroidManifest.xml`:

```xml
<application android:name=".YourApplication" ... > </application>
```

### 3. Displaying a Banner Ad

Use the `HighfivveBannerAdView` custom view to display banner ads in your layouts.

**XML Layout:**

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <!-- Other UI elements -->

    <com.highfivve.advertising_android.ad.banner.HighfivveBannerAdView
        android:id="@+id/highfivve_banner_ad" android:layout_width="wrap_content"

        android:layout_height="wrap_content" android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true" app:position="home_bottom_banner"
        app:pageType="article_list" app:showAd="true" />
    <!-- Initial visibility, can be controlled programmatically -->
```

**Kotlin/Java Code (in your Activity/Fragment):**
```kotlin
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import com.highfivve.advertising_android.ad.banner.HighfivveBannerAdView
import com.highfivve.advertising_android.ad.banner.HighfivveBannerAdListener
class MyActivity : AppCompatActivity() {
    private lateinit var bannerAdView: HighfivveBannerAdView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_my) // Your layout with HighfivveBannerAdView

        bannerAdView = findViewById(R.id.highfivve_banner_ad)

        // Set parameters if not set in XML or to override
        // bannerAdView.position = "home_bottom_banner_prog"
        // bannerAdView.pageType = "dashboard"

        // Set Listener (see next section)
        bannerAdView.listener = object : HighfivveBannerAdListener {
            override fun onAdLoaded(adView: HighfivveBannerAdView) {
                // Ad loaded successfully, view will be visible if showAd is true
                // You might adjust layout or visibility here if needed
                println("Banner ad loaded for position: ${adView.position}")
            }

            override fun onAdFailedToLoad(adView: HighfivveBannerAdView, error: HighfivveAdError) {
                println("Banner ad failed for position ${adView.position}. Error: ${error.message}")
            }

            override fun onAdClicked(adView: HighfivveBannerAdView) {
                println("Banner ad clicked for position: ${adView.position}")
            }

            override fun onAdImpression(adView: HighfivveBannerAdView) {
                println("Banner ad impression for position: ${adView.position}")
            }
        }

      // Load the ad 
      bannerAdView.loadAd()
    }

    override fun onDestroy() {
        bannerAdView.destroy() // Important: Release resources
        super.onDestroy()
    }
}
```

**`HighfivveBannerAdView` XML Attributes:**

* `app:position` (String, required): A unique identifier for this ad slot/position.
* `app:pageType` (String, optional): Contextual information about the page where the ad is
  displayed.
* `app:showAd` (boolean, optional): Controls initial visibility and loading behavior. Defaults to
  `true`.

You can also set these properties programmatically.

### 4. Handling Banner Ad Events

Implement the `HighfivveBannerAdListener` interface and assign it to your `HighfivveBannerAdView`
instance.

```kotlin
bannerAdView.listener = object : HighfivveBannerAdListener {
    override fun onAdLoaded(adView: HighfivveBannerAdView) {
        // Called when an ad is successfully loaded and ready to be displayed. 
    }

    override fun onAdFailedToLoad(adView: HighfivveBannerAdView, error: HighfivveAdError) {
        // Called when an ad request fails.
        // `error` object contains details about the failure.
    }

    override fun onAdClicked(adView: HighfivveBannerAdView) {
        // Called when the ad is clicked by the user.
    }

    override fun onAdImpression(adView: HighfivveBannerAdView) {
        // Called when the ad has been successfully displayed on screen (impression).
    }
} 
```

## Supported Ad Networks


## API Reference (Overview)


## Privacy

- This SDK [does/does not] collect any personally identifiable information (PII) directly.
- If this SDK bundles third-party ad network SDKs, ensure you comply with their data safety
  requirements and guide users to declare necessary permissions and data usage in their app's Google
  Play Data Safety section.
- List any specific permissions (beyond `INTERNET`) or Google Play services dependencies (like
  `play-services-ads-identifier`) if this SDK or its dependencies use them.
- Provide a link to your SDK's privacy policy if applicable.

## License

This SDK is released under the Apache 2.0 License. See
the [Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0) file for more details.

