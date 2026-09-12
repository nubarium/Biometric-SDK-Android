# Nubarium Biometric SDK for Android

Integration guide for the Nubarium `FaceCapture` and `IdCapture` components.

## Requirements

- Android API 24 (Android 7.0) or later.
- A `ComponentActivity` or a subclass such as `AppCompatActivity`.
- Camera and internet access.
- Nubarium API credentials or a previously generated biometric token.
- An `applicationId` registered with Nubarium.

> `FaceCapture` and `IdCapture` register an Activity Result launcher internally. Create them from an activity, not from the `Application` object.

## Installation

### 1. Add the repository

Add JitPack to `dependencyResolutionManagement` in `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://jitpack.io")

            // Only required when access to the repository is private.
            credentials {
                username = providers.gradleProperty("JITPACK_TOKEN").orNull
                    ?: System.getenv("JITPACK_TOKEN")
                password = ""
            }
        }
    }
}
```

Keep repository tokens outside source control. For example, define `JITPACK_TOKEN` in the user-level `~/.gradle/gradle.properties` file or as an environment variable.

### 2. Add the dependency

Add the SDK to the application module:

```kotlin
dependencies {
    implementation("com.github.nubarium:BiometricSDKComponents:v1.893")
}
```

This guide uses SDK version **1.893**. If Nubarium provides a different version,
replace the version in the dependency accordingly.

### 3. Declare Android permissions

Add the following entries to `AndroidManifest.xml`:

```xml
<uses-feature android:name="android.hardware.camera" android:required="true" />
<uses-feature android:name="android.hardware.camera.autofocus" android:required="false" />

<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

The host application must declare the camera permission in its manifest. The SDK
requests the camera runtime permission when needed; if the application has
already obtained it, the capture flow continues immediately.

## Optional startup initialization

The initializers check and request the Google Play Services modules used by face detection and OCR. Call them once from `Application.onCreate()`:

```java
import android.app.Application;

import com.nubarium.sdk.facecapture.FaceCaptureInitializer;
import com.nubarium.sdk.idcapture.IdCaptureInitializer;

public final class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        FaceCaptureInitializer.init(getApplicationContext());
        IdCaptureInitializer.init(getApplicationContext());
    }
}
```

This startup initialization is different from `faceCapture.initialize()` and `idCapture.initialize()`, which validate credentials or a biometric token.

## Face capture and liveness

### Imports

```java
import android.graphics.Bitmap;
import android.os.Bundle;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.nubarium.sdk.components.FaceCapture;
import com.nubarium.sdk.components.FaceResult;
```

### Create and configure `FaceCapture`

Create the component in `Activity.onCreate()` so its internal Activity Result launcher is registered at the correct lifecycle stage:

```java
public final class FaceVerificationActivity extends AppCompatActivity {
    private FaceCapture faceCapture;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        faceCapture = new FaceCapture(this);

        // Use credentials supplied securely by your integration.
        faceCapture.setCredentials(NUB_USERNAME, NUB_PASSWORD);

        faceCapture.setAntispoofing(
                true,
                FaceCapture.ANTISPOOFING_LEVEL_MEDIUM
        );

        // Optional adjustments. The component defaults are normally sufficient.
        // faceCapture.setShowPreview(false);
        // faceCapture.setMaxValidations(3);
        // faceCapture.setTimeoutAttempt(40_000); // Milliseconds

        configureFaceCaptureListeners();
    }
}
```

Do not hardcode, persist in plain text, or print usernames, passwords, authorization headers, biometric tokens, request bodies, or Base64 images in production logs.

### Initialization listener

```java
private void configureFaceCaptureListeners() {
    faceCapture.addOnInitListener(new FaceCapture.OnInitListener() {
        @Override
        public void onInit(String token) {
            // Initialization succeeded. Store the token only in secure storage
            // if your integration needs to reuse it.
        }

        @Override
        public void onFail(String reason) {
            // Current values may include:
            // NOT_VALID, INVALID_CREDENTIALS, UNKNOWN
        }

        @Override
        public void onError(FaceCapture.Error error, String message) {
            switch (error) {
                case TOKEN_REQUEST_ERROR:
                    // Invalid service response or response structure.
                    break;
                case BAD_CREDENTIALS:
                    // HTTP 401 while requesting a token.
                    break;
                case SERVICE_ERROR:
                    // HTTP 500.
                    break;
                case SERVICE_NOT_AVAILABLE:
                    // HTTP 404.
                    break;
                case UNKNOWN:
                    // Another HTTP status, transport failure, or parsing failure.
                    break;
                default:
                    // Handle any additional initialization error.
                    break;
            }
        }
    });

    faceCapture.addOnResultListener(new FaceCapture.OnResultListener() {
        @Override
        public void onSuccess(
                FaceResult faceResult,
                Bitmap faceImage,
                Bitmap areaImage
        ) {
            // faceImage: cropped face.
            // areaImage: complete capture area/frame.
            // faceResult: service result, score and operation identifiers.
        }

        @Override
        public void onFail(
                FaceResult faceResult,
                FaceCapture.ReasonFail reasonFail,
                String reason
        ) {
            // The capture did not satisfy the configured validation criteria.
        }

        @Override
        public void onError(FaceCapture.Error error, String message) {
            // Handle a capture error.
        }

        @Override
        public void onCancel() {
            // The user canceled the component.
        }
    });
}
```

### Start `FaceCapture`

You may pre-initialize the component to validate credentials and reduce work performed when capture starts:

```java
faceCapture.initialize();
```

Start the user-facing flow from an event such as a button click:

```java
faceCapture.start();
```

`start()` also initializes the component when no recent valid initialization is available.

No additional `onActivityResult()` forwarding is required. `FaceCapture`
handles activity results through the Activity Result API.

### Customize face-capture messages

Override the SDK resource names in the application's `strings.xml`:

```xml
<resources>
    <string name="nbm_facial_title">Liveness check and face capture</string>
    <string name="nbm_facial_instructions">Position your face in front of the camera and avoid sudden movements.</string>
    <string name="nbm_facial_instructions_list">Stay in a well-lit area.|Hold the camera at a 90-degree angle.</string>

    <string name="nbm_facial_preview_title">Confirm</string>
    <string name="nbm_facial_preview_instructions">Review the photo. You can retry or continue.</string>

    <string name="nbm_facial_btn_back">Back</string>
    <string name="nbm_facial_btn_start">Start</string>
    <string name="nbm_facial_btn_accept">Accept</string>
    <string name="nbm_facial_btn_cancel">Cancel</string>
    <string name="nbm_facial_btn_validate">Validate</string>
    <string name="nbm_facial_btn_finish">Finish</string>

    <string name="nbm_facial_msg_dont_move">DO NOT MOVE</string>
    <string name="nbm_facial_msg_blurred">FOCUS THE CAMERA AND HOLD STILL</string>
    <string name="nbm_facial_msg_face_outside">PLACE YOUR FACE INSIDE THE OVAL</string>
    <string name="nbm_facial_msg_no_face">POSITION YOUR FACE IN FRONT OF THE CAMERA</string>
    <string name="nbm_facial_msg_many_faces">MAKE SURE YOUR FACE IS THE ONLY ONE VISIBLE</string>
    <string name="nbm_facial_msg_far_away">MOVE A LITTLE CLOSER</string>
    <string name="nbm_facial_msg_too_close">MOVE A LITTLE FARTHER AWAY</string>
    <string name="nbm_facial_msg_fail">MOVE TO A BETTER-LIT AREA</string>
    <string name="nbm_facial_msg_static_eye">LOOK DIRECTLY AT THE CAMERA</string>
    <string name="nbm_facial_msg_captured">DONE</string>
</resources>
```

### Customize colors

The component colors can be customized to match the visual identity of the
host application. Override the following resources in the application's
`res/values/colors.xml` file and replace the hexadecimal values with the colors
chosen by your team:

```xml
<resources>
    <color name="colorPrimary">#162B62</color>
    <color name="colorPrimaryDark">#0F2550</color>
    <color name="colorAccent">#162B62</color>
    <color name="colorSecondary">#89D099</color>
</resources>
```

These values are examples. Integrators may change them to use their own brand
palette.

## ID capture

### Imports

```java
import android.graphics.Bitmap;

import com.nubarium.sdk.common.enums.CaptureMode;
import com.nubarium.sdk.components.IdCapture;
import com.nubarium.sdk.components.IdResult;
```

### Create and configure `IdCapture`

Like `FaceCapture`, create `IdCapture` from a `ComponentActivity` or subclass:

```java
private IdCapture idCapture;

private void configureIdCapture() {
    idCapture = new IdCapture(this);
    idCapture.setCredentials(NUB_USERNAME, NUB_PASSWORD);
    idCapture.setCaptureMode(CaptureMode.AUTO);

    // Optional adjustments:
    // idCapture.setAllowCaptureOnFail(true);
    // idCapture.setMaxValidations(5);
}
```

### ID initialization listener

```java
idCapture.addOnInitListener(new IdCapture.OnInitListener() {
    @Override
    public void onInit(String token) {
        // Initialization succeeded.
    }

    @Override
    public void onFail(String reason) {
        // Token validation failed.
    }

    @Override
    public void onError(IdCapture.Error error, String message) {
        // Credential, service, response, or transport error.
    }
});
```

### ID result listener

```java
idCapture.addOnResultListener(new IdCapture.OnResultListener() {
    @Override
    public void onSuccess(
            IdResult idResult,
            Bitmap frontImage,
            Bitmap backImage,
            CaptureMode resultMode
    ) {
        // Capture completed successfully.
    }

    @Override
    public void onFail(
            IdResult idResult,
            IdCapture.ReasonFail reasonFail,
            String reason
    ) {
        // The capture did not satisfy the configured validation criteria.
    }

    @Override
    public void onError(IdCapture.Error error, String message) {
        // Handle component errors.
    }

    @Override
    public void onCancel() {
        // The user canceled the component.
    }
});
```

### Start `IdCapture`

```java
idCapture.initialize(); // Optional pre-initialization
idCapture.start();
```

No additional `onActivityResult()` forwarding is required. `IdCapture` handles
activity results through the Activity Result API.

### `IdResult`

The result includes:

- `getScore()`: current evaluation score.
- `getResult()`: evaluation result, such as `pass`, `warning`, or `fail`.
- `getRetro()`: feedback tags returned by the service.
- `getOcr()`: OCR results and detected labels.
- `getDocumentIdInfo()`: detected document information.

Treat service-provided values as nullable unless the SDK contract for the deployed version guarantees otherwise.

## Security and production logging

Never log or expose:

- Nubarium username or password.
- Basic Authorization headers.
- Biometric tokens or signatures.
- Complete request or response bodies containing personal information.
- Face, frame, front-ID, or back-ID images in raw bytes or Base64.

Use build-time configuration or secure storage for credentials. If diagnostic logging is required, log only a correlation identifier, HTTP status, elapsed time, and a sanitized error category. Disable verbose logging in release builds.
