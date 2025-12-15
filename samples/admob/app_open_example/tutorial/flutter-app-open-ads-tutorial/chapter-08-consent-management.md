# 第 8 章：用户同意管理（UMP）

## 章节简介

在本章中，我们将学习如何实现用户同意管理（User Messaging Platform，UMP），以符合 GDPR 等隐私法规的要求。我们将创建 `ConsentManager` 类，学习如何获取用户同意、检查是否可以请求广告，以及显示隐私选项表单。

## 为什么需要用户同意管理

### 隐私法规要求

在 GDPR（通用数据保护条例）等隐私法规的影响下，应用需要：

1. **获取用户同意**：在收集或使用用户数据之前获取明确同意
2. **提供选择**：允许用户选择是否接受个性化广告
3. **易于访问**：提供简单的方式让用户修改隐私设置

### Google 的 UMP 解决方案

Google Mobile Ads SDK 提供了 User Messaging Platform（UMP），这是 Google 的 IAB 认证同意管理平台，用于在受 GDPR 影响的国家/地区捕获用户同意。

**注意**：UMP 是一个示例解决方案，你可以选择其他同意管理平台。

## 创建 ConsentManager 类

### 1. 创建文件

在 `lib` 目录下创建新文件 `consent_manager.dart`：

```dart
import 'dart:async';
import 'package:google_mobile_ads/google_mobile_ads.dart';

typedef OnConsentGatheringCompleteListener = void Function(FormError? error);

/// The Google Mobile Ads SDK provides the User Messaging Platform (Google's IAB
/// Certified consent management platform) as one solution to capture consent for
/// users in GDPR impacted countries. This is an example and you can choose
/// another consent management platform to capture consent.
class ConsentManager {
  ConsentManager._();
  static final ConsentManager instance = ConsentManager._();
}
```

**代码说明**：

- 使用单例模式（Singleton Pattern）
- `ConsentManager._()` 是私有构造函数
- `ConsentManager.instance` 是唯一的实例

### 2. 实现 canRequestAds() 方法

添加方法来检查是否可以请求广告：

```dart
/// Helper variable to determine if the app can request ads.
Future<bool> canRequestAds() async {
  return await ConsentInformation.instance.canRequestAds();
}
```

**代码说明**：

- `ConsentInformation.instance.canRequestAds()` 返回一个 `Future<bool>`
- 如果用户已同意或不需要同意，返回 `true`
- 如果用户未同意，返回 `false`

### 3. 实现 isPrivacyOptionsRequired() 方法

添加方法来检查是否需要显示隐私选项入口：

```dart
/// Helper variable to determine if the privacy options form is required.
Future<bool> isPrivacyOptionsRequired() async {
  return await ConsentInformation.instance
          .getPrivacyOptionsRequirementStatus() ==
      PrivacyOptionsRequirementStatus.required;
}
```

**代码说明**：

- 检查是否需要显示隐私选项表单
- 如果返回 `true`，应该在应用中提供访问隐私设置的入口

### 4. 实现 gatherConsent() 方法

添加方法来获取用户同意：

```dart
/// Helper method to call the Mobile Ads SDK to request consent information
/// and load/show a consent form if necessary.
void gatherConsent(
  OnConsentGatheringCompleteListener onConsentGatheringCompleteListener,
) {
  // For testing purposes, you can force a DebugGeography of Eea or NotEea.
  ConsentDebugSettings debugSettings = ConsentDebugSettings(
    // debugGeography: DebugGeography.debugGeographyEea,
  );
  ConsentRequestParameters params = ConsentRequestParameters(
    consentDebugSettings: debugSettings,
  );

  // Requesting an update to consent information should be called on every app launch.
  ConsentInformation.instance.requestConsentInfoUpdate(
    params,
    () async {
      ConsentForm.loadAndShowConsentFormIfRequired((loadAndShowError) {
        // Consent has been gathered.
        onConsentGatheringCompleteListener(loadAndShowError);
      });
    },
    (FormError formError) {
      onConsentGatheringCompleteListener(formError);
    },
  );
}
```

**代码说明**：

- `ConsentDebugSettings`：用于测试的调试设置，可以强制设置地理位置
- `ConsentRequestParameters`：同意请求参数
- `requestConsentInfoUpdate`：请求更新同意信息（应在每次应用启动时调用）
- `loadAndShowConsentFormIfRequired`：如果需要，加载并显示同意表单

### 5. 实现 showPrivacyOptionsForm() 方法

添加方法来显示隐私选项表单：

```dart
/// Helper method to call the Mobile Ads SDK method to show the privacy options form.
void showPrivacyOptionsForm(
  OnConsentFormDismissedListener onConsentFormDismissedListener,
) {
  ConsentForm.showPrivacyOptionsForm(onConsentFormDismissedListener);
}
```

**代码说明**：

- 显示隐私选项表单，允许用户修改隐私设置
- 当表单关闭时，调用 `onConsentFormDismissedListener` 回调

## 完整的 ConsentManager 实现

以下是完整的 `ConsentManager` 类实现：

```dart
import 'dart:async';
import 'package:google_mobile_ads/google_mobile_ads.dart';

typedef OnConsentGatheringCompleteListener = void Function(FormError? error);

/// The Google Mobile Ads SDK provides the User Messaging Platform (Google's IAB
/// Certified consent management platform) as one solution to capture consent for
/// users in GDPR impacted countries. This is an example and you can choose
/// another consent management platform to capture consent.
class ConsentManager {
  ConsentManager._();
  static final ConsentManager instance = ConsentManager._();

  /// Helper variable to determine if the app can request ads.
  Future<bool> canRequestAds() async {
    return await ConsentInformation.instance.canRequestAds();
  }

  /// Helper variable to determine if the privacy options form is required.
  Future<bool> isPrivacyOptionsRequired() async {
    return await ConsentInformation.instance
            .getPrivacyOptionsRequirementStatus() ==
        PrivacyOptionsRequirementStatus.required;
  }

  /// Helper method to call the Mobile Ads SDK to request consent information
  /// and load/show a consent form if necessary.
  void gatherConsent(
    OnConsentGatheringCompleteListener onConsentGatheringCompleteListener,
  ) {
    // For testing purposes, you can force a DebugGeography of Eea or NotEea.
    ConsentDebugSettings debugSettings = ConsentDebugSettings(
      // debugGeography: DebugGeography.debugGeographyEea,
    );
    ConsentRequestParameters params = ConsentRequestParameters(
      consentDebugSettings: debugSettings,
    );

    // Requesting an update to consent information should be called on every app launch.
    ConsentInformation.instance.requestConsentInfoUpdate(
      params,
      () async {
        ConsentForm.loadAndShowConsentFormIfRequired((loadAndShowError) {
          // Consent has been gathered.
          onConsentGatheringCompleteListener(loadAndShowError);
        });
      },
      (FormError formError) {
        onConsentGatheringCompleteListener(formError);
      },
    );
  }

  /// Helper method to call the Mobile Ads SDK method to show the privacy options form.
  void showPrivacyOptionsForm(
    OnConsentFormDismissedListener onConsentFormDismissedListener,
  ) {
    ConsentForm.showPrivacyOptionsForm(onConsentFormDismissedListener);
  }
}
```

## 在应用中集成同意管理

### 更新 AppOpenAdManager

更新 `AppOpenAdManager` 的 `loadAd()` 方法，在加载广告前检查同意状态：

```dart
import 'package:app_open_example/consent_manager.dart';

void loadAd() async {
  // Only load an ad if the Mobile Ads SDK has gathered consent aligned with
  // the app's configured messages.
  var canRequestAds = await ConsentManager.instance.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  AppOpenAd.load(
    adUnitId: adUnitId,
    request: const AdRequest(),
    adLoadCallback: AppOpenAdLoadCallback(
      onAdLoaded: (ad) {
        debugPrint('$ad loaded');
        _appOpenLoadTime = DateTime.now();
        _appOpenAd = ad;
      },
      onAdFailedToLoad: (error) {
        debugPrint('AppOpenAd failed to load: $error');
      },
    ),
  );
}
```

**代码说明**：

- 在加载广告前，先检查是否可以请求广告
- 如果用户未同意，不加载广告

### 在 main.dart 中集成

在应用启动时获取用户同意：

```dart
@override
void initState() {
  super.initState();

  _appLifecycleReactor = AppLifecycleReactor(
    appOpenAdManager: _appOpenAdManager,
  );
  _appLifecycleReactor.listenToAppStateChanges();

  // 获取用户同意
  ConsentManager.instance.gatherConsent((consentGatheringError) {
    if (consentGatheringError != null) {
      // Consent not obtained in current session.
      debugPrint(
        "${consentGatheringError.errorCode}: ${consentGatheringError.message}",
      );
    }

    // Check if a privacy options entry point is required.
    _getIsPrivacyOptionsRequired();

    // Attempt to initialize the Mobile Ads SDK.
    _initializeMobileAdsSDK();
  });

  // This sample attempts to load ads using consent obtained in the previous session.
  _initializeMobileAdsSDK();
}
```

### 初始化 Mobile Ads SDK

只有在获得同意后才初始化 SDK：

```dart
/// Initialize the Mobile Ads SDK if the SDK has gathered consent aligned with
/// the app's configured messages.
void _initializeMobileAdsSDK() async {
  if (_isMobileAdsInitializeCalled) {
    return;
  }

  if (await ConsentManager.instance.canRequestAds()) {
    _isMobileAdsInitializeCalled = true;

    // Initialize the Mobile Ads SDK.
    MobileAds.instance.initialize();

    // Load an ad.
    _appOpenAdManager.loadAd();
  }
}
```

**代码说明**：

- 使用 `_isMobileAdsInitializeCalled` 标志防止重复初始化
- 只有在 `canRequestAds()` 返回 `true` 时才初始化 SDK
- 初始化后立即加载广告

### 显示隐私选项入口

如果需要，在应用栏中显示隐私选项入口：

```dart
var _isPrivacyOptionsRequired = false;

void _getIsPrivacyOptionsRequired() async {
  if (await ConsentManager.instance.isPrivacyOptionsRequired()) {
    setState(() {
      _isPrivacyOptionsRequired = true;
    });
  }
}

List<Widget> _appBarActions() {
  var array = [AppBarItem(AppBarItem.adInpsectorText, 0)];

  if (_isPrivacyOptionsRequired) {
    array.add(AppBarItem(AppBarItem.privacySettingsText, 1));
  }

  return <Widget>[
    PopupMenuButton<AppBarItem>(
      itemBuilder: (context) => array
          .map(
            (item) => PopupMenuItem<AppBarItem>(
              value: item,
              child: Text(item.label),
            ),
          )
          .toList(),
      onSelected: (item) {
        switch (item.value) {
          case 0:
            MobileAds.instance.openAdInspector((error) {
              // Error will be non-null if ad inspector closed due to an error.
            });
          case 1:
            ConsentManager.instance.showPrivacyOptionsForm((formError) {
              if (formError != null) {
                debugPrint("${formError.errorCode}: ${formError.message}");
              }
            });
        }
      },
    ),
  ];
}
```

## 调试设置

### DebugGeography

在测试时，可以使用 `DebugGeography` 来模拟不同地区：

```dart
ConsentDebugSettings debugSettings = ConsentDebugSettings(
  debugGeography: DebugGeography.debugGeographyEea, // 模拟欧洲地区
  // 或
  // debugGeography: DebugGeography.debugGeographyNotEea, // 模拟非欧洲地区
);
```

**注意**：仅在测试时使用，生产环境应注释掉。

### 测试设备 ID

可以添加测试设备 ID：

```dart
ConsentDebugSettings debugSettings = ConsentDebugSettings(
  debugGeography: DebugGeography.debugGeographyEea,
  testDeviceIds: ['YOUR_TEST_DEVICE_ID'],
);
```

## 同意流程

### 首次启动

1. 应用启动
2. 调用 `gatherConsent()`
3. 如果需要，显示同意表单
4. 用户做出选择
5. 根据选择决定是否初始化 SDK

### 后续启动

1. 应用启动
2. 调用 `gatherConsent()`（检查同意状态是否变化）
3. 如果之前已同意，直接初始化 SDK
4. 如果需要，显示隐私选项入口

## 常见问题

### 什么时候调用 gatherConsent()？

应该在每次应用启动时调用 `gatherConsent()`，以检查同意状态是否变化。

### 如果用户拒绝同意会怎样？

如果用户拒绝同意，`canRequestAds()` 将返回 `false`，应用不应该请求广告。

### 如何测试同意流程？

使用 `DebugGeography` 来模拟不同地区，测试同意表单的显示。

### 可以自定义同意表单吗？

UMP 提供的同意表单是标准化的，但你可以选择其他同意管理平台来自定义。

## 实践练习

完成以下练习以巩固本章内容：

1. **创建 ConsentManager**：创建 `consent_manager.dart` 文件并实现类
2. **实现方法**：实现 `canRequestAds()`、`isPrivacyOptionsRequired()`、`gatherConsent()` 和 `showPrivacyOptionsForm()` 方法
3. **更新 AppOpenAdManager**：在 `loadAd()` 中添加同意检查
4. **集成到应用**：在 `main.dart` 中集成同意管理
5. **测试同意流程**：使用 `DebugGeography` 测试同意表单的显示

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解为什么需要用户同意管理
- [ ] 理解 UMP 的作用和用途
- [ ] 创建 `ConsentManager` 单例类
- [ ] 实现 `canRequestAds()` 方法检查是否可以请求广告
- [ ] 实现 `isPrivacyOptionsRequired()` 方法检查是否需要隐私选项
- [ ] 实现 `gatherConsent()` 方法获取用户同意
- [ ] 实现 `showPrivacyOptionsForm()` 方法显示隐私选项表单
- [ ] 在 `AppOpenAdManager` 中集成同意检查
- [ ] 在应用中正确集成同意管理流程
- [ ] 理解同意流程和调试设置

## 下一步

现在我们已经实现了用户同意管理，确保应用符合隐私法规要求。在下一章中，我们将学习如何处理冷启动场景，优化用户体验。

继续学习：[第 9 章：冷启动与加载屏幕](chapter-09-cold-starts.md)
