# 第 7 章：广告过期处理

## 章节简介

在本章中，我们将学习如何处理 App Open 广告的过期问题。App Open Ads 有一个重要的限制：广告在加载后 4 小时内有效，超过这个时间后广告将不再有效，可能无法获得收益。我们将实现时间戳记录、过期检测和自动重新加载机制。

## 广告过期限制

### 4 小时限制

根据 Google Mobile Ads SDK 的文档，App Open Ads 有一个重要的时间限制：

**关键点**：广告引用在加载后 4 小时会过期。超过 4 小时后渲染的广告将不再有效，可能无法获得收益。

这个时间限制在 App Open Ads 的测试版本中被仔细考虑，可能在未来的版本中会有所变化。

### 为什么需要过期检测？

如果不检测广告是否过期，可能会遇到以下问题：

1. **收益损失**：过期的广告可能无法获得收益
2. **用户体验差**：尝试显示过期广告可能导致错误
3. **资源浪费**：持有过期的广告对象占用内存

## 实现过期检测

### 1. 添加时间戳变量

在 `AppOpenAdManager` 类中添加时间戳变量来记录广告加载时间：

```dart
class AppOpenAdManager {
  /// 最大缓存持续时间（4 小时）
  final Duration maxCacheDuration = const Duration(hours: 4);

  /// 记录广告加载时间，用于检测是否过期
  DateTime? _appOpenLoadTime;

  AppOpenAd? _appOpenAd;
  bool _isShowingAd = false;

  // 其他代码...
}
```

**代码说明**：

- `maxCacheDuration`：定义最大缓存持续时间（4 小时）
- `_appOpenLoadTime`：记录广告加载的时间戳，用于后续的过期检测

### 2. 在加载成功时记录时间

更新 `loadAd()` 方法，在广告加载成功时记录时间戳：

```dart
void loadAd() {
  AppOpenAd.load(
    adUnitId: adUnitId,
    request: const AdRequest(),
    adLoadCallback: AppOpenAdLoadCallback(
      onAdLoaded: (ad) {
        debugPrint('$ad loaded');
        _appOpenLoadTime = DateTime.now(); // 记录加载时间
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

- 在 `onAdLoaded` 回调中，使用 `DateTime.now()` 记录当前时间
- 这个时间戳将用于后续的过期检测

### 3. 在展示前检测过期

更新 `showAdIfAvailable()` 方法，在展示广告之前检测是否过期：

```dart
void showAdIfAvailable() {
  if (!isAdAvailable) {
    debugPrint('Tried to show ad before available.');
    loadAd();
    return;
  }
  
  if (_isShowingAd) {
    debugPrint('Tried to show ad while already showing an ad.');
    return;
  }
  
  // 检测广告是否过期
  if (DateTime.now().subtract(maxCacheDuration).isAfter(_appOpenLoadTime!)) {
    debugPrint('Maximum cache duration exceeded. Loading another ad.');
    _appOpenAd!.dispose();
    _appOpenAd = null;
    _appOpenLoadTime = null;
    loadAd();
    return;
  }
  
  // 设置回调并展示广告
  _appOpenAd!.fullScreenContentCallback = FullScreenContentCallback(
    // ... 回调实现
  );
  
  _appOpenAd!.show();
}
```

**代码说明**：

- `DateTime.now().subtract(maxCacheDuration)`：计算 4 小时前的时间
- `isAfter(_appOpenLoadTime!)`：检查加载时间是否早于 4 小时前
- 如果过期，释放旧广告，清空引用，然后加载新广告

### 过期检测逻辑详解

让我们详细分析过期检测的逻辑：

```dart
DateTime.now().subtract(maxCacheDuration).isAfter(_appOpenLoadTime!)
```

**步骤分解**：

1. `DateTime.now()`：获取当前时间
2. `.subtract(maxCacheDuration)`：减去 4 小时，得到 4 小时前的时间
3. `.isAfter(_appOpenLoadTime!)`：检查这个时间是否晚于广告加载时间

**示例**：

- 当前时间：2024-01-01 10:00:00
- 4 小时前：2024-01-01 06:00:00
- 广告加载时间：2024-01-01 05:00:00
- 结果：06:00:00 晚于 05:00:00，所以广告已过期

## 完整的过期处理实现

以下是包含过期检测的完整 `AppOpenAdManager` 实现：

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'dart:io' show Platform;

/// Utility class that manages loading and showing app open ads.
class AppOpenAdManager {
  /// Maximum duration allowed between loading and showing the ad.
  final Duration maxCacheDuration = const Duration(hours: 4);

  /// Keep track of load time so we don't show an expired ad.
  DateTime? _appOpenLoadTime;

  AppOpenAd? _appOpenAd;
  bool _isShowingAd = false;

  String adUnitId = Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/9257395921'
      : 'ca-app-pub-3940256099942544/5575463023';

  /// Load an [AppOpenAd].
  void loadAd() {
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

  /// Whether an ad is available to be shown.
  bool get isAdAvailable {
    return _appOpenAd != null;
  }

  /// Shows the ad, if one exists and is not already being shown.
  ///
  /// If the previously cached ad has expired, this just loads and caches a
  /// new ad.
  void showAdIfAvailable() {
    if (!isAdAvailable) {
      debugPrint('Tried to show ad before available.');
      loadAd();
      return;
    }
    if (_isShowingAd) {
      debugPrint('Tried to show ad while already showing an ad.');
      return;
    }
    
    // 检测广告是否过期
    if (DateTime.now().subtract(maxCacheDuration).isAfter(_appOpenLoadTime!)) {
      debugPrint('Maximum cache duration exceeded. Loading another ad.');
      _appOpenAd!.dispose();
      _appOpenAd = null;
      _appOpenLoadTime = null;
      loadAd();
      return;
    }
    
    // Set the fullScreenContentCallback and show the ad.
    _appOpenAd!.fullScreenContentCallback = FullScreenContentCallback(
      onAdShowedFullScreenContent: (ad) {
        _isShowingAd = true;
        debugPrint('$ad onAdShowedFullScreenContent');
      },
      onAdFailedToShowFullScreenContent: (ad, error) {
        debugPrint('$ad onAdFailedToShowFullScreenContent: $error');
        _isShowingAd = false;
        ad.dispose();
        _appOpenAd = null;
        _appOpenLoadTime = null;
      },
      onAdDismissedFullScreenContent: (ad) {
        debugPrint('$ad onAdDismissedFullScreenContent');
        _isShowingAd = false;
        ad.dispose();
        _appOpenAd = null;
        _appOpenLoadTime = null;
        loadAd();
      },
    );
    _appOpenAd!.show();
  }
}
```

## 清理时间戳

注意在以下情况下需要清理时间戳：

1. **广告过期时**：在检测到过期并释放广告后
2. **广告展示失败时**：在 `onAdFailedToShowFullScreenContent` 中
3. **广告被关闭时**：在 `onAdDismissedFullScreenContent` 中

这样可以确保时间戳与广告对象的状态保持一致。

## 测试过期检测

### 测试方法 1：修改 maxCacheDuration

为了快速测试过期检测功能，可以临时将 `maxCacheDuration` 设置为很短的时间：

```dart
final Duration maxCacheDuration = const Duration(seconds: 10); // 仅用于测试
```

然后：

1. 加载广告
2. 等待 10 秒以上
3. 尝试显示广告
4. 应该看到过期检测触发，自动加载新广告

### 测试方法 2：模拟时间

在测试代码中，可以模拟时间流逝来测试过期检测：

```dart
// 仅用于测试
void testExpiration() {
  final manager = AppOpenAdManager();
  manager.loadAd();
  
  // 模拟时间流逝（仅用于测试，实际应用中不要这样做）
  // 在实际应用中，应该等待真实时间流逝
}
```

**注意**：在生产环境中，应该使用真实的时间检测，不要修改系统时间。

## 过期检测的最佳实践

### 1. 使用常量定义时间限制

```dart
final Duration maxCacheDuration = const Duration(hours: 4);
```

使用常量而不是硬编码数字，便于维护和修改。

### 2. 在多个地方检查

除了在 `showAdIfAvailable()` 中检查，还可以考虑：

- 在定期任务中检查并预加载新广告
- 在应用恢复时检查

### 3. 记录过期事件

可以记录过期事件用于分析：

```dart
if (DateTime.now().subtract(maxCacheDuration).isAfter(_appOpenLoadTime!)) {
  debugPrint('Maximum cache duration exceeded. Loading another ad.');
  // 可以在这里记录分析事件
  _appOpenAd!.dispose();
  _appOpenAd = null;
  _appOpenLoadTime = null;
  loadAd();
  return;
}
```

### 4. 考虑提前刷新

可以在广告接近过期时提前加载新广告：

```dart
bool get isAdNearExpiration {
  if (_appOpenLoadTime == null) return false;
  final timeSinceLoad = DateTime.now().difference(_appOpenLoadTime!);
  return timeSinceLoad > maxCacheDuration - const Duration(minutes: 30);
}
```

## 常见问题

### 为什么是 4 小时？

这是 Google Mobile Ads SDK 的当前限制。这个时间限制在测试版本中被仔细考虑，可能在未来的版本中会有所变化。建议关注官方文档的更新。

### 如果广告在 4 小时内没有显示会怎样？

如果广告在 4 小时内没有显示，它仍然有效。但如果超过 4 小时，应该检测并重新加载。

### 可以修改 maxCacheDuration 吗？

可以，但不建议。4 小时是 Google 的建议值，修改可能导致收益损失或其他问题。

### 如何知道广告是否过期？

通过比较当前时间和加载时间：

```dart
final timeSinceLoad = DateTime.now().difference(_appOpenLoadTime!);
if (timeSinceLoad > maxCacheDuration) {
  // 广告已过期
}
```

## 实践练习

完成以下练习以巩固本章内容：

1. **添加时间戳变量**：在 `AppOpenAdManager` 中添加 `_appOpenLoadTime` 变量
2. **记录加载时间**：在 `loadAd()` 的成功回调中记录时间戳
3. **实现过期检测**：在 `showAdIfAvailable()` 中添加过期检测逻辑
4. **清理时间戳**：确保在所有适当的地方清理时间戳
5. **测试过期检测**：临时修改 `maxCacheDuration` 为短时间，测试过期检测功能

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解 App Open Ads 的 4 小时过期限制
- [ ] 理解为什么需要过期检测
- [ ] 添加时间戳变量记录广告加载时间
- [ ] 在广告加载成功时记录时间戳
- [ ] 实现过期检测逻辑
- [ ] 在检测到过期时自动重新加载广告
- [ ] 在所有适当的地方清理时间戳
- [ ] 测试过期检测功能

## 下一步

现在我们已经实现了广告过期检测，确保不会显示过期的广告。在下一章中，我们将学习如何实现用户同意管理（UMP），以符合 GDPR 等隐私法规的要求。

继续学习：[第 8 章：用户同意管理（UMP）](chapter-08-consent-management.md)
