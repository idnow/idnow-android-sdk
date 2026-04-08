# DOCIDV SDK - Android Documentation

[![Platform](https://img.shields.io/badge/Platform-Android-brightgreen.svg)](https://www.android.com/) [![Kotlin](https://img.shields.io/badge/Kotlin-2.3-blue.svg)](https://kotlinlang.org) [![API](https://img.shields.io/badge/API-24%2B-blueviolet.svg)](https://developer.android.com/tools/releases/platforms)

# Table of Contents

* [Overview](#overview)
  * [Requirements](#requirements)
  * [Compatibility, End of Support, End of Life](#compatibility-matrix)
* [Modules Overview](#modules-overview)
  * [Dependency Graph](#dependency-graph)
  * [Core Module](#core-module)
  * [AI Module](#ai-module)
  * [eID Governikus Module](#eid-governikus-module)
* [Integration](#integration)
  * [Repository Setup](#repository-setup)
  * [Dependencies Configuration](#dependencies-configuration)
* [Usage Example](#usage-example)
* [Integrate the NFC Dependency](#integrate-the-nfc-dependency)
* [SDK Error Codes](#sdk-error-codes)
  * [How to Deal with Errors](#how-to-deal-with-errors)

---

## Overview

DOCIDV SDK is a modular Android SDK for identity verification. Starting with version **1.6.0**, the SDK has been restructured into independent modules, giving you the flexibility to include only the features you need.

### Requirements

* **minSdkVersion:** 24 (Android 7 Nougat)
* **compileSdkVersion:** 36 (Android 16)
* **targetSdkVersion:** 36 (Android 16)
* **Not supported:** devices and emulators based on the x86 architecture

### Compatibility Matrix

Please refer to the following link to find information about current versions, compatibility, end-of-support (EOS) and end-of-life (EOL) dates pertaining to our products: [IDnow Compatibility Matrix: Browser & OS Compatibility guide](https://www.idnow.io/developers/compatibility-overview/)

---

## Modules Overview

The SDK is composed of the following modules:

* **Core** — `io.idnow.docidv:core` — Required — Min SDK 24 — Base module, required by all other modules.

* **AI** — `io.idnow.docidv:ai` — Required\* — Min SDK 24 — Document capture, biometric liveness and identity verification. Corresponds to the existing SDK functionality.

* **eID Governikus** — `io.idnow.docidv:eid-governikus` — Optional — Min SDK 28 — German eID chip reading via Governikus.

> \*In this version, **Core + AI** are both required as a minimum setup. In a future release, AI will become optional as well.

> **Note:** The eID Governikus module raises the minimum SDK level to **28** (Android 9 Pie) due to a restriction from the Governikus SDK.

### Dependency Graph

```
┌──────────────────────────┐
│          Core            │  ← Required by all modules
└──────┬──────────┬────────┘
       │          │
       ▼          ▼
┌──────────┐  ┌───────────────────┐
│    AI    │  │  eID Governikus   │
│(required)│  │    (optional)     │
└──────────┘  │ depends on Core   │
              │      + AI         │
              └───────────────────┘
```

### Core Module

**Artifact:** `io.idnow.docidv:core`

The Core module is the foundation of the DOCIDV SDK. It provides the base infrastructure, public API interfaces and shared utilities used by all other modules. Every integration must include this module.

* Minimum SDK: **24**
* Always required

### AI Module

**Artifact:** `io.idnow.docidv:ai`

The AI module provides the main identity verification capabilities including document capture, biometric liveness detection and the overall identification flow. This module corresponds to the existing SDK functionality.

* Minimum SDK: **24**
* Required in this version (will become optional in a future release)

### eID Governikus Module

**Artifact:** `io.idnow.docidv:eid-governikus`

The eID Governikus module enables reading the chip of German electronic identity documents (eID) using the Governikus SDK. This module is entirely optional and can be added when German eID verification is needed.

* Minimum SDK: **28** (due to a Governikus SDK restriction)
* Optional
* Currently depends on both **Core** and **AI**. In a future release, it will depend only on Core.

---

## Integration

### Repository Setup

In your project's `settings.gradle` (KTS), besides any other repositories you already have, include this maven repository:

```
repositories {
    google()
    mavenCentral()

    // Add this repo containing the DOCIDV library.
    maven {
        url = uri("https://raw.githubusercontent.com/idnow/idnow-android-sdk/main")
    }
}
```

### Dependencies Configuration

The DOCIDV SDK provides a BOM to simplify dependency management. By importing the BOM, you no longer need to specify individual module versions — they are all aligned automatically.

In `libs.versions.toml`:

```
[versions]
docidv = "1.6.0"

[libraries]
idnow-docidv-bom = { group = "io.idnow.docidv", name = "bom", version.ref = "docidv" }
idnow-docidv-core = { group = "io.idnow.docidv", name = "core" }
idnow-docidv-ai = { group = "io.idnow.docidv", name = "ai" }
idnow-docidv-eid-governikus = { group = "io.idnow.docidv", name = "eid-governikus" }
```

In your app's `build.gradle`:

```
dependencies {
    // Import the BOM
    implementation(platform(libs.idnow.docidv.bom))

    // No need to specify versions — managed by the BOM
    implementation(libs.idnow.docidv.core)
    implementation(libs.idnow.docidv.ai)

    // Optional - German eID (raises minSdk to 28)
    implementation(libs.idnow.docidv.eid.governikus)
}
```

Alternatively, without version catalog:

```
dependencies {
    // Import the BOM
    implementation platform('io.idnow.docidv:bom:1.6.0')

    // No need to specify versions
    implementation 'io.idnow.docidv:core'
    implementation 'io.idnow.docidv:ai'

    // Optional - German eID (raises minSdk to 28)
    implementation 'io.idnow.docidv:eid-governikus'
}
```

> **Note:** The old docidv-sdk repository will still contains a package version with both ai and core plugin. It will be removed in a future release but can still be used for now.

---

## Usage Example

The public API remains unchanged. Initialize the SDK and start an identification as follows:

```
/**
 * Initialize the IDnow SDK
 */
private fun initSDK() {
    val idnowConfig = IDnowDocIDVConfig.Builder()
        .withLanguage("en") // set specific ISO language code
        .withHttpLogging(true)
        .build()
    IDnowDocIDV.getInstance().initialize(this, idnowConfig)
}

/**
 * Call this method after your backend generates the identToken
 */
private fun startIdentification(identToken: String) {
    IDnowDocIDV.getInstance().start(identToken) { iDnowDocIDVResult ->
        when (iDnowDocIDVResult.resultType) {
            IDnowDocIDVResult.ResultType.FINISHED -> {
                // Ident completed successfully
                handleIdentificationSuccess()
            }

            IDnowDocIDVResult.ResultType.CANCELLED -> {
                // Ident was canceled
                handleIdentificationCanceled(cancelReason = iDnowDocIDVResult.statusCode)
            }

            IDnowDocIDVResult.ResultType.ERROR -> {
                // Ident has an error
                handleIdentificationError(errorCode = iDnowDocIDVResult.statusCode)
            }
        }
    }
}
```

### Language Configuration

Using `withLanguage("lang_code")` you can configure the IDnow library to use a specific language. These ISO 639-1 language codes are currently supported: bg (Bulgarian), cs (Czech), da (Danish), de (German), el (Greek), en (English), es (Spanish), et (Estonian), fi (Finnish), fr (French), hr (Croatian), hu (Hungarian), it (Italian), ja (Japanese), ka (Georgian), ko (Korean), lt (Lithuanian), lv (Latvian), nb (Norwegian), nl (Dutch), pl (Polish), pt (Portuguese), ro (Romanian), ru (Russian), sk (Slovak), sl (Slovenian), sr (Serbian), sv (Swedish), tr (Turkish), zh (Chinese).

> **Best Practice:** Our recommended best practice for optimal user experience is to allow the SDK to use the device language instead of setting a specific lang_code.

> **Language Limitations:** Some components that rely on 3rd party technologies such as Biometric Liveness or NFC scanning do not support dynamic localization and use the device language.

---

## Integrate the NFC Dependency

The SDK also supports NFC scanning to read data from the chip on e-documents. The sdk is available in 2 variant, with NFC feature and without NFC, providing the best flexibility for different use cases of our customers. We recommend using the NFC feature for the best security, fraud prevention and user experience.

* To use the NFC feature, please reach out to your IDnow contact or IDnow Support ([support@idnow.io](mailto:support@idnow.io)) and obtain the required NFC dependencies. For information about the integration and usage, please refer to the section: [Integrate the NFC Dependency](#integrate-the-nfc-dependency)

After contacting Customer Support with a request for the NFC package, you will receive an archive to integrate into your application. This archive is compatible with the corresponding SDK version and onwards, until a new or updated version of the NFC library is released. (For example, if the name of the archive contains the keywords `for AI SDK v5.13.0`, it means that this NFC library is compatible with AI SDK v5.13.0+ onwards, until an updated version of the NFC library is released).

The contents of the received archive will contain:

* Detailed instructions file on how to set up the NFC dependency + fixes on potential build issues which you might encounter based on your local setup

* AAR files for NFC dependency to include in your app

**Overview of the NFC integration:**

1. Copy all the `.aar` files from the given archive's `libs` folder into your own app's `libs` folder.

2. Make sure to compile the `libs` folder. In your `app/build.gradle` file, add the below:

```
implementation fileTree(include: ['*.aar'], dir: 'libs')
// or if you have also jar files, use this line
// implementation fileTree(include: ['*.jar', '*.aar'], dir: 'libs')
```

1. In your project level `build.gradle` file, make sure to have this:

```
repositories {
    //...
    flatDir {
        dirs 'libs'
    }
    //...
}
```

1. The above AAR files require some transitive dependencies, which you need to add manually in your `app/build.gradle` file:

```
// ReadID Dependencies
// iddoc connector
implementation 'com.google.code.gson:gson:2.11.0'
implementation 'androidx.annotation:annotation:1.5.0'
implementation 'org.bouncycastle:bcpkix-jdk18on:1.78.1'
// readid mrz
implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.8.6'
implementation('com.google.android.gms:play-services-tflite-java:16.1.0') {
    exclude group: "org.tensorflow"
}
// readid core
implementation 'org.jetbrains.kotlin:kotlin-stdlib:1.9.24'
implementation 'net.sf.scuba:scuba-sc-android:0.0.26'
implementation 'net.sf.scuba:scuba-smartcards:0.0.20'
implementation 'edu.ucar:jj2000:5.2'
implementation("org.ejbca.cvc:cert-cvc:1.4.13") {
    exclude group: 'org.bouncycastle', module: 'bcprov-jdk15on'
}
implementation 'org.jmrtd:jmrtd:0.7.42'
// readid ui core
implementation 'androidx.fragment:fragment:1.8.1'
implementation 'androidx.preference:preference-ktx:1.2.1'
implementation 'com.google.android.material:material:1.12.0'
implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
implementation 'androidx.databinding:viewbinding:8.5.2'
// viz capture
implementation "com.google.mlkit:barcode-scanning:17.3.0"
implementation "com.google.mlkit:face-detection:16.1.7"
```

1. Add the following ProGuard rules in your application's `proguard-rules.pro` file:

```
# ReadID
-keep,includedescriptorclasses class com.readid.** { *; }
-keep enum nl.innovalor.logging.data.** { *; }
-keep enum nl.innovalor.logging.data.**$** { *; }
-keepclassmembers class nl.innovalor.logging.data.** { <fields>; }
-keep,includedescriptorclasses class net.sf.scuba.smartcards.IsoDepCardService { public <init>(***); }
-keep,includedescriptorclasses class nl.innovalor.cert.** { public <init>(***); }
-keep,includedescriptorclasses class org.bouncycastle.** { *; }
-keepclassmembers class * extends net.sf.scuba.data.Country { *; }
-keep,includedescriptorclasses class nl.innovalor.mrtd.model.** { private <fields>; <methods>; }
-dontwarn com.google.gson.**
-dontwarn javax.naming.**
-dontwarn javax.naming.directory.*
-dontwarn jj2000.disp.*
-dontwarn jj2000.j2k.decoder.*
-dontwarn org.jmrtd.imageio.*
-dontwarn org.jmrtd.jj2000.**
-dontwarn org.jnbis.imageio.*
```

---

## SDK Error Codes

In case of `IDnowDocIDVResult.ResultType.ERROR`, the `IDnowDocIDVResult.getStatusCode()` method returns one of the error codes below:

* **E100** — Ident code syntax incorrect

* **E101** — Ident code not found

* **E102** — Ident code expired

* **E103** — Ident code already completed

* **E130** — Get ident resources failed; invalid response

* **E131** — Get ident resources failed; server reachability

* **E150** — Start ident failed; invalid response

* **E151** — Start ident failed; server reachability

* **E152** — Start ident failed; missing session key

* **E160** — Process force closed

* **E170** — Socket connection force closed

* **E171** — Process force closed

* **E180** — Missing application context

### How to Deal with Errors

* **E102** — It is recommended to create another ident, and restart the process with the new ident code.

* **E103** — It is recommended to show a screen to the user with the message that they have submitted all info needed and that they should wait for the final result.

* **E170** — It is recommended to notify the user that the ident process timed out or was started on a different device and ask them to try again.

* **E180** — This is to alert the host app if the context has been lost (OS restarted/killed SDK process).

* **All other error codes** — It is recommended to show a generic error for the user and ask them to try again by restarting the process.