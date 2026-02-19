# IDnow - eID SDK Changelog - Android

All notable changes to this project will be documented in this file.

## [1.1.2] - 2026-02-19
### Fixed
- Remove logs.

## [1.1.1] - 2026-01-12
### Fixed
- TLS Pinning.

## [1.1.0] - 2025-11-14
### Added
- Accessibility.
- Implemented a TLS pinning check for API calls.

## [1.0.5] - 2025-09-05
### Fixed
- General bug fixes and performance improvements.

## [1.0.4] - 2025-08-08
### Fixed
- Some lotties not showing with dynamic feature.

## [1.0.3] - 2025-08-08
### Added
- Support to dynamic feature.

## [1.0.2] - 2025-07-31
### Added
- A new Lottie animation for the NFC scanning screen.

### Fixed
- Russian translations
- General bug fixes and performance improvements.

## [1.0.1] - 2025-06-03
### Fixed
- General bug fixes and performance improvements.

## [1.0.0] - 2025-05-21

We're thrilled to introduce **Version 1.0.0** of the eID Android SDK!
This release marks a major overhaul of the architecture and functionalities, aiming to deliver a smoother, more performant and robust german ID card authenticate flow.

### Major Features
* **Refactored Architecture:**
    * **New architecture:** The SDK has been completely re-architected and redesigned to simplify the authentication flow.
    * **Modernized Dependencies:** Updated to the new versions of Android frameworks.
    * **Optimized Codebase:** A complete Kotlin code refactoring for improved readability, maintainability, and scalability. 
    * **Enhanced Error Handling:** Implemented a clearer EIDError handling system to let you easily manage the end of the session.

* **Customization**
    * **Design System:** By using Sunflower, the SDK is fully customizable in terms of colors, spacing, radius or fonts to let you apply your own app theme. It also create uniformity
    * **Font:** We let your customize the font family for heading and content.
    * **Documentation:** New, detailed documentation has been done for this customization. It helps integrating our solution with theme management.

### Key Functionalities
* **5 pin authentication:** Create your own 6-digit PIN code through our 5-step guided process, then use it for authentication 
* **6 pin authentication:** Securely authenticate using your existing 6-digit PIN code.
* **Can pin authentication:** Card Access Number (CAN) support in case of 5/6 pin errors.
* **External card unblocking:** We redirect you to Ausweis app in case you blocked (or already blocked) you card.

### Fixes
/

### Next Steps
We'll continue to improve eID with regular updates and fixes. Stay tuned for upcoming releases!