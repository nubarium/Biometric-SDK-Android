# Biometric-SDK-Android


## SDK compatibility

- Starting Android 6.0 with API v23 or ABOVE.
- Mobile Android-based platforms.

## Installation

Install the Android SDK using Gradle.

### Install using Gradle

**Step 1: Declare repositories**
In the Project `build.gradle` file, declare the `jitpack` repository:

```groovy
maven {
  url 'https://jitpack.io'
}
```

**Step 2: Add dependencies**
In the application `build.gradle` file, add the <u>latest Android SDK</u> package:

Groovy

```groovy
dependencies {
    // Get the latest version from Nubarium Biometrics SDK repository
    implementation 'com.github.nubariumdev:BiometricSDKComponents-Android:0.9.8' 
}
```

Apps targeting lighter installations could use an alternate version that allows to download some artifacts after the application is installed.

```groovy
`dependencies {`
    `// Get the latest version from Nubarium Biometrics SDK repository`
    `implementation 'com.github.nubariumdev:BiometricSDKComponents-Android-Lite:0.9.8'` 
`}`
```



## Setting required permissions

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

## Known issues

### Compatibility

Some text..

### Maven repository

Some text..



## Integrate

## Before you begin

- You must install the Android SDK.
- Ensure that in your app `build.gradle` file, `applicationId`'s value (in the `defaultConfig` block) matches the app's app ID in Nubarium.
- Get the Nubarium Key or API Credentials. It is required to successfully initialize the SDK.
- The codes in this document are example implementations. Make sure to change the `<NUB_KEY>`, `<NUB_USERNAME>`,  `<NUB_PASSWORD>` and other placeholders as needed.
- All the steps in this document are mandatory unless stated otherwise.

## Initializing the Android SDK

It's recommended to initialize the SDK in the [global Application class/subclass](https://developer.android.com/reference/android/app/Application). That is to ensure the SDK can start in any scenario (for example, deep linking).

**Step 1: Import Nubarium library**
In your Application class, import the Nubairum library classes:

JavaKotlin

```java
import com.nubarium.components.enums.*;
import com.nubarium.components.sdk.*;
import com.nubarium.components.tools.*;
```

**Step 2: Initialize the SDK**

**Local variables**

It requires to declare the component as local variable.

```java
private Facial facial;    // For facial component
private IdValidator idValidator;   // For id validator
```

In the global Application `onCreate`, create an instance of the component and set the credentials or API Key (either of the 2 methods can be used) and set the configuration.

**Facial**

```java
// Class initialization with the Application Context
facial = new Facial(this);

// Step 1
// Either of the 2 methods can be used, but only one.
// Set the credentials provided by Nubarium.
facial.setCredentials(<NUB_USERNAME>,<NUB_PASSWORD>);
// Set the Api Key provided by Nubarium.
facial.setApiKey(<NUB_KEY>);

// Step 2 - OPTIONAL
// In case the application has been initialized before and want to reuse a valid token
// Set the valid token and specifies whether autogenerate a new one 
// if the given token is not valid.
facial.setToken(<NUB_TOKEN>, true);  

// Step 3
// Set the basic configuration (Options)
facial.setLivenessRequired(true);  // Default value es true
facial.setShowHelp(false);   // Default value is false
facial.setShowVideo(false);   // Default value is false
facial.setShowPreview(false);   // Defaul values is false


```

1. The first step you set the Credentials or Api Key.

2. The second step is optional and  is used in case the application has been initialized before and want to reuse a valid token.

3. The third step is to configure the behavior of the component.

   * *setLivenesRequired* : Specifies whether the photo capture requires liveness detection.

   * *setShowHelp* : Specifies whether the help button is displayed. It requires the url to be specified in the component String values.

   * *setShowVideo* : Specifies whether the video button is displayed. It requires the url to be specified in the component String values.

   * *setShowPreview* : Specifies whether the dialog requiring a confirmation with a preview photo is displayed. 

     

**Setting up the Activity Result**

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		// Seeting the facial component with the request and result code.
    facial.process(requestCode, resultCode, data);
    super.onActivityResult(requestCode, resultCode, data);
}
```

It is recommended to use the initialization listener, to detect any fail or save the initialization token.

**Initialization with initialization listener**

```java
facial.addOnInitListener(new Facial.OnInitListener() {
    @Override
    public void onInit(String token) {
        // Saves the token in your local storage to reuse in case you need.
    }

    @Override
    public void onFail(String reason) {
        // Track the reason of the initialization fail.
    }
});
```



## Starting 

As in the application the component is declared as a local variable, it can be started in programmatically or in some event such as onClick button.

```java
facial.start();
```

### Setting a result listener

To receive the images and result of component execution it is necessary to setting up a result listener.

```java
facial.addOnResultListener(new Facial.OnResultListener() {

  @Override
  public void onSuccess(FaceResult faceResult, Bitmap faceImage, Bitmap areaImage) {
    
  }

  @Override
  public void onFail(Facial.ReasonFail reasonFail, String reason) {

  }

  @Override
  public void onError(Facial.Error error, String message) {

  }
});
```

- The `onSuccess()` callback method is invoked if the execution of the component was successful, the method returns the following elements.
  - faceResult : An instance of FaceResult with information like confidence and a attack indicator.
  - faceImage : A bitmap with the face cropped.
  - areaImage: A bitmap of the area where the face was framed
- The `onFail(ReasonFail reasonFail, String reason)` callback method is invoked when the liveness validation failed for the given configuration.
- The `onError(Error error, String message)` callback method is invoked when the component throws an error.

## Enabling debug mode

Optional
You can enable debug logs by calling set DebugLog

```java
facial.setDebugLog(true);
```

> ### 📘Note
>
> To see full debug logs, make sure to call `setDebugLog` before invoking other SDK methods.
>
> 

> ### 🚧 Warning
>
> To avoid leaking sensitive information, make sure debug logs are disabled before distributing the app.

## 
