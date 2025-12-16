# 第 5 章 加载原生模板广告

## 引言

加载原生模板广告是集成过程的核心步骤。与平台特定实现不同，原生模板广告不需要 `factoryId`，只需要设置 `nativeTemplateStyle` 即可。本章将详细讲解如何在 Flutter 中加载原生模板广告，包括创建 `NativeAd`、配置样式、处理回调等。

## NativeAd 类概述

### 基本概念

`NativeAd` 是用于加载和显示原生广告的类。对于原生模板广告，我们需要：

1. 设置 `adUnitId`
2. 设置 `nativeTemplateStyle`（不需要 `factoryId`）
3. 设置 `listener` 处理事件
4. 调用 `load()` 方法加载广告

### 关键区别

与平台特定实现的关键区别：

| 特性 | 原生模板广告 | 平台特定实现 |
|------|------------|-------------|
| **factoryId** | 不需要 | 必需 |
| **nativeTemplateStyle** | 必需 | 不需要 |
| **原生代码** | 不需要 | 需要 |

## 基本加载实现

### 创建 NativeAd

```dart
NativeAd? _nativeAd;
bool _nativeAdIsLoaded = false;

void _loadAd() async {
  // 检查是否可以请求广告
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  setState(() {
    _nativeAdIsLoaded = false;
  });

  _nativeAd = NativeAd(
    adUnitId: _adUnitId,
    listener: NativeAdListener(
      onAdLoaded: (ad) {
        print('$NativeAd loaded.');
        setState(() {
          _nativeAdIsLoaded = true;
        });
      },
      onAdFailedToLoad: (ad, error) {
        print('$NativeAd failedToLoad: $error');
        ad.dispose();
      },
    ),
    request: const AdRequest(),
    nativeTemplateStyle: NativeTemplateStyle(
      templateType: TemplateType.medium,
      // ... 样式配置
    ),
  )..load();
}
```

## 设置 nativeTemplateStyle

### 完整样式配置

在创建 `NativeAd` 时，必须设置 `nativeTemplateStyle`：

```dart
_nativeAd = NativeAd(
  adUnitId: _adUnitId,
  nativeTemplateStyle: NativeTemplateStyle(
    templateType: TemplateType.medium,
    mainBackgroundColor: const Color(0xfffffbed),
    callToActionTextStyle: NativeTemplateTextStyle(
      textColor: Colors.white,
      style: NativeTemplateFontStyle.monospace,
      size: 16.0,
    ),
    primaryTextStyle: NativeTemplateTextStyle(
      textColor: Colors.black,
      style: NativeTemplateFontStyle.bold,
      size: 16.0,
    ),
    secondaryTextStyle: NativeTemplateTextStyle(
      textColor: Colors.black,
      style: NativeTemplateFontStyle.italic,
      size: 16.0,
    ),
    tertiaryTextStyle: NativeTemplateTextStyle(
      textColor: Colors.black,
      style: NativeTemplateFontStyle.normal,
      size: 16.0,
    ),
  ),
  // ... 其他配置
)..load();
```

## 广告状态管理

### 状态变量

管理广告的加载状态：

```dart
class NativeExampleState extends State<NativeExample> {
  NativeAd? _nativeAd;
  bool _nativeAdIsLoaded = false;
  
  // ...
}
```

### 更新状态

在广告加载成功或失败时更新状态：

```dart
onAdLoaded: (ad) {
  setState(() {
    _nativeAdIsLoaded = true;
  });
},

onAdFailedToLoad: (ad, error) {
  setState(() {
    _nativeAdIsLoaded = false;
  });
  ad.dispose();
},
```

## 完整的加载实现

以下是示例项目中的完整实现：

```dart
/// Loads a native ad.
void _loadAd() async {
  // Only load an ad if the Mobile Ads SDK has gathered consent aligned with
  // the app's configured messages.
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  setState(() {
    _nativeAdIsLoaded = false;
  });

  _nativeAd = NativeAd(
    adUnitId: _adUnitId,
    listener: NativeAdListener(
      onAdLoaded: (ad) {
        print('$NativeAd loaded.');
        setState(() {
          _nativeAdIsLoaded = true;
        });
      },
      onAdFailedToLoad: (ad, error) {
        print('$NativeAd failedToLoad: $error');
        ad.dispose();
      },
      onAdClicked: (ad) {},
      onAdImpression: (ad) {},
      onAdClosed: (ad) {},
      onAdOpened: (ad) {},
      onAdWillDismissScreen: (ad) {},
      onPaidEvent: (ad, valueMicros, precision, currencyCode) {},
    ),
    request: const AdRequest(),
    nativeTemplateStyle: NativeTemplateStyle(
      templateType: TemplateType.medium,
      mainBackgroundColor: const Color(0xfffffbed),
      callToActionTextStyle: NativeTemplateTextStyle(
        textColor: Colors.white,
        style: NativeTemplateFontStyle.monospace,
        size: 16.0,
      ),
      primaryTextStyle: NativeTemplateTextStyle(
        textColor: Colors.black,
        style: NativeTemplateFontStyle.bold,
        size: 16.0,
      ),
      secondaryTextStyle: NativeTemplateTextStyle(
        textColor: Colors.black,
        style: NativeTemplateFontStyle.italic,
        size: 16.0,
      ),
      tertiaryTextStyle: NativeTemplateTextStyle(
        textColor: Colors.black,
        style: NativeTemplateFontStyle.normal,
        size: 16.0,
      ),
    ),
  )..load();
}
```

## 广告单元 ID

### 测试广告单元 ID

在开发和测试阶段，使用测试广告单元 ID：

```dart
final String _adUnitId = Platform.isAndroid
    ? 'ca-app-pub-3940256099942544/2247696110'  // Android 测试 ID
    : 'ca-app-pub-3940256099942544/3986624511'; // iOS 测试 ID
```

### 生产环境

在生产环境中，使用你在 AdMob 控制台创建的真实广告单元 ID。

## 资源清理

### 在 dispose() 中清理

在组件销毁时，必须清理广告资源：

```dart
@override
void dispose() {
  _nativeAd?.dispose();
  _nativeAd = null;
  super.dispose();
}
```

### 加载新广告前清理

在加载新广告之前，清理旧的广告：

```dart
void _loadAd() async {
  // 清理旧广告
  _nativeAd?.dispose();
  _nativeAd = null;
  
  // 加载新广告
  _nativeAd = NativeAd(/* ... */)..load();
}
```

## 错误处理

### 处理加载失败

```dart
onAdFailedToLoad: (ad, error) {
  print('NativeAd failedToLoad:');
  print('  Error code: ${error.code}');
  print('  Error domain: ${error.domain}');
  print('  Error message: ${error.message}');
  
  // 清理资源
  ad.dispose();
  setState(() {
    _nativeAdIsLoaded = false;
    _nativeAd = null;
  });
  
  // 可以在这里实现重试逻辑
},
```

### 常见错误

- **错误代码 0**：通常表示配置问题（如应用 ID 未设置）
- **错误代码 3**：网络错误
- **错误代码 8**：内部错误

## 实践练习

1. 实现 `_loadAd()` 方法
2. 配置 `nativeTemplateStyle`
3. 实现 `NativeAdListener` 的回调
4. 管理广告状态
5. 实现资源清理逻辑

## 常见问题

### Q: 为什么不需要 `factoryId`？

A: 原生模板广告使用 Google 提供的预定义模板，SDK 会自动处理渲染，因此不需要在原生层注册工厂。

### Q: 可以同时加载多个原生模板广告吗？

A: 可以。每个广告需要使用不同的 `NativeAd` 实例。

### Q: 如果广告加载失败怎么办？

A: 检查错误信息，确认配置是否正确（应用 ID、广告单元 ID 等），然后可以尝试重新加载。

### Q: 样式配置是必需的吗？

A: `templateType` 是必需的，其他样式参数是可选的。如果不配置，将使用默认样式。

## 总结与检查清单

### 本章要点

- 使用 `NativeAd` 类加载原生模板广告
- 必须设置 `nativeTemplateStyle`（不需要 `factoryId`）
- 通过 `NativeAdListener` 处理广告事件
- 及时清理广告资源

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `NativeAd` 类的使用
- [ ] 如何设置 `nativeTemplateStyle`
- [ ] `NativeAdListener` 的基本回调
- [ ] 如何清理广告资源

下一章，我们将学习如何显示原生模板广告。
