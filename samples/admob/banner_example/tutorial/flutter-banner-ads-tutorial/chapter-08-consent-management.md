# 第 8 章：用户同意管理（UMP）

## 章节简介

在本章中，我们将学习如何实现用户同意管理（User Messaging Platform，UMP），以符合 GDPR 等隐私法规的要求。我们将使用 `ConsentManager` 类，学习如何获取用户同意、检查是否可以请求广告，以及显示隐私选项表单。这与 App Open Ads 的同意管理类似，但我们将结合 Banner Ads 的具体实现。

## 为什么需要用户同意管理

### 隐私法规要求

在 GDPR（通用数据保护条例）等隐私法规的影响下，应用需要：

1. **获取用户同意**：在收集或使用用户数据之前获取明确同意
2. **提供选择**：允许用户选择是否接受个性化广告
3. **易于访问**：提供简单的方式让用户修改隐私设置

### Google 的 UMP 解决方案

Google Mobile Ads SDK 提供了 User Messaging Platform（UMP），这是 Google 的 IAB 认证同意管理平台，用于在受 GDPR 影响的国家/地区捕获用户同意。

**注意**：UMP 是一个示例解决方案，你可以选择其他同意管理平台。

## ConsentManager 类

### 查看现有实现

在 `banner_example` 项目中，`ConsentManager` 类已经实现。让我们查看它的结构：

```dart
import 'dart:async';
import 'package:google_mobile_ads/google_mobile_ads.dart';

typedef OnConsentGatheringCompleteListener = void Function(FormError? error);

class ConsentManager {
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
    ConsentDebugSettings debugSettings = ConsentDebugSettings(
      // debugGeography: DebugGeography.debugGeographyEea,
    );
    ConsentRequestParameters params = ConsentRequestParameters(
      consentDebugSettings: debugSettings,
    );

    ConsentInformation.instance.requestConsentInfoUpdate(
      params,
      () async {
        ConsentForm.loadAndShowConsentFormIfRequired((loadAndShowError) {
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

## 在 Banner Ads 中集成同意管理

### 在加载广告前检查同意

在 `_loadAd()` 方法中，我们应该先检查用户是否同意：

```dart
final _consentManager = ConsentManager();

void _loadAd() async {
  // 检查是否可以请求广告
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  // 检查组件是否仍然挂载
  if (!mounted) {
    return;
  }

  // 获取广告尺寸并加载广告
  final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
    MediaQuery.sizeOf(context).width.truncate(),
  );

  if (size == null) {
    return;
  }

  BannerAd(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    size: size,
    listener: BannerAdListener(
      // ... 监听器实现
    ),
  ).load();
}
```

### 在应用启动时获取同意

在 `initState()` 中获取用户同意：

```dart
@override
void initState() {
  super.initState();

  // 获取用户同意
  _consentManager.gatherConsent((consentGatheringError) {
    if (consentGatheringError != null) {
      debugPrint(
        "${consentGatheringError.errorCode}: ${consentGatheringError.message}",
      );
    }

    // 检查是否需要隐私选项入口
    _getIsPrivacyOptionsRequired();

    // 尝试初始化 Mobile Ads SDK
    _initializeMobileAdsSDK();
  });

  // 尝试使用之前会话的同意信息
  _initializeMobileAdsSDK();
}
```

### 初始化 Mobile Ads SDK

只有在获得同意后才初始化 SDK：

```dart
var _isMobileAdsInitializeCalled = false;

void _initializeMobileAdsSDK() async {
  if (_isMobileAdsInitializeCalled) {
    return;
  }

  if (await _consentManager.canRequestAds()) {
    _isMobileAdsInitializeCalled = true;

    // 初始化 Mobile Ads SDK
    MobileAds.instance.initialize();

    // 加载广告
    _loadAd();
  }
}
```

### 显示隐私选项入口

如果需要，在应用栏中显示隐私选项入口：

```dart
var _isPrivacyOptionsRequired = false;

void _getIsPrivacyOptionsRequired() async {
  if (await _consentManager.isPrivacyOptionsRequired()) {
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
            _consentManager.showPrivacyOptionsForm((formError) {
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

## 同意流程

### 首次启动

1. 应用启动
2. 调用 `gatherConsent()`
3. 如果需要，显示同意表单
4. 用户做出选择
5. 根据选择决定是否初始化 SDK 和加载广告

### 后续启动

1. 应用启动
2. 调用 `gatherConsent()`（检查同意状态是否变化）
3. 如果之前已同意，直接初始化 SDK 和加载广告
4. 如果需要，显示隐私选项入口

## 常见问题

### 什么时候调用 gatherConsent()？

应该在每次应用启动时调用 `gatherConsent()`，以检查同意状态是否变化。

### 如果用户拒绝同意会怎样？

如果用户拒绝同意，`canRequestAds()` 将返回 `false`，应用不应该请求广告。

### 如何测试同意流程？

使用 `DebugGeography` 来模拟不同地区，测试同意表单的显示。

## 实践练习

完成以下练习以巩固本章内容：

1. **集成同意检查**：在 `_loadAd()` 中添加同意检查
2. **实现同意获取**：在 `initState()` 中获取用户同意
3. **实现 SDK 初始化**：只有在获得同意后才初始化 SDK
4. **测试同意流程**：使用 `DebugGeography` 测试同意表单的显示

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解为什么需要用户同意管理
- [ ] 理解 UMP 的作用和用途
- [ ] 在加载广告前检查用户同意
- [ ] 在应用启动时获取用户同意
- [ ] 只有在获得同意后才初始化 SDK
- [ ] 显示隐私选项入口
- [ ] 理解同意流程和调试设置

## 下一步

现在我们已经实现了用户同意管理，确保应用符合隐私法规要求。在下一章中，我们将整合所有知识，创建一个完整的集成示例。

继续学习：[第 9 章：完整集成示例](chapter-09-complete-integration.md)
