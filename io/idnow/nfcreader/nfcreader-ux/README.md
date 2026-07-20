# NFC Reader UX

[![pipeline status](https://gitlab.eu.idnow.group/idv/docidv/nfc-reader-ux-android/badges/develop/pipeline.svg)](https://gitlab.eu.idnow.group/idv/docidv/nfc-reader-ux-android/-/commits/develop)
[![coverage report](https://gitlab.eu.idnow.group/idv/docidv/nfc-reader-ux-android/badges/develop/coverage.svg)](https://gitlab.eu.idnow.group/idv/docidv/nfc-reader-ux-android/-/commits/develop)
[![latest release](https://gitlab.eu.idnow.group/idv/docidv/nfc-reader-ux-android/-/badges/release.svg)](https://gitlab.eu.idnow.group/idv/docidv/nfc-reader-ux-android/-/releases)

NFC Reader UX is the user experience for NFC document reading based on the [NFC Reader SDK](https://gitlab.eu.idnow.group/idv/docidv/nfc-reader-android).

## Getting started

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        // ...

        maven("https://raw.githubusercontent.com/idnow/idnow-android-sdk/main")
    }
}

// app/build.gradle.kts
implementation("io.idnow.nfcreader:nfcreader-ux:<latest>")
```

Internal projects that depend on the `dev` variant add the GitLab **group** registry as a Maven repository. Use the `docidv` group (id `3241`), which contains this SDK:

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        maven {
            url = uri("https://gitlab.eu.idnow.group/api/v4/groups/3241/-/packages/maven")
            name = "GitLab"
            credentials(HttpHeaderCredentials::class) {
                // CI:    name = "Job-Token",    value = System.getenv("CI_JOB_TOKEN")
                // local: name = "Deploy-Token", value = <read_package_registry token>
                name = gitlabTokenHeader
                value = gitlabTokenValue
            }
            authentication { create<HttpHeaderAuthentication>("header") }
        }
    }
}

// app/build.gradle.kts
dependencies {
    implementation("io.idnow.nfcreader:nfcreader-ux:<version>-dev")
}
```

### Configure the library

To use the NFC Reader UX library, it should first be configured. All of these parameters are detailed in the [Settings](#settings) section.
A common call would look like the following:

```kotlin
NfcReaderUx.configure(context, config) { result ->
    // Result is of type NfcUxResult and contains information about the success or failure
    when (result) {
        NfcUxResult.Success -> TODO("Handle success")
        // Failure contains information about the error type as NfcUxError
        is NfcUxResult.Failure -> TODO("Handle failure")
    }
}
```

> [!important]
> Make sure that the result callback does not hold a reference to an object likely to change due to configuration change.
> A good place to call configure would be inside a ViewModel, surviving configuration change.

### Introduce the library entrypoint

Import the `NfcReaderUxFragment` entrypoint as part of your navigation graph.

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation>
    
    <fragment
        android:id="@+id/myHostFragment">
        <action
            android:id="@+id/action_myHostFragment_to_nfcReaderUxFragment"
            app:destination="@id/nfcReaderUxFragment" />
    </fragment>

    <fragment
        android:id="@+id/nfcReaderUxFragment"
        android:name="io.idnow.nfc.reader.ux.ui.entrypoint.NfcReaderUxFragment" />

</navigation>
```

### Start the flow

To effectively start the NFC Reader UX flow, you only have to navigate to the entrypoint you added to your graph:

```kotlin
findNavController().navigate(MyHostFragmentDirections.actionMyHostFragmentToNfcReaderUxFragment())
```

> [!note]
> When the flow finishes, it automatically closes the flow fragment, thus removing it from your backstack.

## Settings

### Theme customization

The UX library is based on [IDnow Design System](https://gitlab.eu.idnow.group/idv/docidv/design-system-android) (Sunflower).
If you are already using it, the theme you applied will automatically be used, check the library for it's integration otherwise.

### Configuration

The NFC Reader UX configuration is defined as follow:

```kotlin
data class NfcUxConfig(
    val mode: NfcReaderMode,
    val protocol: NfcProtocol,
    val documentType: NfcDocumentType,
    val consentUrl: String?,
)
```

| Property Name  | Description                                                                                                                     |
|----------------|---------------------------------------------------------------------------------------------------------------------------------|
| mode           | Whether the reading operation should be done **Online** or in **Local**                                                         |
| protocol       | The information which will be used to perform the reading, either **CAN** or **MRZ**.                                           |
| documentType   | The type of the document which will be read. Only impact UX with different title and animation.                                 |
| consentUrl     | The consent url which should redirect the user to a consent page for NFC reading. If not set, the `consent tip` will be hidden. |
| hideSkipButton | Allows the "Skip this step" button to be hidden if set to true.                                                                 |

For more information on `mode` and `protocol` properties, please refer respectively to the [reader mode](https://idnowgmbh.atlassian.net/wiki/spaces/MWC/pages/3618803599/Android+iOS+NFC+Reader+SDK#Reader-mode)
and [authentication protocols](https://idnowgmbh.atlassian.net/wiki/spaces/MWC/pages/3618803599/Android+iOS+NFC+Reader+SDK#Authentication-protocols-and-keys)
documentation.

### Translations

The library is translated in English by default but it is possible to override these translations.
You should provide an implementation of `NfcUxTranslation` which defines the translations using the [StringResource](nfc-reader-ux/src/main/java/io/idnow/nfc/reader/ux/translations/resource/StringResource.kt)
class. 

```kotlin
class MyCustomTranslations : NfcUxTranslations {
    override fun activationTitle(): StringResource = R.string.my_custom_activation_title.asResource()
    
    // ...
}
```

### Result

The result returned from the listener is an `NfcUxResult` and contains information about the success of failure of the flow:

| Type                | Data Type  |
|---------------------|------------|
| NfcUxResult.Success | none       |
| NfcUxResult.Failure | NfcUxError |

The `NfcUxError` can be any of the following:

| Error Type     | Description                                                                          |
|----------------|--------------------------------------------------------------------------------------|
| Aborted        | The user has aborted from the quit flow with the optional given reason.              |
| NoMoreRetry    | The user has quit the flow because the retry count has been consumed.                |
| RefusedConsent | The user has quit the flow from the consent screen by refusing the NFC read consent. |
| ReaderError    | An error occurred during the NFC Tag read operation.                                 |
