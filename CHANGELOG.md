# Changelog

All notable changes to the `advertising_android` SDK will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
## [0.0.3] - 2025-09-02

### Changed

- added support for meta sdk for google ads mediation (currently not active)

___


## [0.0.2] - 2025-09-02

### Added

- new method to retrieve all available ad slots from the configuration:
  `HighfivveAdManager.getAvailableAdSlots(): List<AdSlot>`
- added support for meta sdk for google ads mediation

___

## [0.0.1] - 2025-07-18

### Added

- **Initial Release of the Highfivve Advertising Android SDK.**
- **SDK Core:**
    - `HighfivveAdManager`: Singleton for SDK-wide operations.
        - `initialize(context: Context, publisherCode: String, config: HighfivveConfig? = null)`:
          Method for initializing the SDK with application context, publisher code, and optional
          custom configurations.
        - Support for optional `HighfivveConfig` data class to provide local/default settings for ad
          behavior, slot definitions, and underlying SDK parameters (e.g., Prebid).
- **Banner Ads:**
    - `HighfivveBannerAdView`: Custom Android View for displaying banner ads.
        - Support for XML attributes (`app:position`, `app:pageType`, `app:showAd`) and programmatic
          configuration of ad position, page type, and initial visibility/loading.
        - `loadAd()`: Method to explicitly request a banner ad.
        - `destroy()`: Method to clean up resources and listeners, crucial for Activity/Fragment
          lifecycle management.
    - `HighfivveBannerAdListener`: Interface for banner ad lifecycle events:
        - `onAdLoaded(adView: HighfivveBannerAdView)`
        - `onAdFailedToLoad(adView: HighfivveBannerAdView, error: HighfivveAdError)`
        - `onAdClicked(adView: HighfivveBannerAdView)`
        - `onAdImpression(adView: HighfivveBannerAdView)`
- **Interstitial Ads:**
    -
    `HighfivveAdManager.loadInterstitialAd(activity: Activity, position: String, pageType: String? = null)`:
    Method to preload interstitial ads, requiring an Activity context.
    - `HighfivveAdManager.showInterstitialAd(activity: Activity, position: String? = null)`: Method
      to display a preloaded interstitial ad, requiring an Activity context. (Clarify if `position`
      is optional and its behavior if omitted).
    - `HighfivveAdManager.isInterstitialAdReady(position: String): Boolean` (If this method is
      implemented and public).
    - `HighfivveInterstitialAdListener`: Interface assignable to
      `HighfivveAdManager.interstitialAdListener` for interstitial ad lifecycle events:
        - `onAdLoaded(position: String)`
        - `onAdFailedToLoad(position: String, error: HighfivveAdError)`
        - `onAdShown(position: String)`
        - `onAdFailedToShow(position: String, error: HighfivveAdError)`
        - `onAdClicked(position: String)`
        - `onAdDismissed(position: String)`
- **Error Handling:**
    - `HighfivveAdError`: Data class providing details (`message`, optional `code`) about ad request
      or display failures.
- **Documentation & Examples:**
    - Initial `README.md` with installation, setup, and usage examples for banner and interstitial
      ads.
    - KDoc comments for public classes and methods.
    - (If applicable) Included a sample application module demonstrating SDK integration.
- **Build & Configuration:**
    - Published as an AAR to MavenCentral / [Your Repository].
    - Initial ProGuard/R8 guidance in README.

### Changed

- N/A (Initial Release)

### Deprecated

- N/A (Initial Release)

### Removed

- N/A (Initial Release)

### Fixed

- N/A (Initial Release)

### Security

- Added disclaimer regarding customer status with Highfivve GmbH for SDK usage.
- Noted requirements for `INTERNET` permission and considerations for Google Play Data Safety if
  mediating third-party SDKs.

