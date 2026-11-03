# Changelog

All notable changes to the TrustPlatform Android SDK will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.5.0]

### Changed

- **Breaking:** `minSdk` raised from 24 to 28 (Android 9.0), because the Governikus eID module bundled with `:trustplatform-docidv` requires API 28. Integrators targeting Android 7.0–8.1 must raise their app's `minSdk` or stay on the previous SDK version.
- **Breaking:** `TrustPlatformSession.registerForResult(activity, onEvent)` now takes a `(TrustPlatformEvent) -> Unit` handler and returns a `TrustPlatformLauncher` with `launch(config)` and `stop()`, instead of an `ActivityResultLauncher<TrustPlatformConfig>`.
- A session aborted on the server is now reported as `TrustPlatformEvent.Aborted`, and closing the SDK before the flow ends (for example with the back button) as `TrustPlatformEvent.Dismissed`.

### Added

- `:trustplatform-docidv` now bundles the IDnow DocIDV Governikus module (`io.idnow.docidv:eid-governikus`), so German eID (Personalausweis) steps handled natively open instead of failing with `MissingModuleException`.
- `TrustPlatformEvent.Progress(StepProgress)` reports what the current step is waiting for (`InProgress`, `EnduserInputNeeded`, `InManualReview`) whenever the value changes. Requires a Player version that sends `stepProgress` on `session-state`; with an older Player every step reports `InProgress`, so no progress events arrive.
- `TrustPlatformLauncher.stop()` closes the SDK without ending the session and without sending an event.
- `TrustPlatformLauncher.launch(config)` does nothing while a run of the same launcher is still active (also between `stop()` and the SDK closing), so a double tap cannot start a second run.

### Removed

- **Breaking:** `TrustPlatformResult`; use `TrustPlatformEvent`.
- **Breaking:** `TrustPlatformError.Cancelled`; use `TrustPlatformEvent.Aborted` or `TrustPlatformEvent.Dismissed`.

### Fixed

- The WebView was reloaded only when the session URL changed, so a step that wrote no session state left the screen frozen. The SDK now sends a `session-refresh` message to the Player on every poll and the Player re-renders in place instead of reloading, which also removes the white flash between steps. Requires a Player version that handles `session-refresh`; with an older Player the URL-change reload remains the fallback.

## [0.4.1]

### Changed

- The styling context injected into the WebView is now scheme-scoped: `backgroundColour` is a per-scheme map (`{ dark }`) instead of a flat hex. Requires a Player version that reads the new shape.

## [0.4.0]

### Added

- The SDK now sends device context (OS, device model, manufacturer, locale, timezone, network details) when acquiring a session.
- Added `ACCESS_NETWORK_STATE` permission, required for network capability detection.

### Changed

- Added dark mode support for the WebView.

## [0.3.0] - 2026-09-08

### Added

- `TrustPlatformError.NativeHandlerFailure` with a machine-readable error code.
- `TrustPlatformError.errorDescription(context)` extension for localized, user-facing error messages.

### Changed

- Updated IDnow DocIDV SDK from `1.8.0` to `1.12.0`.
- Updated Sunflower design system from `1.2.33` to `1.2.34`.
- Updated `litert-support-api` from `1.4.0` to `1.4.2` to align with the litert version used internally by the DocIDV SDK.
- Updated Koin from `4.2.1` to `4.2.2`, jmrtd from `0.8.6` to `0.8.8`, scuba-sc-android from `0.0.26` to `0.0.27`, and Android Gradle Plugin from `9.3.1` to `9.4.0`.

### Fixed

- Fixed `TrustPlatformEnvironment` production and sandbox player URLs to include the `/player` path component, matching the behaviour of the custom environment.

## [0.2.1-beta] - 2026-08-18

- CI pipeline validation release. No functional changes.

## [0.2.0-beta] - 2026-08-14

- CI pipeline validation release. No functional changes.

## [0.1.0] - 2026-08-13

### Added

- Initial release of the TrustPlatform Android SDK.
- `:trustplatform` core module — flow execution engine with WebView and native directive support.
- `:trustplatform-common` module — shared SPI types (NativeHandlerProvider, NativeDirective).
- `:trustplatform-docidv` module — DocIDV native handler integrating the IDnow DocIDV SDK.
- `:trustplatform-bom` — Bill of Materials for aligned dependency management.
