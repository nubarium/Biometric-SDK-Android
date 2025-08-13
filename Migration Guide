# Migration Guide: Nubarium Biometrics Android SDK
**From v1.319 → v1.891**

This guide explains how to upgrade your Android application from version **v1.319** to **v1.891** of the Nubarium Biometrics SDK.  
It covers all required changes, including new initializers for both **Face Capture** and **ID Capture** components.

---

## 1) Summary of Key Changes
- **Repository Setup:** Updated Gradle (Kotlin DSL) with JitPack credentials.
- **SDK Dependency:** New version `v1.891`.
- **Face Capture Class Rename:** `FacialCapture` → `FaceCapture`, `FacialResult` → `FaceResult`.
- **New Initializers:**  
  - `FaceCaptureInitializer` for face capture.  
  - `IdCaptureInitializer` for ID capture.
- **Manifest:** Streamlined permissions; only keep what’s needed.
- **ID Result API:** Use `getScore()` instead of deprecated `getConfidence()`.

---

## 2) Project Configuration

### 2.1 Add JitPack Repository
```kotlin
// settings.gradle.kts or build.gradle.kts (Project-level)
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://jitpack.io")
            credentials {
                username = "YOUR_JITPACK_TOKEN" // Replace with your token
                password = "" // Leave empty
            }
        }
    }
}
```

### 2.2 Update SDK Dependency
```kotlin
// app/build.gradle.kts
dependencies {
    implementation("com.github.nubarium:BiometricSDKComponents:v1.891")
}
```

---

## 3) Manifest Permissions
```xml
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />

<uses-permission android:name="android.permission.CAMERA" android:required="true" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- Add if using audio/video capture -->
<!-- <uses-permission android:name="android.permission.RECORD_AUDIO" /> -->
```

---

## 4) Application Initializers (New)
```java
public class MyApp extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // Initialize Face Capture
        com.nubarium.sdk.facecapture.FaceCaptureInitializer.init(getApplicationContext());

        // Initialize ID Capture
        com.nubarium.sdk.idcapture.IdCaptureInitializer.init(getApplicationContext());
    }
}
```
Declare in manifest:
```xml
<application
    android:name=".MyApp"
    ... >
</application>
```

---

## 5) Face Capture Integration

### 5.1 Imports
```java
import com.nubarium.sdk.components.FaceCapture;
import com.nubarium.sdk.components.FaceResult;
```

### 5.2 Setup
```java
private FaceCapture faceCapture;

private void setupFaceCapture(Context context) {
    faceCapture = new FaceCapture(context);
    faceCapture.setCredentials(<NUB_USERNAME>, <NUB_PASSWORD>);
    faceCapture.setShowPreview(false);
    faceCapture.setAntispoofing(true, FaceCapture.ANTISPOOFING_LEVEL_MEDIUM);
}
```

### 5.3 Activity Result
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    faceCapture.process(requestCode, resultCode, data);
    super.onActivityResult(requestCode, resultCode, data);
}
```

### 5.4 Listeners
```java
faceCapture.addOnInitListener(new FaceCapture.OnInitListener() {
    @Override public void onInit(String token) {}
    @Override public void onError(FaceCapture.Error error, String message) {}
    @Override public void onFail(String reason) {}
});

faceCapture.addOnResultListener(new FaceCapture.OnResultListener() {
    @Override public void onSuccess(FaceResult result, Bitmap faceImage, Bitmap areaImage) {}
    @Override public void onFail(FaceCapture.ReasonFail reasonFail, String reason) {}
    @Override public void onError(FaceCapture.Error error, String message) {}
});
```

### 5.5 Start
```java
faceCapture.initialize(); // optional
faceCapture.start();
```

---

## 6) ID Capture Integration

### 6.1 Imports
```java
import com.nubarium.components.enums.CaptureMode;
import com.nubarium.components.sdk.IdResult;
import com.nubarium.components.sdk.IdCapture;
```

### 6.2 Setup
```java
private IdCapture idCapture;

private void setupIdCapture(Context context) {
    idCapture = new IdCapture(context);
    idCapture.setCredentials(<NUB_USERNAME>, <NUB_PASSWORD>);
    idCapture.setCaptureMode(CaptureMode.AUTO);
    idCapture.setAllowCaptureOnFail(true);
    idCapture.setMaxValidations(5);
}
```

### 6.3 Activity Result
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    idCapture.process(requestCode, resultCode, data);
    super.onActivityResult(requestCode, resultCode, data);
}
```

### 6.4 Listeners
```java
idCapture.addOnInitListener(new IdCapture.OnInitListener() {
    @Override public void onInit(String token) {}
    @Override public void onError(IdCapture.Error error, String message) {}
    @Override public void onFail(String reason) {}
});

idCapture.addOnResultListener(new IdCapture.OnResultListener() {
    @Override
    public void onSuccess(IdResult result, Bitmap frontImage, Bitmap backImage, CaptureMode mode) {}
    @Override
    public void onFail(IdResult result, IdCapture.ReasonFail reasonFail, String reason, String[] retro) {}
    @Override
    public void onError(IdCapture.Error error, String message) {}
});
```

### 6.5 Start
```java
idCapture.start();
```

---

## 7) QA Checklist
- [ ] Dependency updated to `v1.891`.
- [ ] JitPack repository configured.
- [ ] Added `FaceCaptureInitializer` and `IdCaptureInitializer`.
- [ ] Updated imports and class names.
- [ ] Manifest cleaned of unused permissions.
- [ ] Switched to `getScore()` for ID results.

---

## 8) Troubleshooting
- **No capture start:** Check initializers in `Application.onCreate()`.
- **Camera errors:** Ensure `CAMERA` permission granted at runtime.
- **Initialization errors:** Verify credentials and network access.

---
