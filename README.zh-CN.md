# Nubarium Android 生物识别 SDK

[English](README.md) | **简体中文**

本文档介绍如何集成 Nubarium `FaceCapture`（人脸采集）和 `IdCapture`（证件采集）组件。

## 环境要求

- Android API 24（Android 7.0）或更高版本。
- `ComponentActivity` 或其子类，例如 `AppCompatActivity`。
- 相机和网络访问权限。
- Nubarium API 凭据，或已生成的生物识别令牌。
- 已在 Nubarium 注册的应用 `applicationId`。

> `FaceCapture` 和 `IdCapture` 会在内部注册 Activity Result Launcher。因此，必须在 Activity 中创建组件实例，不要使用 `Application` 对象创建。

## 安装

### 1. 添加仓库

在 `settings.gradle.kts` 的 `dependencyResolutionManagement` 中添加 JitPack：

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://jitpack.io")

            // 仅在访问私有仓库时需要。
            credentials {
                username = providers.gradleProperty("JITPACK_TOKEN").orNull
                    ?: System.getenv("JITPACK_TOKEN")
                password = ""
            }
        }
    }
}
```

请勿将仓库令牌提交到源代码管理系统。可以将 `JITPACK_TOKEN` 配置在用户级 `~/.gradle/gradle.properties` 文件中，或通过环境变量提供。

### 2. 添加依赖

在应用模块中添加 SDK：

```kotlin
dependencies {
    implementation("com.github.nubarium:BiometricSDKComponents:v1.893")
}
```

本文档使用的 SDK 版本为 **1.893**。如果 Nubarium 向你提供了其他版本，请相应地替换依赖中的版本号。

### 3. 声明 Android 权限

在 `AndroidManifest.xml` 中添加：

```xml
<uses-feature android:name="android.hardware.camera" android:required="true" />
<uses-feature android:name="android.hardware.camera.autofocus" android:required="false" />

<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

宿主应用必须在清单中声明相机权限。SDK 会在需要时请求相机运行时权限；如果应用已提前获得该权限，采集流程将直接继续。

## 可选的启动初始化

初始化器会检查并请求人脸检测和 OCR 所需的 Google Play 服务模块。请在 `Application.onCreate()` 中调用一次：

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

此启动初始化不同于 `faceCapture.initialize()` 和 `idCapture.initialize()`；后两者用于验证凭据或生物识别令牌。

## 人脸采集与活体检测

### 导入类

```java
import android.graphics.Bitmap;
import android.os.Bundle;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.nubarium.sdk.components.FaceCapture;
import com.nubarium.sdk.components.FaceResult;
```

### 创建并配置 `FaceCapture`

请在 `Activity.onCreate()` 中创建组件，以便其内部 Activity Result Launcher 在正确的生命周期阶段完成注册：

```java
public final class FaceVerificationActivity extends AppCompatActivity {
    private FaceCapture faceCapture;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        faceCapture = new FaceCapture(this);

        // 使用集成方安全提供的凭据。
        faceCapture.setCredentials(NUB_USERNAME, NUB_PASSWORD);

        faceCapture.setAntispoofing(
                true,
                FaceCapture.ANTISPOOFING_LEVEL_MEDIUM
        );

        // 可选调整。通常使用组件默认值即可。
        // faceCapture.setShowPreview(false);
        // faceCapture.setMaxValidations(3);
        // faceCapture.setTimeoutAttempt(40_000); // 毫秒

        configureFaceCaptureListeners();
    }
}
```

在生产日志中，禁止硬编码、明文保存或打印用户名、密码、Authorization 请求头、生物识别令牌、请求正文以及 Base64 图像。

### 初始化监听器

```java
private void configureFaceCaptureListeners() {
    faceCapture.addOnInitListener(new FaceCapture.OnInitListener() {
        @Override
        public void onInit(String token) {
            // 初始化成功。仅当集成确实需要复用令牌时，
            // 才将其保存在安全存储中。
        }

        @Override
        public void onFail(String reason) {
            // 当前可能的值包括：
            // NOT_VALID、INVALID_CREDENTIALS、UNKNOWN
        }

        @Override
        public void onError(FaceCapture.Error error, String message) {
            switch (error) {
                case TOKEN_REQUEST_ERROR:
                    // 服务响应或响应结构无效。
                    break;
                case BAD_CREDENTIALS:
                    // 请求令牌时返回 HTTP 401。
                    break;
                case SERVICE_ERROR:
                    // HTTP 500。
                    break;
                case SERVICE_NOT_AVAILABLE:
                    // HTTP 404。
                    break;
                case UNKNOWN:
                    // 其他 HTTP 状态、传输错误或解析错误。
                    break;
                default:
                    // 处理其他初始化错误。
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
            // faceImage：裁剪后的人脸图像。
            // areaImage：完整采集区域/画面。
            // faceResult：服务结果、评分和操作标识符。
        }

        @Override
        public void onFail(
                FaceResult faceResult,
                FaceCapture.ReasonFail reasonFail,
                String reason
        ) {
            // 本次采集未满足所配置的验证条件。
        }

        @Override
        public void onError(FaceCapture.Error error, String message) {
            // 处理采集错误。
        }

        @Override
        public void onCancel() {
            // 用户取消了组件操作。
        }
    });
}
```

### 启动 `FaceCapture`

可以预先初始化组件，以验证凭据并减少开始采集时需要执行的工作：

```java
faceCapture.initialize();
```

通过按钮点击等用户事件启动界面流程：

```java
faceCapture.start();
```

如果近期没有有效的初始化结果，`start()` 也会初始化组件。

无需额外转发 `onActivityResult()`。`FaceCapture` 会通过 Activity Result API
处理 Activity 返回结果。

### 自定义人脸采集文案

可在应用的 `strings.xml` 中覆盖 SDK 的同名资源：

```xml
<resources>
<string name="nbm_facial_title">活体检测与人脸采集</string>
<string name="nbm_facial_instructions">请将面部正对相机，并避免突然移动。</string>
<string name="nbm_facial_instructions_list">请保持环境光线充足。|请以 90 度角握持相机。</string>

<string name="nbm_facial_preview_title">确认</string>
<string name="nbm_facial_preview_instructions">请检查照片。你可以重新采集或继续。</string>

<string name="nbm_facial_btn_back">返回</string>
<string name="nbm_facial_btn_start">开始</string>
<string name="nbm_facial_btn_accept">确定</string>
<string name="nbm_facial_btn_cancel">取消</string>
<string name="nbm_facial_btn_validate">验证</string>
<string name="nbm_facial_btn_finish">完成</string>

<string name="nbm_facial_msg_dont_move">请勿移动</string>
<string name="nbm_facial_msg_blurred">请对焦相机并保持不动</string>
<string name="nbm_facial_msg_face_outside">请将面部置于椭圆框内</string>
<string name="nbm_facial_msg_no_face">请将面部正对相机</string>
<string name="nbm_facial_msg_many_faces">请确保画面中只有你的面部</string>
<string name="nbm_facial_msg_far_away">请靠近一些</string>
<string name="nbm_facial_msg_too_close">请远离一些</string>
<string name="nbm_facial_msg_fail">请前往光线更充足的环境</string>
<string name="nbm_facial_msg_static_eye">请正视相机</string>
<string name="nbm_facial_msg_captured">完成</string>
</resources>
```

### 自定义颜色

组件颜色可以根据宿主应用的视觉风格进行自定义。在应用的 `res/values/colors.xml` 文件中覆盖以下资源，并将十六进制值替换为团队选择的颜色：

```xml
<resources>
    <color name="colorPrimary">#162B62</color>
    <color name="colorPrimaryDark">#0F2550</color>
    <color name="colorAccent">#162B62</color>
    <color name="colorSecondary">#89D099</color>
</resources>
```

以上颜色仅为示例，集成方可以替换为自己的品牌配色。

## 证件采集

### 导入类

```java
import android.graphics.Bitmap;

import com.nubarium.sdk.common.enums.CaptureMode;
import com.nubarium.sdk.components.IdCapture;
import com.nubarium.sdk.components.IdResult;
```

### 创建并配置 `IdCapture`

与 `FaceCapture` 一样，请在 `ComponentActivity` 或其子类中创建 `IdCapture`：

```java
private IdCapture idCapture;

private void configureIdCapture() {
    idCapture = new IdCapture(this);
    idCapture.setCredentials(NUB_USERNAME, NUB_PASSWORD);
    idCapture.setCaptureMode(CaptureMode.AUTO);

    // 可选调整：
    // idCapture.setAllowCaptureOnFail(true);
    // idCapture.setMaxValidations(5);
}
```

### 证件采集初始化监听器

```java
idCapture.addOnInitListener(new IdCapture.OnInitListener() {
    @Override
    public void onInit(String token) {
        // 初始化成功。
    }

    @Override
    public void onFail(String reason) {
        // 令牌验证失败。
    }

    @Override
    public void onError(IdCapture.Error error, String message) {
        // 凭据、服务、响应或传输错误。
    }
});
```

### 证件采集结果监听器

```java
idCapture.addOnResultListener(new IdCapture.OnResultListener() {
    @Override
    public void onSuccess(
            IdResult idResult,
            Bitmap frontImage,
            Bitmap backImage,
            CaptureMode resultMode
    ) {
        // 采集成功完成。
    }

    @Override
    public void onFail(
            IdResult idResult,
            IdCapture.ReasonFail reasonFail,
            String reason
    ) {
        // 本次采集未满足所配置的验证条件。
    }

    @Override
    public void onError(IdCapture.Error error, String message) {
        // 处理组件错误。
    }

    @Override
    public void onCancel() {
        // 用户取消了组件操作。
    }
});
```

### 启动 `IdCapture`

```java
idCapture.initialize(); // 可选的预初始化
idCapture.start();
```

无需额外转发 `onActivityResult()`。`IdCapture` 会通过 Activity Result API
处理 Activity 返回结果。

### `IdResult`

结果包含：

- `getScore()`：当前验证评分。
- `getResult()`：验证结果，例如 `pass`、`warning` 或 `fail`。
- `getRetro()`：服务返回的反馈标签。
- `getOcr()`：OCR 结果和检测到的标签。
- `getDocumentIdInfo()`：检测到的证件信息。

除非所部署版本的 SDK 契约明确保证非空，否则应将服务返回值视为可能为空。

## 安全与生产日志

禁止记录或公开以下信息：

- Nubarium 用户名或密码。
- Basic Authorization 请求头。
- 生物识别令牌或签名。
- 包含个人信息的完整请求或响应正文。
- 人脸、完整画面、证件正面或证件背面的原始字节或 Base64 图像。

请通过构建时配置或安全存储管理凭据。如果必须记录诊断信息，仅记录关联标识符、HTTP 状态码、耗时和经过清理的错误类别，并在 Release 构建中禁用详细日志。
