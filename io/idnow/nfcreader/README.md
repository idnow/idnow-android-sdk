# NFC Reader

An Android SDK for reading electronic machine-readable travel documents (eMRTD) — such as biometric passports and eID cards — over NFC. The reader establishes a secure session with the document's chip, reads its data groups, and verifies their authenticity, either locally or against a remote service.

The repository contains two Gradle modules:

| Module       | Description                                                                                 |
|--------------|---------------------------------------------------------------------------------------------|
| `:NFCReader` | The reusable SDK, published as an `.aar` to a Maven repository (`io.idnow.nfcreader:nfcreader`). |
| `:app`       | A demo application showcasing how to integrate and drive the SDK.                            |

## Features

- Reads eMRTD chips following the ICAO 9303 standard (built on [JMRTD](https://jmrtd.org/)).
- Two authentication protocols: **MRZ** (document number + date of birth + date of expiry) and **CAN**.
- Two operating modes:
  - **Local** — reading and validation happen on-device against bundled certificates.
  - **Remote** — reading happens on-device while validation is delegated to a backend service.
- Reads data groups DG1, DG2 (face image), DG11, DG12, plus COM/SOD, with Active and Chip Authentication support.
- Step-by-step progress callbacks and event/error reporting for logging and telemetry.

## Requirements

- Android `minSdk` **21** (SDK), `compileSdk` **35**
- Java **11**
- Kotlin **2.2.x** / Android Gradle Plugin **8.11.x**
- A physical device with NFC hardware

## Getting started

### Add the dependency

The SDK is published to a Maven repository under the coordinates `io.idnow.nfcreader:nfcreader`. Once the repository is configured in your project:

```kotlin
dependencies {
    implementation("io.idnow.nfcreader:nfcreader:<version>")
}
```

### Manifest permissions

The SDK declares the NFC permission and feature, so consumers do not need to add them manually:

```xml
<uses-permission android:name="android.permission.NFC" />
<uses-feature android:name="android.hardware.nfc" android:required="false" />
```

## Usage

The public entry point is the `NfcReader` singleton (`io.idnow.nfcreader.main.external.NfcReader`).

```kotlin
// 1. Activate once, as early as possible (e.g. in Application.onCreate).
NfcReader.getInstance().activate(applicationContext)

// 2. Configure a session: choose a mode and a protocol, and supply a listener.
NfcReader.getInstance().setupSession(
    mode = NfcReaderMode.Remote(serviceUrl = "https://...", sessionId = sessionId),
    // or NfcReaderMode.Local(certificatesPath = "ml.pem")
    protocol = NfcProtocol.MRZ(documentNumber, dateOfBirth, dateOfExpiry),
    // or NfcProtocol.CAN(can)
    nfcReaderListener = object : NfcReaderListener {
        override fun onSuccess(result: NfcReaderResult) { /* read DG1, DG2, ... */ }
        override fun onError(error: NfcReaderError) { /* handle error */ }
        override fun onStep(step: NfcReadingStep) { /* update UI with progress */ }
        override fun onAborted(cause: NfcReaderError.TerminalError) { /* session aborted */ }
    },
    nfcReaderEventListener = null, // optional, for logging events/errors
)

// 3. Start reading from a ComponentActivity (foreground NFC dispatch).
NfcReader.getInstance().startReading(activity)

// Stop the underlying reader (suspending) when leaving the foreground.
NfcReader.getInstance().stopReading()

// Terminate the session early with a cause and details.
NfcReader.getInstance().abort(cause = "...", details = "...")
```

The lifecycle is: `activate` → `setupSession` → `startReading` → (`onSuccess` / `onError` / `onAborted`). A session is finalized on a successful read or an abort; call `setupSession` again to start another one.

See the `:app` module — in particular `SdkModuleImpl` — for a complete integration example.

## Building

```bash
# Build the SDK
./gradlew :NFCReader:assemble

# Build and install the demo app (dev flavor)
./gradlew :app:assembleDevDebug
```

### Product flavors

Both modules use a `mode` flavor dimension with `dev` and `prod` variants. The `prodDebug` variant is disabled.

## Testing

```bash
# Run unit tests
./gradlew :NFCReader:testDevDebugUnitTest

# Generate a Jacoco coverage report
./gradlew :NFCReader:jacocoDebugCodeCoverage
```

The coverage report is written under `NFCReader/build/reports/`.

## Publishing

The SDK is published as an `.aar`. Artifacts are written to a local Maven layout under `NFCReader/build/gh-maven` (configured in the root `build.gradle.kts`) and then pushed to the public GitHub Maven repository by `pushToGithub.sh` from the CI deploy stage.

```bash
# Publish to the local gh-maven layout
./gradlew :NFCReader:publishProdPublicationToGithubRepository
```

## CI/CD

Continuous integration is defined in `.gitlab-ci.yml` (builds, tests, and the publish/deploy stage that runs `pushToGithub.sh`).

## License

Proprietary — © IDnow.
