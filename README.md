# Biometric SDK for Android
Nubarium Biometrics Android SDK guides for developers.
[![GitHub Release](https://badgen.net/badge/release/v1.200/cyan)]()  


## SDK compatibility

- Starting Android 6.0 with API v23 or ABOVE.
- Mobile Android-based platforms.

## Installation

Install the Android SDK using Gradle.

### Prerequisites

- You must have your credentials or API Token. 
- Request your credentials at support.

### Install using Gradle

**Step 1: Declare repositories**
In the Project `build.gradle` file, declare the `jitpack` repository:

```groovy
maven {
  url 'https://jitpack.io'
  credentials { username authToken }
}
```

And set de token value in `gradle.properties`  .

```groovy
authToken=AUTH_TOKEN_PROVIDED_BY_NUBARIUM
```

**Step 2: Add dependencies**
In the application `build.gradle` file, add the <u>latest Android SDK</u> package:

```groovy
dependencies {
    // Get the latest version from Nubarium Biometrics SDK repository
    implementation 'com.github.nubarium:BiometricSDKComponents:v1.200' 
}
```



### Setting required permissions

Add the following permissions to `AndroidManifest.xml`:

AndroidManfiest.xml

```xml
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
<uses-feature android:name="android.hardware.camera.flash" />

<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- In case you use the VideoRecording component -->
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />


```

### Known issues

#### Compatibility

#### Maven repository



## Integrate

### Before you begin

- You must install the Android SDK.
- Ensure that in your app `build.gradle` file, `applicationId`'s value (in the `defaultConfig` block) matches the app's app ID in Nubarium.
- Get the Nubarium Key or API Credentials. It is required to successfully initialize the SDK.
- The codes in this document are example implementations. Make sure to change the `<NUB_KEY>`, `<NUB_USERNAME>`,  `<NUB_PASSWORD>` and other placeholders as needed.
- All the steps in this document are mandatory unless stated otherwise.

## Initializing the Android SDK

It's recommended to initialize the SDK in the global Application class/subclass. 

### Facial Capture

#### **Step 1: Import Nubarium library**

In your Application class, import the Nubairum library classes:

```java
import com.nubarium.components.sdk.FacialResult;
import com.nubarium.components.sdk.FacialCapture;
```

#### **Step 2: Initialize the SDK**

**Local variables**

It requires to declare the component as local variable.

```java
private FacialCapture facialCapture;    // Facial Capture component
```

In the global Application `onCreate`, create an instance of the component and set the credentials or API Key (either of the 2 methods can be used) and set the configuration.

```java
// Class initialization with the Application Context
facialCapture = new FacialCapture(this);

// Step 1
// Either of the 2 methods can be used, but only one.
// Set the credentials provided by Nubarium.
facialCapture.setCredentials(<NUB_USERNAME>,<NUB_PASSWORD>);

// Step 3
// Set the basic configuration (Options)
facialCapture.setLivenessRequired(true);  // Default value is true
facialCapture.setShowPreview(false);   // Defaul values is false

```

1. The first step you set the Credentials or Api Key.

2. The second step is optional and  is used in case the application has been initialized before and want to reuse a valid token.

3. The third step is to configure the behavior of the component.

   * *setLivenesRequired* : Specifies whether the photo capture requires liveness detection.

   * *setShowPreview* : Specifies whether the dialog requiring a confirmation with a preview photo is displayed. 

#### Step 3: **Setting up the Activity Result**

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		// Seeting the facial component with the request and result code.
    facialCapture.process(requestCode, resultCode, data);
    super.onActivityResult(requestCode, resultCode, data);
}
```

It is recommended to use the initialization listener, to detect any fail or save the initialization token.

#### Step 4: Setting the initialization listener

```java
facialCapture.addOnInitListener(new FacialCapture.OnInitListener() {
    @Override
    public void onInit(String token) {
        // Saves the token in your local storage to reuse in case you need.
    }

    @Override
    public void onError(FacialCapture.Error error, String message) {
        // Track the erro of the initialization.
    }	    
	   
    @Override
    public void onFail(String reason) {
        // Track the reason of the initialization fail.
    }
});
```

#### Step 5: Setting a result listener

To receive the images and result of component execution it is necessary to setting up a result listener.

```java
facialCapture.addOnResultListener(new FacialCapture.OnResultListener() {

  @Override
  public void onSuccess(FaceResult faceResult, Bitmap faceImage, Bitmap areaImage) {
    
  }

  @Override
  public void onFail(FacialCapture.ReasonFail reasonFail, String reason) {

  }

  @Override
  public void onError(FacialCapture.Error error, String message) {

  }
});
```

- The `onSuccess()` callback method is invoked if the execution of the component was successful, the method returns the following elements.
  - faceResult : An instance of FaceResult with information like confidence and a attack indicator.
  - faceImage : A bitmap with the face cropped.
  - areaImage: A bitmap of the area where the face was framed
- The `onFail(String reason)` callback method is invoked when the liveness validation failed for the given configuration.
- The `onError(String error)` callback method is invoked when the component throws an error.

#### Step 5: Start component

As in the application the component is declared as a local variable, it can be started in programmatically or in some event such as onClick button.

```java
facialCapture.start();
```

### ID Capture

#### **Step 1: Import Nubarium library**

In your Application class, import the Nubairum library classes:

```java
import com.nubarium.components.enums.CaptureMode;
import com.nubarium.components.sdk.IdResult;
import com.nubarium.components.sdk.IdCapture;
```

#### **Step 2: Initialize the SDK**

**Local variables**

It requires to declare the component as local variable.

```java
private IdCapture idCapture;   // Id Capture component
```

In the global Application `onCreate`, create an instance of the component and set the credentials or API Key (either of the 2 methods can be used) and set the configuration.

```java
// Class initialization with the Application Context
idCapture = new IdCapture(this);

// Step 1
// Either of the 2 methods can be used, but only one.
// Set the credentials provided by Nubarium.
idCapture.setCredentials(<NUB_USERNAME>,<NUB_PASSWORD>);

// Step 3
// Set the basic configuration (Options)
idCapture.setCaptureMode(CaptureMode.AUTO);  // Default value is CaptureMode.AUTO
idCapture.setShowPreview(false);   // Defaul values is false
```

1. The first step you set the Credentials or Api Key.

2. The second step is optional and  is used in case the application has been initialized before and want to reuse a valid token.

3. The third step is to configure the behavior of the component.

   * *setCaptureMode* : Specifies the capture method, it could be useful if need to force a PASSIVE o ACTIVE MODE.

   * *setShowPreview* : Specifies whether the dialog requiring a confirmation with a preview photo is displayed. 

#### Step 3: **Setting up the Activity Result**

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		// Seeting the facial component with the request and result code.
    idCapture.process(requestCode, resultCode, data);
    super.onActivityResult(requestCode, resultCode, data);
}
```

It is recommended to use the initialization listener, to detect any fail or save the initialization token.

#### Step 4: Setting the initialization listener

```java
idCapture.addOnInitListener(new IdCapture.OnInitListener() {
    @Override
    public void onInit(String token) {
        // Saves the token in your local storage to reuse in case you need.
    }

    @Override
    public void onError(FacialCapture.Error error, String message) {
        // Track the erro of the initialization.
    }	    
	    
    @Override
    public void onFail(String reason) {
        // Track the reason of the initialization fail.
    }
});
```
#### Step 5: Setting a result listener

To receive the images and result of component execution it is necessary to setting up a result listener.

```java
idCapture.addOnResultListener(new Facial.OnResultListener() {

  @Override
  public void onSuccess(IdResult validateResultRet, Bitmap frontImage, Bitmap backImage, CaptureMode captureMode) {
    
  }

  @Override
  public void onFail(IdCapture.ReasonFail reasonFail, String reason) {

  }

  @Override
  public void onError(IdCapture.Error error, String message) {

  }
});
```

#### Step 6: Start component

As in the application the component is declared as a local variable, it can be started in programmatically or in some event such as onClick button.

```java
idCapture.start();
```
