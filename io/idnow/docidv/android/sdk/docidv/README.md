# Table of Contents

- [Overview](#overview)
  - [Requirements](#requirements)
  - [Compatibility, End of Support, End of Life](#compatibility-matrix)
- [Integration](#integration)
  - [AAR library](#aar-library)
- [Usage example](#usage-example)
- [Integrate the NFC dependency](#integrate-the-nfc-dependency)
- [SDK error codes](#sdk-error-codes)
  - [How to deal with errors](#how-to-deal-with-errors)

## Overview

DOCIDV SDK supports NFC scanning to read data from the chip on e-documents. The SDK is available in 2 flavours, with NFC feature and without NFC, providing the best flexibility for different use cases of our customers. We recommend using the NFC feature for the best security, fraud prevention and user experience.

- To use the NFC feature, please reach out to your IDnow contact or IDnow Support (support@idnow.io) and obtain the required NFC dependencies. For information about the integration and usage, please refer to the section: [Integrate the NFC dependency](#integrate-the-nfc-dependency)
- In case you would like to continue using our newest version of the library without NFC, please go through the rest of the documentation and follow the integration guide below.

### Compatibility Matrix
Please refer to the following link to find information about current versions, compatibility, end-of-support (EOS) and end-of-life (EOL) dates pertaining to our products: [IDnow Compatibility Matrix: Browser & OS Compatibility guide](https://www.idnow.io/developers/compatibility-overview/)

### Requirements
- minSdkVersion: 24 (Android 7 Nougat)
- compileSdkVersion: 35 (Android 15)
- targetSdkVersion: 35 (Android 15)
- **not supported: devices and emulators based on the x86 architecture**

### AAR library

In your project's `settings.gradle` for KTS, besides any other repositories you already have, include this maven repository:

```groovy
repositories {
  google()
  mavenCentral()
  
  // Add this repo containing the docidv library.
  maven {
      url = uri("https://raw.githubusercontent.com/idnow/idnow-android-sdk/main")
  }
}
```

If you are using Groovy in your project, add the following line in the app module's build.gradle.
```groovy
dependencies {
    implementation 'io.idnow.docidv.android.sdk:docidv:x.y.z' // replace "x.y.z" with the version you want to include
}
```

Or if you use versions catalog, use the below lines:
- in `libs.versions.toml` file:
```toml
[versions]
docidv="1.1.0"

[libraries]
idnow-docidv = { group = "io.idnow.docidv.android.sdk", name = "docidv",  version.ref="docidv" }
```
- in app's `build.gradle` file:
```groovy
implementation(libs.idnow.docidv)
```


#### NOTE: We also supply a special version of our SDK, which is a 1:1 copy of the main version, but it does not contain the FintecSystems SDK.
```toml
[versions]
docidv="1.1.0-no-fintec-XS2A"
```

## Usage example

```kt
    /**
     * Initialize the IDnow SDK
     */
    private fun initSDK() {
        val idnowConfig = IDnowDocIDVConfig.Builder()
            .withLanguage("en") //set specific ISO language code
            .withHttpLogging(true)
            .build()
        IDnowDocIDV.getInstance().initialize(this, idnowConfig)
    }

    /**
     * Call this method after your backend generates the identToken
     */
    private fun startIdentification(identToken: String) {
        IDnowSDK.getInstance().startIdent(identToken) { iDnowDocIDVResult ->
            when (iDnowDocIDVResult.resultType) {
                IDnowDocIDVResult.ResultType.FINISHED -> {
                    //ident completed successfully
                    handleIdentificationSuccess()
                }

                IDnowDocIDVResult.ResultType.CANCELLED -> {
                    //ident was canceled
                    handleIdentificationCanceled(cancelReason = iDnowDocIDVResult.statusCode)
                }

                IDnowDocIDVResult.ResultType.ERROR -> {
                    //ident has an error
                    handleIdentificationError(errorCode = iDnowDocIDVResult.statusCode)
                }
            }
        }
    }
```

Using `withLanguage("lang_code")` you can configure the IDnow library to use a specific language. These ISO 639-1 language codes are currently supported: bg (Bulgarian), cs (Czech), da (Danish), de (German), el (Greek), en (English), es (Spanish), et (Estonian), fi (Finnish), fr (French), hr (Croatian), hu (Hungarian), it (Italian), ja (Japanese), ka (Georgian), ko (Korean), lt (Lithuanian), lv (Latvian), nb (Norwegian), nl (Dutch), pl (Polish), pt (Portuguese), ro (Romanian), ru (Russian), sk (Slovak), sl (Slovenian), sr (Serbian), sv (Swedish), tr (Turkish), zh (Chinese).

### Integrate the NFC dependency
After contacting Customer Support with a request for the NFC package, you will receive an archive to integrate into your application.
This archive is compatible with the corresponding SDK version and onwards, until a new or updated version of the NFC library is released. (For example, if the name of the archive contains the keywords `for AI SDK v5.13.0`, it'd mean that this NFC library is compatible with AI SDK v5.13.0+ onwards, until an updated version of the NFC library is released).

The contents of the received archive will contain:
- detailed instructions file on how to set-up the NFC dependency + fixes on potential build issues which you might encounter based on your local setup
- AAR files for NFC dependency to include in your app

Overview of the NFC integration
- copy all the `.aar` files from the given archive's `libs` folder into your own app's `libs` folder
- make sure to compile the `libs` folder. 
In your `app/build.gradle` file, add the below:
```groovy
implementation fileTree(include: ['*.aar'], dir: 'libs')
//or if you have also jar files, use this line
//implementation fileTree(include: ['*.jar', '*.aar'], dir: 'libs')
```
In your project level `build.gradle` file, make sure to have this
```groovy
repositories {
  //...
  flatDir {
    dirs 'libs'
  }
  //...
}
```
- the above `aar` files require some transitive dependencies, which you need to add manually in your `app/build.gradle` file:
```groovy
    //ReadID Dependencies
    //iddoc connector
    implementation 'com.google.code.gson:gson:2.11.0'
    implementation 'androidx.annotation:annotation:1.5.0'
    implementation 'org.bouncycastle:bcpkix-jdk18on:1.78.1'
    //readid mrz
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.8.6'
    implementation ('com.google.android.gms:play-services-tflite-java:16.1.0') {
        exclude group: "org.tensorflow"
    }
    //readid core
    implementation 'org.jetbrains.kotlin:kotlin-stdlib:1.9.24'
    implementation 'net.sf.scuba:scuba-sc-android:0.0.26'
    implementation 'net.sf.scuba:scuba-smartcards:0.0.20'
    implementation 'edu.ucar:jj2000:5.2'
    implementation("org.ejbca.cvc:cert-cvc:1.4.13") {
        exclude group: 'org.bouncycastle', module: 'bcprov-jdk15on'
    }
    implementation 'org.jmrtd:jmrtd:0.7.42'
    //readid ui core
    implementation 'androidx.fragment:fragment:1.8.1'
    implementation 'androidx.preference:preference-ktx:1.2.1'
    implementation 'com.google.android.material:material:1.12.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.databinding:viewbinding:8.5.2'
    //viz capture
    implementation "com.google.mlkit:barcode-scanning:17.3.0"
    implementation "com.google.mlkit:face-detection:16.1.7"
    //end
```
- add the following proguard rules in your application's `proguard-rules.pro` file
```pro
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

### Note:
Our recommended best practice for optimal user experience is to allow the SDK to use the device language instead of the lang_code.

### Language limitations
Some components that rely on 3rd party technologies such as Biometric Liveness or NFC scanning do not support the dynamic localization and use the device language.

## SDK error codes

In case of IDnowDocIDVResult.ResultType.ERROR, the IDnowDocIDVResult.getStatusCode() method returns one of the error codes below.

```
"E100" --> Ident code syntax incorrect
"E101" --> Ident code not found
"E102" --> Ident code expired
"E103" --> Ident code already completed
"E130" --> Get ident resources failed; invalid response
"E131" --> Get ident resources failed; server reachability
"E150" --> Start ident failed; invalid response
"E151" --> Start ident failed; server reachability
"E152" --> Start ident failed; missing session key
"E170" --> Socket connection force closed
"E171" --> Process force closed
"E180" --> Missing application context
```

### How to deal with errors

- For E102 it is recommended to create another ident, and restart the process with the new ident code.
- For E103 it is recommended to show a screen to the user with the message that they have submitted all info needed and that they should wait for the final result.
- For E170 it is recommended to notify the user that the ident process timed out or was started on a different device and ask them to try again.
- E180 is to alert the host app if the context has been lost (OS restarted/killed SDK process).
- For all other error codes it is recommended to show a generic error for the user and ask them to try again by restarting the process.
