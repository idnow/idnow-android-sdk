## IDnow - DocIDV SDK Changelog - Android

## [1.6.0] - 2026-03-25
### Added
- Modular Architecture 
    - The SDK has been restructured into independent modules
    - io.idnow.docidv:core — New base module, required by all other modules.
    - io.idnow.docidv:ai — Contains the existing SDK functionality (document capture, biometric liveness, identity verification).
    - io.idnow.docidv:eid-governikus(new, optional) — German eID chip reading via Governikus (requires minSdk 28).
- BOM Support
    - Added Bill of Materials (io.idnow.docidv:bom) for simplified dependency version management.
### Notes    
- The public API is unchanged — no code changes required on the integration side. 
- Replace the single io.idnow.docidv.android.sdk:docidv dependency with core + ai (minimum setup).
- It is also possible to keep io.idnow.docidv.android.sdk:docidv for now but it will be removed in a future release.


## [1.3.0] - 2026-02-12
### Fixes
- Country names are now translated into the set language correctly during the country selection on OTP screen
- Bugfixes and technical improvements.
## [1.2.0] - 2026-01-22
### Added
- Accessibility Enhancements:
    - Further progress toward full WCAG 2.1 compliance.
    - More components updated for improved accessibility.
    - Liveness now accessible across multiple backend vendor options.
- Design & Customization
    - Configurable design themes tailored to customer requirements.
    - Support for light and dark mode.
    - Custom Lottie animations now supported.
    - 
### Fixes:
- Functional and accessibility bug fixes • Stability enhancements • Overall UX refinements

## [1.1.0] - 2025-11-18
### Added
- Screen redesign to comply with Android accessibility guidelines.
- Support for light and dark mode.
- Adjustable text size and spacing (honors system preferences).
- Improved compatibility with TalkBack and screen readers (labels, focus order, actions).
- Enhanced color contrast for better readability.
- Clear announcements for component states (error, selected, disabled, progress).
- Visible and consistent focus indicators.
- Proper content descriptions for images and icons.

### Fixes:
- Optimized memory usage and app responsiveness.
- Several bugfixes to improve user experience.

## [1.0.2] - 2025-08-13
### Added
- Biometric fixes and 16kb support

## [1.0.0] - 2025-06-24
### Added
- This is an initial version of DocIDV SDK with AutoIdent flow support