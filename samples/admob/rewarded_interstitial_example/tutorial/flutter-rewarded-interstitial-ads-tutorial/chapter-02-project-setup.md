# 第 2 章 项目配置与依赖

## 引言

在开始集成 Rewarded Interstitial Ads 之前，我们需要正确配置 Flutter 项目。本章将指导你完成所有必要的配置步骤，包括添加依赖、配置 Android 和 iOS 平台设置。

## 添加依赖

首先，我们需要在 `pubspec.yaml` 文件中添加 `google_mobile_ads` 依赖。

### 修改 pubspec.yaml

打开项目的 `pubspec.yaml` 文件，在 `dependencies` 部分添加 `google_mobile_ads`：

```yaml
dependencies:
  flutter:
    sdk: flutter
  google_mobile_ads: ^6.0.0
```

### 安装依赖

在终端中运行以下命令安装依赖：

```bash
flutter pub get
```

这将下载并安装 `google_mobile_ads` 包及其所有依赖项。

## Android 平台配置

### 1. 添加权限

打开 `android/app/src/main/AndroidManifest.xml` 文件，确保包含以下权限：

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```

这些权限通常已经包含在 Flutter 项目中，但请确认它们存在。

### 2. 添加 AdMob 应用 ID

在 `AndroidManifest.xml` 的 `<application>` 标签内，添加 AdMob 应用 ID：

```xml
<application
    android:label="rewarded_interstitial_example"
    android:name="${applicationName}"
    android:icon="@mipmap/ic_launcher">
    <!-- 其他配置 -->
    
    <meta-data
        android:name="com.google.android.gms.ads.APPLICATION_ID"
        android:value="ca-app-pub-3940256099942544~3347511713"/>
</application>
```

**重要提示**：`ca-app-pub-3940256099942544~3347511713` 是 Google 提供的测试应用 ID。在生产环境中，你需要将其替换为你在 AdMob 控制台创建的实际应用 ID。

### 3. 最低 SDK 版本

确保 `android/app/build.gradle` 中的 `minSdkVersion` 至少为 19：

```gradle
android {
    defaultConfig {
        minSdkVersion 19
    }
}
```

## iOS 平台配置

### 1. 添加 AdMob 应用 ID

打开 `ios/Runner/Info.plist` 文件，添加 `GADApplicationIdentifier` 键：

```xml
<key>GADApplicationIdentifier</key>
<string>ca-app-pub-3940256099942544~1458002511</string>
```

**重要提示**：`ca-app-pub-3940256099942544~1458002511` 是 Google 提供的测试应用 ID。在生产环境中，你需要将其替换为你在 AdMob 控制台创建的实际应用 ID。

### 2. SKAdNetworkItems（可选但推荐）

为了支持 iOS 14+ 的 SKAdNetwork，建议在 `Info.plist` 中添加 `SKAdNetworkItems` 数组。这有助于广告归因和优化。示例项目已经包含了完整的 SKAdNetwork 配置。

## 测试广告单元 ID

在开发和测试阶段，使用 Google 提供的测试广告单元 ID：

- **Android 测试广告单元 ID**：`ca-app-pub-3940256099942544/5354046379`
- **iOS 测试广告单元 ID**：`ca-app-pub-3940256099942544/6978759866`

**重要**：在开发和测试时，必须使用测试广告单元 ID。使用真实广告单元 ID 进行测试可能导致你的 AdMob 账户被暂停。

### 在代码中使用测试广告单元 ID

在 `main.dart` 中，我们可以这样定义广告单元 ID：

```dart
final String _adUnitId = Platform.isAndroid
    ? 'ca-app-pub-3940256099942544/5354046379'
    : 'ca-app-pub-3940256099942544/6978759866';
```

## 验证配置

完成配置后，运行以下命令验证项目配置是否正确：

```bash
flutter doctor
```

确保所有必要的工具都已正确安装和配置。

## 实践练习

1. 在你的项目中添加 `google_mobile_ads` 依赖
2. 配置 Android 平台的 `AndroidManifest.xml`
3. 配置 iOS 平台的 `Info.plist`
4. 验证配置是否正确

## 常见问题

### Q: 我应该在哪里获取真实的 AdMob 应用 ID？

A: 登录 [AdMob 控制台](https://apps.admob.com/)，创建应用后，你可以在应用设置中找到应用 ID。

### Q: 测试广告单元 ID 和真实广告单元 ID 有什么区别？

A: 测试广告单元 ID 专门用于开发和测试，不会产生真实的广告收益。真实广告单元 ID 用于生产环境，会产生真实的广告展示和收益。

### Q: 如果我不配置 AdMob 应用 ID 会怎样？

A: 应用可能无法正确加载广告，或者会显示错误信息。配置应用 ID 是使用 Google Mobile Ads SDK 的必要步骤。

## 总结与检查清单

### 本章要点

- 在 `pubspec.yaml` 中添加 `google_mobile_ads` 依赖
- 在 Android 的 `AndroidManifest.xml` 中配置应用 ID
- 在 iOS 的 `Info.plist` 中配置应用 ID
- 使用测试广告单元 ID 进行开发和测试

### 检查清单

在继续下一章之前，确保你已完成：

- [ ] 在 `pubspec.yaml` 中添加了 `google_mobile_ads` 依赖
- [ ] 运行了 `flutter pub get` 安装依赖
- [ ] 在 Android `AndroidManifest.xml` 中配置了应用 ID
- [ ] 在 iOS `Info.plist` 中配置了应用 ID
- [ ] 验证了项目配置（`flutter doctor`）

配置完成后，我们就可以开始加载广告了！
