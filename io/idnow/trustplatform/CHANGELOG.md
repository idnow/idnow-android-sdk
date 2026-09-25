# Changelog

All notable changes to the TrustPlatform Android SDK will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
