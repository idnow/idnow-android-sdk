# eID - IDnow

[![Platform](https://img.shields.io/badge/Platform-Android-brightgreen.svg)](https://www.android.com/)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.0-blue.svg)](https://kotlinlang.org)
[![API](https://img.shields.io/badge/API-23%2B-blueviolet.svg)](https://developer.android.com/tools/releases/platforms?hl#6.0)

## Table of Contents

- [About](#about)
- [Key Features](#key-features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Integration](#integration)
    - [Starting the SDK](#starting-the-sdk)
    - [Handle Result](#handle-result)
    - [Error Description](#error-description)
- [Customization](#customization)
- [Sample Application](#sample-application)

## About

The German government introduced RFID chip-based electronic ID cards in November 2010. 
The eID Android SDK is a library designed to authenticate with these German ID cards. The flow requires the personal PIN code and uses the NFC smartphone chip.

## Key Features

* **6-digit PIN flow**: Securely authenticate using your existing 6-digit PIN code.
* **5-digit PIN flow**: Create your own 6-digit PIN code through our 5-step guided process, then use it for authentication.
* **CAN PIN**: Card Access Number (CAN) support in case of error.

## Requirements

* **Android Studio:** Recommended IDE for development.
* **Android SDK:** Minimum API level 23 (Android 6 Marshmallow) or higher.
* **Kotlin:** 2.0.0.
* **Authada library:** Provider used to scan the chip.
* **NFC:** NFC-enabled smartphone. If available but not enabled on the user device settings, a permission screen is displayed.

## Installation

To install the eID SDK, open your app and add the following dependencies:

1. Add the repositories.

In `settings.gradle.kts` file, add these 2 maven directories needed.
Authada repository needs the credentials given to you by our CSM team.

```gradle
repositories {
    google()
    mavenCentral()

    // Add this 1st repo containing the eID's framework.
    maven {
         url = uri("https://raw.githubusercontent.com/idnow/idnow-android-sdk/main")
    }
    // Add this 2nd repo containing the Authada's provider. 
    maven {
        url = uri("https://repo.authada.de/public/")
        authentication {
            create<BasicAuthentication>("basic")
        }
        credentials {
            username = "your_authada_username"
            password = "your_authada_pwd"
        }
    }
}
```

2. Add the eID SDK dependency.

There are 2 methods to add the sdk's dependency:

- If you are using groovy, in `build.gradle`, add this dependency
```gradle
dependencies {
    implementation 'io.idnow.eid.android:eid-sdk:x.y.z'
}
```

- If you are using .kts, open `libs.versions.toml` and add the eid version:

```kotlin
eid = "x.y.z"
```
and define the library by adding this line in the same file : 
```kotlin
eid = { group = "io.idnow.eid.android", name = "eid-sdk", version.ref = "eid" }
```
and then add the created dependency in `build.gradle.kts` : 
```gradle
dependencies {
    implementation(libs.eid)
}
````

👏 You have now access to eID SDK, lets see how to work with it.

## Integration
### Starting the SDK

Here is an implementation example of launching the eID SDK from a host app:
```kotlin
fun startEidActivity(activity: Activity, token: String) {
    val eidConfig = EIDConfig.Builder().build()
    
    EIDSdk.start(
        activity = activity, 
        token = token, 
        callback = this, 
        config = eidConfig
    )
}
```
This code calls the main `start` method to launch the eID library. It takes several parameters:

| Parameters | Type          | Description |
| ---------- | ------------- | ----------- |
| activity   | `Activity`    | The client host app activity that launches the SDK. |
| token      | `String`      | The Ident token provided. |
| callback   | `EIDCallback` | Interface to listen to for receiving the result of the eID session. |
| config     | `EIDConfig`   | Object used to configure the SDK (see [Customization](#customization) section). |

🎉 The eID SDK can now be launched from your host app. Now let's see how to customize it.

### Handle Result
The eID SDK doesn't provide any live events, only a session result: success or error with the error type returned.
For that, implement our listener `EIDCallback` by overriding our 2 methods:
```kotlin
override fun onSuccess() {
    // Handle session success.
}

override fun onFailure(eidError: EIDError) {
    // Handle the returned session error.
    when(eidError) {
        ...
    }
}
```
### Error Description
When the SDK stops with an error, you have access to several types of errors from the `onFailure` method. You can easily work with these errors to display any content in your application. Here is the list of available `EIDError` types with their descriptions:
```kotlin
sealed class EIDError {
    /** Session has been cancelled by user. */
    data class Aborted(val reason: String? = null) : EIDError()

    /** Any API network error. */
    data object NetworkError : EIDError()

    /** NFC is not available on the device. */
    data object NfcNotAvailable : EIDError()

    /** Given token is not correct. */
    data object InvalidToken : EIDError()

    /** Given token has already been completed. */
    data object TokenAlreadyCompleted : EIDError()

    /** Internal error occurred during the session. */
    data object InternalError : EIDError()

    /** The eID card is already blocked or user blocked it during the session. */
    data object CardBlocked: EIDError()

    /** The eID card is deactivated, authority needs to be contacted. */
    data object CardDeactivated: EIDError()

    /** The scanned card is not compatible, faulty or expired. User should change or update their document. */
    data object InvalidCard: EIDError()

    /** Timeout occurred during the scan session. */
    data object SessionTimeout: EIDError()
}
```

## Customization
The `config` field contains all the parameters to customize your eID experience:
- Specify `showTermsAndConditions(true/false)` to display or hide the terms and conditions screen.
- Specify `customFont` to personalize the font (Typeface) for headings & content.
- Specify `theme` to customize all screen designs. `EIDTheme` is a public object used to let you modify colors, spacing, radius, font sizes, etc. to match your own app theme.

Here is an example of theme and font customization to apply to eID:

```kotlin
/** Define a custom font object to apply to eID SDK text. */
private val exampleCustomFont = EIDCustomFont(
    heading = Typeface.MONOSPACE,
    regularContent = Typeface.SERIF,
    mediumContent = Typeface.SANS_SERIF
)
```

```kotlin
/** Define the custom theme that you want to apply to the SDK. */
private val exampleTheme: EIDTheme = EIDTheme(
    color = EIDColorsApi(
        brandColors = EIDBrandColorsApi(primary = "#000000", secondary = "#FF0000"),
        greyColors = EIDGreyColorsApi(grey100 = "#FFFDD0")
    ),
    radius = EIDRadiusApi(radius2 = 24),
    spacing = EIDSpacingApi(spacing2 = 24),
    font = EIDFontApi(fontSize = EIDFontSizeApi(size0 = 1.2f, size1 = 1.4f))
)
```

```kotlin
/** Set this theme and font directly in the EIDConfig object. */
fun startEidActivity(activity: Activity, token: String) {
    val eidConfig = EIDConfig.Builder()
        .showTermsAndConditions(true)
        .customFont(exampleCustomFont)
        .theme(exampleTheme)
        .build()

    EIDSdk.start(
        activity = activity, 
        token = token, 
        callback = this, 
        config = eidConfig
    )
}
```

🎨 You are now fully ready to install, implement, integrate and customize our SDK.
