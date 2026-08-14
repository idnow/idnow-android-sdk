# TrustPlatform Android SDK

Integrate IDnow Trust Platform identity verification flows into your Android app.

## Requirements

| Requirement | Minimum version  |
| ----------- | ---------------- |
| Android API | 24 (Android 7.0) |
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

## Register for Results

Call `TrustPlatformSession.registerForResult` in `onCreate()`, before the activity reaches `onStart()`:

```kotlin
class MainActivity : ComponentActivity() {

    private lateinit var launcher: ActivityResultLauncher<TrustPlatformConfig>

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        launcher = TrustPlatformSession.registerForResult(this) { result ->
            when (result) {
                is TrustPlatformResult.Completed -> { /* verification succeeded */ }
                is TrustPlatformResult.Cancelled -> { /* user cancelled */ }
                is TrustPlatformResult.Failed    -> { /* result.message contains details */ }
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
launcher.launch(
    TrustPlatformConfig(
        token = token,
        environment = TrustPlatformEnvironment.Production,
    )
)
```

## Handle Results

| Variant           | When it occurs                               |
| ----------------- | -------------------------------------------- |
| `Completed`       | The user completed the verification flow     |
| `Cancelled`       | The user cancelled the flow                  |
| `Failed(message)` | The flow failed — `message` contains details |

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
