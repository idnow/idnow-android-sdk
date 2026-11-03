# TrustPlatform Android SDK

Integrate IDnow Trust Platform identity verification flows into your Android app.

## Requirements

| Requirement | Minimum version  |
| ----------- | ---------------- |
| Android API | 28 (Android 9.0) |
| Kotlin      | 2.x              |

## Installation

### 1. Add the Maven repository

Add the following to your `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        maven { url = uri("https://raw.githubusercontent.com/idnow/idnow-android-sdk/main") }
    }
}
```

### 2. Add the dependency

Use the BOM to keep module versions in sync:

```kotlin
// build.gradle.kts
dependencies {
    implementation(platform("io.idnow.trustplatform:bom:<version>"))
    implementation("io.idnow.trustplatform:core")
}
```

## Register for Events

Call `TrustPlatformSession.registerForResult` in `onCreate()`, before the activity reaches `onStart()`. It returns a `TrustPlatformLauncher`; call `session.launch(config)` to open the SDK and `session.stop()` to close it without an event:

```kotlin
class MainActivity : ComponentActivity() {

    private lateinit var session: TrustPlatformLauncher

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        session = TrustPlatformSession.registerForResult(this) { event ->
            when (event) {
                is TrustPlatformEvent.Progress -> { /* event.progress says what the current step is waiting for */ }
                is TrustPlatformEvent.Completed -> {
                    // The flow completed. Query your backend for the outcome.
                    // See https://docs.eu.platform.idnow.io/docs/integration/get-session-results
                }
                is TrustPlatformEvent.Aborted -> { /* the session was aborted on the server */ }
                is TrustPlatformEvent.Dismissed -> { /* the SDK closed before the flow ended; the session keeps running */ }
                is TrustPlatformEvent.Failed -> { /* event.error contains details */ }
            }
        }
    }
}
```

## Configure the SDK

Pass a `TrustPlatformEnvironment` when you launch a session.

| Environment                           | Description                                                   |
| ------------------------------------- | ------------------------------------------------------------- |
| `TrustPlatformEnvironment.Production` | The production environment                                    |
| `TrustPlatformEnvironment.Sandbox`    | The sandbox environment for development and testing (default) |

### Configure Permissions

Add the following to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.CAMERA" />
```

## Launch a Session

[Create a session](https://docs.eu.platform.idnow.io/docs/integration/create-session) via the API, then launch the SDK with the session token:

```kotlin
session.launch(
    TrustPlatformConfig(
        token = token,
        environment = TrustPlatformEnvironment.Production,
    )
)
```

## Events

The handler runs on the main thread and receives `TrustPlatformEvent`s:

| Event                               | Meaning                                                                                       |
| ----------------------------------- | --------------------------------------------------------------------------------------------- |
| `Progress(progress: StepProgress)`  | What the current step is waiting for; reported only when the value changes                    |
| `Completed`                         | The flow finished                                                                             |
| `Aborted`                           | The session was aborted on the server                                                         |
| `Dismissed`                         | The SDK closed before the flow ended (for example the back button); the session keeps running |
| `Failed(error: TrustPlatformError)` | The flow failed; `error.message` describes why, suitable for logging                          |

Any number of `Progress` events are followed by at most one terminal event. `session.stop()` closes the SDK without ending the session and without sending an event.

## DocIDV Handler (Optional)

### Add the Dependency

```kotlin
// build.gradle.kts
dependencies {
    implementation(platform("io.idnow.trustplatform:bom:<version>"))
    implementation("io.idnow.trustplatform:core")
    implementation("io.idnow.trustplatform:docidv")
}
```

> **Note:** NFC permissions for eID scanning are included automatically via manifest merging from the DocIDV handler's dependencies. No additional NFC entries are required in your manifest.

### How It Works

When you add `trustplatform-docidv` to your dependencies, the SDK discovers the DocIDV handler automatically at runtime via `ServiceLoader` — no registration call is required. If the dependency is absent, the SDK runs the DocIDV step in the webview instead.
