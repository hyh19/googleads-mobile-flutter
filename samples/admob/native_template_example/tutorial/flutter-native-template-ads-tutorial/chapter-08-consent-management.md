# 第 8 章 用户同意管理

## 引言

对于面向欧洲经济区（EEA）和英国用户的应用，必须遵守 GDPR（通用数据保护条例）要求。Google 提供了 User Messaging Platform (UMP) 来帮助开发者收集和管理用户同意。本章将详细讲解如何集成 UMP 并实现用户同意管理，确保原生模板广告的加载符合 GDPR 要求。

## GDPR 合规要求

### 什么是 GDPR？

GDPR（General Data Protection Regulation）是欧盟的数据保护法规，要求：

1. **用户同意**：在收集或使用用户数据之前，必须获得用户明确同意
2. **透明性**：向用户清楚说明数据收集和使用方式
3. **用户控制**：用户必须能够随时撤回同意

### 为什么需要用户同意管理？

- **法律要求**：在 EEA 和英国，不遵守 GDPR 可能导致巨额罚款
- **应用商店要求**：Google Play 和 App Store 要求应用遵守相关法规
- **广告收益**：只有在获得用户同意后，才能展示个性化广告，从而获得更高收益

## ConsentManager 实现

示例项目中的 `ConsentManager` 类封装了 UMP 的功能：

```dart
import 'dart:async';
import 'package:google_mobile_ads/google_mobile_ads.dart';

typedef OnConsentGatheringCompleteListener = void Function(FormError? error);

class ConsentManager {
  /// 检查是否可以请求广告
  Future<bool> canRequestAds() async {
    return await ConsentInformation.instance.canRequestAds();
  }

  /// 检查是否需要显示隐私选项表单
  Future<bool> isPrivacyOptionsRequired() async {
    return await ConsentInformation.instance
            .getPrivacyOptionsRequirementStatus() ==
        PrivacyOptionsRequirementStatus.required;
  }

  /// 收集用户同意
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

  /// 显示隐私选项表单
  void showPrivacyOptionsForm(
    OnConsentFormDismissedListener onConsentFormDismissedListener,
  ) {
    ConsentForm.showPrivacyOptionsForm(onConsentFormDismissedListener);
  }
}
```

## 核心方法详解

### canRequestAds()

检查是否可以请求广告。只有在用户同意后，才能请求和显示广告：

```dart
Future<bool> canRequestAds() async {
  return await ConsentInformation.instance.canRequestAds();
}
```

使用示例：

```dart
void _loadAd() async {
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    debugPrint('Cannot request ads. User consent not obtained.');
    return;
  }
  
  // 加载广告
  _nativeAd = NativeAd(/* ... */)..load();
}
```

### isPrivacyOptionsRequired()

检查是否需要显示隐私选项表单。某些情况下，用户需要能够修改他们的同意选择：

```dart
Future<bool> isPrivacyOptionsRequired() async {
  return await ConsentInformation.instance
          .getPrivacyOptionsRequirementStatus() ==
      PrivacyOptionsRequirementStatus.required;
}
```

使用示例：

```dart
void _getIsPrivacyOptionsRequired() async {
  if (await _consentManager.isPrivacyOptionsRequired()) {
    setState(() {
      _isPrivacyOptionsRequired = true;
    });
  }
}
```

### gatherConsent()

收集用户同意。这是最重要的方法，应该在应用启动时调用：

```dart
void gatherConsent(
  OnConsentGatheringCompleteListener onConsentGatheringCompleteListener,
) {
  ConsentRequestParameters params = ConsentRequestParameters(
    consentDebugSettings: ConsentDebugSettings(),
  );

  ConsentInformation.instance.requestConsentInfoUpdate(
    params,
    () async {
      // 如果需要，加载并显示同意表单
      ConsentForm.loadAndShowConsentFormIfRequired((loadAndShowError) {
        onConsentGatheringCompleteListener(loadAndShowError);
      });
    },
    (FormError formError) {
      // 处理错误
      onConsentGatheringCompleteListener(formError);
    },
  );
}
```

### showPrivacyOptionsForm()

显示隐私选项表单，允许用户修改他们的同意选择：

```dart
void showPrivacyOptionsForm(
  OnConsentFormDismissedListener onConsentFormDismissedListener,
) {
  ConsentForm.showPrivacyOptionsForm(onConsentFormDismissedListener);
}
```

## 集成到应用中

### 在 initState() 中收集同意

在应用启动时，我们应该收集用户同意：

```dart
@override
void initState() {
  super.initState();
  
  final _consentManager = ConsentManager();
  
  _consentManager.gatherConsent((consentGatheringError) {
    if (consentGatheringError != null) {
      debugPrint(
        "${consentGatheringError.errorCode}: ${consentGatheringError.message}",
      );
    }
    
    // 检查是否需要显示隐私选项入口
    _getIsPrivacyOptionsRequired();
    
    // 初始化 Mobile Ads SDK
    _initializeMobileAdsSDK();
  });
  
  // 尝试使用之前会话中获得的同意
  _initializeMobileAdsSDK();
}
```

### 初始化 Mobile Ads SDK

只有在获得用户同意后，才能初始化 Mobile Ads SDK：

```dart
void _initializeMobileAdsSDK() async {
  if (_isMobileAdsInitializeCalled) {
    return;
  }

  if (await _consentManager.canRequestAds()) {
    _isMobileAdsInitializeCalled = true;
    MobileAds.instance.initialize();
    _loadAd();
  }
}
```

### 在加载广告前检查同意

在加载原生模板广告之前，检查用户同意状态：

```dart
void _loadAd() async {
  // 检查是否可以请求广告
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    debugPrint('Cannot request ads. User consent not obtained.');
    return;
  }

  // 加载广告
  _nativeAd = NativeAd(/* ... */)..load();
}
```

### 显示隐私选项入口

如果需要在应用栏中显示隐私选项入口：

```dart
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
              // 处理错误
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

## 完整集成示例

以下是完整的用户同意管理集成示例：

```dart
class NativeExampleState extends State<NativeExample> {
  final _consentManager = ConsentManager();
  var _isMobileAdsInitializeCalled = false;
  var _isPrivacyOptionsRequired = false;
  NativeAd? _nativeAd;
  bool _nativeAdIsLoaded = false;

  @override
  void initState() {
    super.initState();
    
    // 收集用户同意
    _consentManager.gatherConsent((consentGatheringError) {
      if (consentGatheringError != null) {
        debugPrint(
          "${consentGatheringError.errorCode}: ${consentGatheringError.message}",
        );
      }
      
      _getIsPrivacyOptionsRequired();
      _initializeMobileAdsSDK();
    });
    
    // 尝试使用之前会话中获得的同意
    _initializeMobileAdsSDK();
  }

  void _getIsPrivacyOptionsRequired() async {
    if (await _consentManager.isPrivacyOptionsRequired()) {
      setState(() {
        _isPrivacyOptionsRequired = true;
      });
    }
  }

  void _initializeMobileAdsSDK() async {
    if (_isMobileAdsInitializeCalled) {
      return;
    }

    if (await _consentManager.canRequestAds()) {
      _isMobileAdsInitializeCalled = true;
      MobileAds.instance.initialize();
      _loadAd();
    }
  }

  void _loadAd() async {
    var canRequestAds = await _consentManager.canRequestAds();
    if (!canRequestAds) {
      return;
    }

    // 加载原生模板广告
    _nativeAd = NativeAd(/* ... */)..load();
  }
}
```

## 调试设置

在开发和测试阶段，你可以使用调试设置来模拟不同地区：

```dart
ConsentDebugSettings debugSettings = ConsentDebugSettings(
  debugGeography: DebugGeography.debugGeographyEea, // 模拟 EEA 地区
  // 或者
  // debugGeography: DebugGeography.debugGeographyNotEea, // 模拟非 EEA 地区
);

ConsentRequestParameters params = ConsentRequestParameters(
  consentDebugSettings: debugSettings,
);
```

**重要**：在生产环境中，不要设置 `debugGeography`，让系统自动检测用户地区。

## 实践练习

1. 实现 `ConsentManager` 类
2. 在应用启动时收集用户同意
3. 在获得同意后初始化 Mobile Ads SDK
4. 在加载广告前检查同意状态
5. 实现隐私选项表单显示功能

## 常见问题

### Q: 如果用户拒绝同意，我还能显示广告吗？

A: 可以，但只能显示非个性化广告。非个性化广告的收益通常较低。

### Q: 我需要在每次应用启动时都显示同意表单吗？

A: 不需要。UMP 会记住用户的选择，只有在必要时才会显示表单。

### Q: 如何测试同意流程？

A: 使用 `DebugGeography` 来模拟不同地区，或添加测试设备 ID。

### Q: 如果我的应用不在 EEA 或英国，还需要实现同意管理吗？

A: 虽然法律上可能不需要，但实现同意管理仍然是一个好习惯，可以提高用户信任度。

## 总结与检查清单

### 本章要点

- GDPR 要求应用在收集用户数据前获得同意
- UMP 提供了收集和管理用户同意的工具
- 只有在获得用户同意后，才能请求和显示广告
- 用户必须能够随时修改他们的同意选择

### 检查清单

在继续下一章之前，确保你理解：

- [ ] GDPR 合规要求
- [ ] `ConsentManager` 的实现
- [ ] 如何在应用启动时收集同意
- [ ] 如何检查是否可以请求广告
- [ ] 如何显示隐私选项表单

下一章，我们将学习最佳实践和常见问题。
