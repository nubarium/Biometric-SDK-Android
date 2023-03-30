# Biometric SDK for Android
Nubarium Biometrics Android SDK guides for developers.
[![GitHub Release](https://badgen.net/badge/release/v1.221/cyan)]()  


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
    implementation 'com.github.nubarium:BiometricSDKComponents:v1.221' 
}
```



### Setting required permissions

Add the following permissions to `AndroidManifest.xml`:

AndroidManfiest.xml

```xml
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
<uses-permission android:name="android.permission.CAMERA" android:required="true" />

<uses-permission android:name="android.permission.INTERNET" />

<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- In case you use the VideoRecording component -->
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />


```

### Known issues

#### Compatibility



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

// Either of the 2 methods can be used, but only one.
// Set the credentials provided by Nubarium.
facialCapture.setCredentials(<NUB_USERNAME>,<NUB_PASSWORD>);

// Set the basic configuration (Options)
facialCapture.setLivenessRequired(true);  // Default value is true
facialCapture.setShowPreview(false);   // Defaul values is false

```

1. First, you have to set the Credentials or Api Key.
3. Then configure the behavior of the component.

   * *setLivenesRequired* : Specifies whether the photo capture requires liveness detection.
* *setShowPreview* : Specifies whether the dialog requiring a confirmation with a preview photo is displayed. 

**Setting up a Help Video (Optional)**

A help video can be enabled with the component, with this video a more detailed explanation of how to perform the tests can be provided. 

```java
// By default is false
facialCapture.enableVideoHelp(false);
```

In case that its *enabled*, its neccesary to provide a video URL in your `strings.xml` resource.

```xml
<string name="nbm_facial_url_video">https://yourcompany.com/facial.mp4</string>
```

**Customize messages (Optional)**

The messages and labels can be customized in your  `strings.xml` resource.

```xml
<string name="nbm_facial_title">Prueba de vida y Captura facial.</string>
<string name="nbm_facial_instructions">Para realizar la prueba de vida y la captura facial es necesario que enfoque la camara directamente en su rostro y evite moverse de manera brusca para un mejor enfoque.</string>
<string name="nbm_facial_instructions_list">Permanezca en un lugar iluminado.|Sostenga la camara en un ángulo de 90 grados.</string>

<string name="nbm_facial_preview_title">Confirmar</string>
<string name="nbm_facial_preview_instructions">Al realizar la validación tomamos una fotografía que usaremos en nuestro proceso de validación, si deseas, puedes reintentarlo oprimiendo en Reintentar, de lo contrarío sólo oprime el boton para Continuar.</string>

<string name="nbm_facial_btn_back">Regresar</string>
<string name="nbm_facial_btn_start">Iniciar</string>
<string name="nbm_facial_btn_accept">Aceptar.</string>
<string name="nbm_facial_btn_cancel">Cancelar</string>
<string name="nbm_facial_btn_validate">Validar</string>
<string name="nbm_facial_btn_finish">Finalizar</string>

<string name="nbm_facial_msg_dont_move">NO TE MUEVAS</string>
<string name="nbm_facial_msg_blurred">ENFOCA LA CAMARA Y NO SE MUEVA</string>
<string name="nbm_facial_msg_face_outside">COLOQUE SU ROSTRO DENTRO DEL OVALO</string>
<string name="nbm_facial_msg_no_face">COLOQUE SU ROSTRO FRENTE A LA CAMARA</string>
<string name="nbm_facial_msg_many_faces">ASEGURE QUE SU ROSTRO SEA EL UNICO</string>
<string name="nbm_facial_msg_far_away">ACERCATE UN POCO</string>
<string name="nbm_facial_msg_too_close">ALEJATE UN POCO</string>
<string name="nbm_facial_msg_fail">COLOQUESE EN UN LUGAR MEJOR ILUMINADO</string>
<string name="nbm_facial_msg_static_eye">ENFOQUE SU ROSTRO FRENTE A LA CAMARA</string>
<string name="nbm_facial_msg_captured">LISTO</string>
```

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

// Either of the 2 methods can be used, but only one.
// Set the credentials provided by Nubarium.
idCapture.setCredentials(<NUB_USERNAME>,<NUB_PASSWORD>);

// Set the basic configuration (Options)
idCapture.setCaptureMode(CaptureMode.AUTO);  // Default value is CaptureMode.AUTO
idCapture.setAllowCaptureOnFail(true); // Optional, let to take the ID capture even the validation fails.
idCapture.setMaxValidations(5);  // Optional, set the maximum of validations to finalize the task.
```

1. First, you have to set the Credentials or Api Key.
3. Then configure the behavior of the component.

   * *setCaptureMode* : Specifies the capture method, it could be useful if need to force a PASSIVE o ACTIVE MODE.
   * *setAllowCaptureOnFail* : Specifies the flag, that lets to take the ID capture even the validation fails.
   * *setMaxValidations* : Specifies maximum number of validations.

**Setting up a Help Video (Optional)**

A help video can be enabled with the component, with this video a more detailed explanation of how to perform the tests can be provided. 

```java
// By default is false
idCapture.enableVideoHelp(false);
```

In case that its *enabled*, its neccesary to provide a video URL in your `strings.xml` resource.

```xml
<string name="nbm_id_url_video">https://yourcompany.com/id.mp4</string>
```

**Customize messages (Optional)**

The messages and labels can be customized in your  `strings.xml` resource.

```xml
<string name="nbm_id_title">Captura de Identificación</string>
<string name="nbm_id_instructions">Favor de capturar ambos lados de su credencial de elector.</string>

<string name="nbm_id_btn_proceed_capture">CAPTURAR IDENTIFICACION</string>

<string name="nbm_id_btn_back">Regresar</string>
<string name="nbm_id_btn_start">Iniciar</string>
<string name="nbm_id_btn_accept">Aceptar</string>
<string name="nbm_id_btn_cancel">Cancelar</string>
<string name="nbm_id_btn_validate">Validar</string>
<string name="nbm_id_btn_finish">Finalizar</string>

<string name="nbm_id_msg_validating">ESPERA UN MOMENTO, VALIDANDO ID</string>
<string name="nbm_id_msg_center_inside">CENTRA EL ID DENTRO DEL RECUADRO</string>
<string name="nbm_id_msg_too_far">ACERCA EL ID</string>
<string name="nbm_id_msg_taking_picture">ESPERA UN MOMENTO, VALIDANDO ID</string>
<string name="nbm_id_msg_try_better">INTENTE EN UN LUGAR MEJOR ILUMINADO</string>
<string name="nbm_id_msg_no_id">COLOCA EL ID DENTRO DEL RECUADRO</string>
<string name="nbm_id_msg_blurred">ENFOQUE LA CAMARA</string>
<string name="nbm_id_msg_not_valid">La identificación capturada no es válida, favor de capturarla de nuevo</string>
<string name="nbm_id_msg_captured">Capturado</string>
```

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
idCapture.addOnResultListener(new IdCapture.OnResultListener() {

  @Override
  public void onSuccess(IdResult validateResultRet, Bitmap frontImage, Bitmap backImage, CaptureMode captureMode) {
    
  }

  @Override
  public void onFail(IdResult validateResultRet,IdCapture.ReasonFail reasonFail, String reason, String[] retro) {
