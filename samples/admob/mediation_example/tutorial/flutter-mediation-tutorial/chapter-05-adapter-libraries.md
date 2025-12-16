# 第 5 章 添加中介适配器库

## 引言

适配器库是连接 Google Mobile Ads SDK 和第三方广告网络的桥梁。要在应用中使用某个广告网络，你需要在项目中添加对应的适配器库。本章将详细讲解如何在 Android 和 iOS 平台添加适配器库，以及如何配置网络特定的设置。

## 理解适配器库

### 什么是适配器库？

适配器库是一个中间层，负责：

- 将 Google Mobile Ads SDK 的广告请求转换为第三方网络的请求格式
- 处理第三方网络的响应并转换为 Google Mobile Ads SDK 的格式
- 管理第三方网络的 SDK 初始化和生命周期

### 适配器库命名

适配器库通常命名为：`com.google.ads.mediation:<network-name>-adapter`

例如：

- AppLovin: `com.google.ads.mediation:applovin-adapter`
- Unity Ads: `com.google.ads.mediation:unity-adapter`

## Android 平台配置

### 在 build.gradle 中添加依赖

打开 `android/app/build.gradle` 文件，在 `dependencies` 部分添加适配器库：

```gradle
dependencies {
    // Google Mobile Ads SDK
    implementation 'com.google.android.gms:play-services-ads:22.6.0'
    
    // 中介适配器库
    implementation 'com.google.ads.mediation:applovin:12.0.0.0'
    implementation 'com.google.ads.mediation:unity:4.12.0.0'
    implementation 'com.google.ads.mediation:ironsource:8.1.0.0'
    // ... 其他适配器
}
```

### 常见适配器库列表

以下是一些常用的适配器库及其最新版本（版本号可能会更新，请查看官方文档）：

```gradle
// AppLovin
implementation 'com.google.ads.mediation:applovin:12.0.0.0'

// Unity Ads
implementation 'com.google.ads.mediation:unity:4.12.0.0'

// IronSource
implementation 'com.google.ads.mediation:ironsource:8.1.0.0'

// Vungle
implementation 'com.google.ads.mediation:vungle:6.13.1.0'

// Chartboost
implementation 'com.google.ads.mediation:chartboost:9.6.0.0'

// AdColony
implementation 'com.google.ads.mediation:adcolony:4.8.0.16.0'

// Facebook Audience Network
implementation 'com.google.ads.mediation:facebook:6.16.0.0'

// Tapjoy
implementation 'com.google.ads.mediation:tapjoy:13.1.2.0'
```

### 同步项目

添加依赖后，同步 Gradle 项目：

```bash
cd android
./gradlew build
```

或者在 Android Studio 中点击"Sync Now"。

## iOS 平台配置

### 在 Podfile 中添加依赖

打开 `ios/Podfile` 文件，在 `target 'Runner' do` 部分添加适配器库：

```ruby
target 'Runner' do
  use_frameworks!
  use_modular_headers!

  flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))
  
  # Google Mobile Ads SDK
  pod 'Google-Mobile-Ads-SDK', '~> 11.0'
  
  # 中介适配器库
  pod 'GoogleMobileAdsMediationAppLovin', '~> 12.0'
  pod 'GoogleMobileAdsMediationUnityAds', '~> 4.12'
  pod 'GoogleMobileAdsMediationIronSource', '~> 8.1'
  # ... 其他适配器
end
```

### 常见适配器库列表

```ruby
# AppLovin
pod 'GoogleMobileAdsMediationAppLovin', '~> 12.0'

# Unity Ads
pod 'GoogleMobileAdsMediationUnityAds', '~> 4.12'

# IronSource
pod 'GoogleMobileAdsMediationIronSource', '~> 8.1'

# Vungle
pod 'GoogleMobileAdsMediationVungle', '~> 6.13'

# Chartboost
pod 'GoogleMobileAdsMediationChartboost', '~> 9.6'

# AdColony
pod 'GoogleMobileAdsMediationAdColony', '~> 4.8'

# Facebook Audience Network
pod 'GoogleMobileAdsMediationFacebook', '~> 6.16'

# Tapjoy
pod 'GoogleMobileAdsMediationTapjoy', '~> 13.1'
```

### 安装 Pods

添加依赖后，安装 Pods：

```bash
cd ios
pod install
```

## 网络特定配置

某些广告网络需要额外的配置，例如 SDK Key 或 App ID。

### AppLovin 配置

#### Android

在 `AndroidManifest.xml` 中添加：

```xml
<meta-data
    android:name="applovin.sdk.key"
    android:value="YOUR_APPLOVIN_SDK_KEY"/>
```

#### iOS

在 `Info.plist` 中添加：

```xml
<key>AppLovinSdkKey</key>
<string>YOUR_APPLOVIN_SDK_KEY</string>
```

或者在 `Podfile` 中配置：

```ruby
pod 'GoogleMobileAdsMediationAppLovin', '~> 12.0' do
  # AppLovin SDK Key
  ENV['APPLOVIN_SDK_KEY'] = 'YOUR_APPLOVIN_SDK_KEY'
end
```

### Unity Ads 配置

Unity Ads 通常不需要额外配置，但确保在 Unity Ads 控制台中正确配置了应用。

### IronSource 配置

IronSource 需要在控制台中配置应用，但通常不需要在代码中添加额外配置。

## 查找适配器库版本

### 官方文档

每个适配器的文档都包含最新的版本信息：

- [Android 适配器列表](https://developers.google.com/admob/android/mediate)
- [iOS 适配器列表](https://developers.google.com/admob/ios/mediate)

### 版本兼容性

确保适配器版本与 Google Mobile Ads SDK 版本兼容。通常，适配器文档会说明支持的 SDK 版本范围。

## 验证适配器安装

### 检查适配器状态

在应用初始化时，检查适配器是否已正确安装：

```dart
MobileAds.instance.initialize().then((initializationStatus) {
  initializationStatus.adapterStatuses.forEach((key, value) {
    if (value.initializationState == AdapterInitializationState.MISSING) {
      debugPrint('Warning: $key adapter is missing. Check your dependencies.');
    } else if (value.initializationState == AdapterInitializationState.READY) {
      debugPrint('$key adapter is ready.');
    }
  });
});
```

### 常见问题排查

如果适配器状态显示为 `MISSING`：

1. **检查依赖是否正确添加**：确认 `build.gradle` 或 `Podfile` 中已添加依赖
2. **同步项目**：确保已同步 Gradle 或安装 Pods
3. **检查版本兼容性**：确保适配器版本与 SDK 版本兼容
4. **清理并重建**：尝试清理项目并重新构建

## 实践练习

1. 在 Android 项目中添加至少一个适配器库
2. 在 iOS 项目中添加至少一个适配器库
3. 配置 AppLovin SDK Key（如果使用 AppLovin）
4. 验证适配器是否正确安装

## 常见问题

### Q: 我需要添加所有适配器库吗？

A: 不需要。只添加你在 AdMob 控制台中配置的网络对应的适配器库。

### Q: 适配器库会增加应用大小吗？

A: 是的，每个适配器库都会增加应用大小。只添加你需要的适配器。

### Q: 如何知道应该使用哪个版本的适配器？

A: 查看官方文档，选择与你的 Google Mobile Ads SDK 版本兼容的适配器版本。

### Q: 如果适配器初始化失败怎么办？

A: 检查网络特定配置（如 SDK Key）是否正确，查看日志了解具体错误信息。

## 总结与检查清单

### 本章要点

- 适配器库是连接 Google Mobile Ads SDK 和第三方网络的桥梁
- 在 Android 的 `build.gradle` 中添加适配器依赖
- 在 iOS 的 `Podfile` 中添加适配器依赖
- 某些网络需要额外的配置（如 AppLovin SDK Key）

### 检查清单

在继续下一章之前，确保你已完成：

- [ ] 在 Android 项目中添加了适配器库依赖
- [ ] 在 iOS 项目中添加了适配器库依赖
- [ ] 配置了网络特定的设置（如需要）
- [ ] 验证了适配器是否正确安装

下一章，我们将学习如何使用 Platform Channel 调用第三方 SDK API。
