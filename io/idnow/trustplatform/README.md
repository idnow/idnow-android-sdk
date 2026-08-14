# TrustPlatform Android SDK

Android SDK for integrating IDnow Trust Platform identity verification flows into your app.

## Requirements

- Android API 24+ (Android 7.0)
- Java 17 toolchain
- Kotlin 2.x

## Modules

| Module                 | Description                                           |
| ---------------------- | ----------------------------------------------------- |
| `trustplatform`        | Core SDK — session lifecycle, web player              |
| `trustplatform-common` | Shared SPI interfaces for native handlers             |
| `trustplatform-docidv` | Optional DocID native handler (document verification) |
| `trustplatform-bom`    | Bill of Materials for version alignment               |

## Integration

### 1. Add the Maven repository

Add the raw-git Maven repository to your project's `settings.gradle.kts`:

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

    // Optional: include for native document verification support.
    // Without it, document verification falls back to the web player.
    implementation("io.idnow.trustplatform:docidv")
}
```

### 3. Register for results

Register the session launcher in your `ComponentActivity`. This **must** be called before `onStart()` (i.e., at field initialisation or in `onCreate()`):

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

### 4. Launch a session

```kotlin
launcher.launch(TrustPlatformConfig(token = "<your-client-token>", environment = TrustPlatformEnvironment.Sandbox))
```

`token` is the client token obtained from your backend after initiating a flow execution. `environment` targets the Player deployment:

| Value                                        | Player URL                            |
| -------------------------------------------- | ------------------------------------- |
| `TrustPlatformEnvironment.Production`        | `https://player.eu.platform.idnow.io` |
| `TrustPlatformEnvironment.Sandbox` (default) | `https://player.eu.platform.idnow.sx` |

## Result types

`TrustPlatformResult` is a sealed class with three variants:

| Variant           | Meaning                                                          |
| ----------------- | ---------------------------------------------------------------- |
| `Completed`       | The user completed the verification flow successfully            |
| `Cancelled`       | The user pressed back / cancelled the flow                       |
| `Failed(message)` | The flow failed; `message` contains a human-readable description |

## Permissions

The SDK declares `INTERNET` in its own manifest. If you include `trustplatform-docidv`, your app also needs `CAMERA`:

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" android:required="true" />
```

## Native handler extension model

Native handlers let specific flow steps run as native Android screens instead of the web player. `trustplatform-docidv` is the first such handler (provider `"docidv"`).

The SDK discovers handlers at runtime via `java.util.ServiceLoader`. If a handler module is on the classpath and its provider matches the directive, it handles the step natively. If the module is absent or cannot handle the directive, the step falls back to the web player — so the SDK works without any handler module.
