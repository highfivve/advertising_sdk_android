# Changelog

All notable changes to the `advertising_android` SDK will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.0.8] - 2026-09-28

### Fixed

- A real, high-impact bug from the 0.0.7 next-gen SDK migration: Google's own ad request never
  received the user's consent/personalization decision. `ConsentManager`'s `npa`/`gdpr` extras
  were (and still are) correctly forwarded to Prebid/InMobi via `HeaderBiddingSdk
  .onConsentUpdated`, but neither `HighfivveBannerAd` nor `HighfivveInterstitialAd` ever applied
  that signal to Google's own `BannerAdRequest`/`AdRequest` - the next-gen SDK has no per-request
  consent API (unlike the legacy SDK's `addNetworkExtrasBundle`), only the global
  `MobileAds.setRequestConfiguration(...)`, and nothing ever called it with a consent-derived
  value. In Google Ad Manager reporting terms this meant the AdID was reported "Missing" (no
  identifier attached at all) rather than "Active"/"Restricted", on effectively every Android ad
  request regardless of the user's actual consent choice - not just the ones from users who
  declined tracking. `HighfivveAdvertising.updateConsent(...)` now also calls a new
  `applyRequestConfiguration()`, which sets
  `RequestConfiguration.PublisherPrivacyPersonalizationState`
  from the same `ConsentManager.shouldRequestNonPersonalizedOnly()` signal already used for the
  Prebid/InMobi path. `setTestDeviceIds(...)` now goes through the same function instead of
  building its own `RequestConfiguration` from scratch, since `MobileAds.setRequestConfiguration`
  replaces the whole configuration object rather than merging into it - the two call sites would
  otherwise silently clobber each other's setting.

## [0.0.7] - 2026-08-25

### Added

- Audit/debug mode: `HighfivveAdvertising.setAuditModeEnabled(Boolean)` swaps banner/interstitial
  ad requests to Google's public test ad unit IDs and enables Prebid Server debug echo + verbose
  Prebid SDK logging. `HighfivveAdvertising.getDebugSnapshot()` reports the SDK's actual internal
  state for QA/publisher diagnostics: config-fetch outcome/source/timestamp (new - previously
  computed transiently and discarded), the raw `app-config.json` last received, active
  header-bidding SDKs, the current consent snapshot, and a bounded log of recent ad lifecycle
  events (new - `HighfivveBannerAd`/`HighfivveInterstitialAd` record every event regardless of
  whether a listener is assigned, so this is visible without wiring one up first).
  `HighfivveAdvertising.openAdInspector()` wraps Google's native `MobileAds.openAdInspector(...)`;
  `setTestDeviceIds(List<String>)` registers this device as a Google Mobile Ads test device for
  physical-device testing.
- `HighfivveDebugAuditActivity`: a ready-to-use, self-contained audit/debug screen for native
  (non-Flutter) consumers of this SDK, showing everything `getDebugSnapshot()` exposes plus
  configured ad slots and the recent-events log. Launch with
  `HighfivveDebugAuditActivity.start(context)`; registered in this library's manifest so no
  consumer-side manifest changes are needed. Mirrors the Flutter plugin's
  `HighfivveDebugAuditView`/`HighfivveDebugAuditPage`.

### Changed

- Migrated from the legacy Google Mobile Ads SDK (`com.google.android.gms:play-services-ads`) to
  Google's next-gen SDK (`com.google.android.libraries.ads.mobile.sdk:ads-mobile-sdk`). Confirmed
  by decompiling the actual `1.4.0` artifact (not just docs) that Google Ad Manager support carries
  over closely: `AdManagerAdView`→`AdView`, `AdManagerAdRequest.Builder`→`BannerAdRequest.Builder`/
  `AdRequest.Builder`, `addCustomTargeting`→`putCustomTargeting`,
  `addNetworkExtrasBundle(AdMobAdapter::class.java, ...)`→
  `putAdSourceExtrasBundle(AdMobAdapter::class.java, ...)` (same bundled `AdMobAdapter` class,
  same `Bundle` shape - `ConsentManager`'s consent/npa extras logic needed no changes at all).
  `HighfivveInterstitialAd` uses next-gen's "single load" pattern (not the newer
  `InterstitialAdPreloader` queue API) to stay a mechanical port of the existing design.
  `MobileAds.initialize` now takes an `InitializationConfig` requiring the AdMob application ID
  programmatically; read from the `com.google.android.gms.ads.APPLICATION_ID` manifest meta-data
  tag consuming apps already declare, so no consumer-facing setup changes.
- Mediation adapters bumped to their next-gen-native versions:
  `com.google.ads.mediation:facebook:6.22.0.0` and `com.google.ads.mediation:inmobi:11.4.0.0`
  (both previously depended on the legacy SDK transitively; confirmed via decompilation that
  `InMobiConsent.updateGDPRConsent(JSONObject)` keeps the same signature on the new version).
- One real Kotlin/Java-interop surprise found only by compiling against the real artifact: `BannerAd
  .getAdSize()` doesn't resolve as the `ad.adSize` property-access sugar from this module (unclear
  root cause - possibly how this SDK's interfaces are compiled) - calling `ad.getAdSize()` directly
  works and is what's used in `HighfivveBannerAd`.

## [0.0.6] - 2026-07-15

### Added

- GDPR/GPP/CCPA consent support: `HighfivveAdvertising.updateConsent(ConsentInfo)`, `ConsentInfo`,
  `AdPersonalizationState`, and a new `BLOCKED_BY_CONSENT` ad event fired when an ad request is
  skipped due to the current consent state.
- `HighfivveBannerAd` now automatically reloads itself on an interval (client-side, so header
  bidding runs a fresh auction on every reload). Configurable via `isAutoRefreshEnabled` (default
  `true`) and `refreshIntervalMillis` (default 30s, coerced to a 10s minimum).
- `HighfivveInterstitialAd` now automatically preloads the next ad after the current one is
  dismissed. Configurable via `isAutoReloadEnabled` (default `true`) and `reloadDelayMillis`
  (default 0 = immediate).
- `consumer-rules.pro` published alongside the AAR so consuming apps' R8/ProGuard builds keep the
  kotlinx.serialization-generated serializers this SDK needs at runtime.

### Changed

- `com.google.android.gms:play-services-ads` is now declared as an `api` dependency instead of
  `implementation`, since its types (`AdManagerBannerView`, `AdSize`, etc.) are exposed on this
  SDK's own public API - consumers no longer need to guess/redeclare a compatible version
  themselves.
- `compileSdk` bumped to 36 (from 34). API 37 was tried but isn't buildable with our current
  Android Gradle Plugin (8.1.4): its platform SDK package uses a newer repository schema that this
  AGP version's bundled parser can't read (`Failed to find Platform SDK with path:
  platforms;android-37`, plus `package.xml parsing problem... unexpected element "abis"`) - needs
  an AGP upgrade first.

### Removed

- Unused `com.google.android.exoplayer:*` and `com.google.code.gson:gson` dependencies (neither was
  referenced anywhere in this SDK's source).

### Fixed

- `HighfivveBannerAd`: the Prebid-won creative size wasn't being reported to listeners - it computed
  the corrected size but notified `onLoadedAdSizeChanged` beforehand with the wrong (default GAM)
  size, so anything sizing itself off that callback (e.g. the Flutter plugin's banner widget) never
  saw the real ad size.
- `isLoading` on `HighfivveBannerAd`/`HighfivveInterstitialAd` no longer gets stuck `true` forever
  after an `AD_NOT_FOUND` event.
- `HighfivveInterstitialAd` now registers a `FullScreenContentCallback` - previously no callback was
  registered at all, so `AD_OPENED`, `AD_IMPRESSION`, `AD_CLICKED`, and `AD_CLOSED` events never
  fired for interstitials.

## [0.0.5] - 2025-10-21

### Changed
- updated README

### Added
- added support for inmobi sdk for google ads mediation

___

## [0.0.4] - 2025-09-02

### Fixed

- missingfieldexception when meta sdk is not integrated

___
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

