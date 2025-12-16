# 第 2 章：项目配置与依赖

## 章节简介

在本章中，我们将学习如何配置 Flutter 项目以支持 Google Mobile Ads，包括创建项目、添加依赖、配置 Android 和 iOS 平台设置，以及获取测试广告单元 ID。这是实现 Rewarded Ads 的基础步骤。

## 创建 Flutter 项目

如果你还没有 Flutter 项目，可以使用以下命令创建一个新项目：

```bash
flutter create rewarded_example
cd rewarded_example
```

如果你已经有项目，可以直接在现有项目中添加 Google Mobile Ads 支持。

## 添加 google_mobile_ads 依赖

### 更新 pubspec.yaml

打开项目的 `pubspec.yaml` 文件，在 `dependencies` 部分添加 `google_mobile_ads` 依赖：

```yaml
name: rewarded_example
description: "Example project for demoing rewarded ads."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.9.2

dependencies:
  flutter:
    sdk: flutter
  google_mobile_ads: ^6.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0

flutter:
  uses-material-design: true
```

### 安装依赖

保存文件后，运行以下命令安装依赖：

```bash
flutter pub get
```

这将下载并安装 `google_mobile_ads` 包及其所有依赖项。

## Android 平台配置

### 1. 更新 AndroidManifest.xml

打开 `android/app/src/main/AndroidManifest.xml` 文件，确保包含以下权限和配置：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.rewarded_example">
    
    <!-- 必需的权限 -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    
    <application
        android:label="rewarded_example"
        android:name="${applicationName}"
        android:icon="@mipmap/ic_launcher">
        
        <!-- 其他配置 -->
        
    </application>
</manifest>
```

### 2. 更新 build.gradle

打开 `android/app/build.gradle` 文件，确保 `minSdkVersion` 至少为 21：

```gradle
android {
    compileSdkVersion 34

    defaultConfig {
        applicationId "com.example.rewarded_example"
        minSdkVersion 21  // 确保至少为 21
        targetSdkVersion 34
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionCode
    }
    
    // 其他配置...
}
```

### 3. 添加 AdMob App ID

在 `android/app/src/main/AndroidManifest.xml` 的 `<application>` 标签内添加 AdMob App ID：

```xml
<application>
    <!-- 其他配置 -->
    
    <meta-data
        android:name="com.google.android.gms.ads.APPLICATION_ID"
        android:value="ca-app-pub-3940256099942544~3347511713"/>
</application>
```

**注意**：这是测试 App ID。在生产环境中，你需要使用从 AdMob 控制台获取的真实 App ID。

## iOS 平台配置

### 1. 更新 Info.plist

打开 `ios/Runner/Info.plist` 文件，添加 AdMob App ID：

```xml
<key>GADApplicationIdentifier</key>
<string>ca-app-pub-3940256099942544~1458002511</string>
```

**注意**：这是测试 App ID。在生产环境中，你需要使用从 AdMob 控制台获取的真实 App ID。

### 2. 更新 Podfile

打开 `ios/Podfile` 文件，确保 `platform` 版本至少为 12.0：

```ruby
platform :ios, '12.0'
```

### 3. 安装 CocoaPods 依赖

在 `ios` 目录下运行：

```bash
cd ios
pod install
cd ..
```

## 获取测试广告单元 ID

在开发和测试阶段，Google 提供了测试广告单元 ID，你可以直接使用这些 ID 进行测试，无需创建真实的广告单元。

### 测试广告单元 ID

- **Android Rewarded Ads**: `ca-app-pub-3940256099942544/5224354917`
- **iOS Rewarded Ads**: `ca-app-pub-3940256099942544/1712485313`

### 测试 App ID

- **Android**: `ca-app-pub-3940256099942544~3347511713`
- **iOS**: `ca-app-pub-3940256099942544~1458002511`

### 重要提示

**始终使用测试广告进行开发和测试**

在构建和测试应用时，请确保使用测试广告而不是生产环境的真实广告。未遵循此要求可能导致账号被暂停。

## 获取生产环境广告单元 ID

当你准备发布应用时，需要从 AdMob 控制台获取真实的广告单元 ID：

1. 登录 [Google AdMob 控制台](https://apps.admob.com/)
2. 选择或创建你的应用
3. 导航到「广告单元」页面
4. 点击「创建广告单元」
5. 选择「Rewarded」广告格式
6. 为广告单元命名
7. 复制生成的广告单元 ID

### 创建 Rewarded 广告单元

在 AdMob 控制台中：

1. 选择「应用」→「广告单元」
2. 点击「创建广告单元」
3. 选择「Rewarded」
4. 输入广告单元名称（例如："Rewarded - Coins"）
5. 点击「创建」
6. 复制广告单元 ID（格式：`ca-app-pub-XXXXXXXXXXXXXXXX/XXXXXXXXXX`）

## 验证配置

### 检查依赖安装

运行以下命令检查依赖是否正确安装：

```bash
flutter pub deps
```

你应该能看到 `google_mobile_ads` 包及其依赖项。

### 检查平台配置

#### Android

检查 `android/app/src/main/AndroidManifest.xml` 是否包含：

- Internet 权限
- AdMob App ID meta-data

#### iOS

检查 `ios/Runner/Info.plist` 是否包含：

- `GADApplicationIdentifier` 键和值

### 运行测试

创建一个简单的测试文件来验证配置：

```dart
// test_config.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

void main() {
  test('Google Mobile Ads package is available', () {
    expect(MobileAds.instance, isNotNull);
  });
}
```

运行测试：

```bash
flutter test test_config.dart
```

## 常见配置问题

### Android 配置问题

**问题**：`minSdkVersion` 太低

**解决方案**：将 `minSdkVersion` 更新为至少 21

**问题**：缺少 Internet 权限

**解决方案**：在 `AndroidManifest.xml` 中添加 Internet 权限

### iOS 配置问题

**问题**：Pod 安装失败

**解决方案**：

```bash
cd ios
pod deintegrate
pod install
cd ..
```

**问题**：缺少 AdMob App ID

**解决方案**：在 `Info.plist` 中添加 `GADApplicationIdentifier`

## 实践练习

完成以下练习以巩固本章内容：

1. **创建项目**：创建一个新的 Flutter 项目（如果还没有）
2. **添加依赖**：在 `pubspec.yaml` 中添加 `google_mobile_ads` 依赖
3. **配置 Android**：更新 `AndroidManifest.xml` 和 `build.gradle`
4. **配置 iOS**：更新 `Info.plist` 和 `Podfile`
5. **验证配置**：运行 `flutter pub get` 和 `flutter doctor` 确保一切正常

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 创建或配置 Flutter 项目以支持 Google Mobile Ads
- [ ] 在 `pubspec.yaml` 中添加 `google_mobile_ads` 依赖
- [ ] 配置 Android 平台的权限和 AdMob App ID
- [ ] 配置 iOS 平台的 AdMob App ID
- [ ] 获取并使用测试广告单元 ID
- [ ] 了解如何获取生产环境的广告单元 ID
- [ ] 验证项目配置是否正确

## 下一步

配置完成后，我们就可以开始编写代码了。在下一章中，我们将学习如何使用 `RewardedAd.load()` 方法加载激励广告。

继续学习：[第 3 章：加载激励广告](chapter-03-loading-ads.md)
