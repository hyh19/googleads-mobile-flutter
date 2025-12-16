# 第 6 章 处理广告事件

## 引言

在广告的生命周期中，会发生各种事件，如广告显示、用户点击、广告关闭等。通过 `FullScreenContentCallback`，我们可以监听这些事件并做出相应的处理。本章将详细讲解如何处理所有广告事件，以及如何正确清理广告资源。

## FullScreenContentCallback 概述

`FullScreenContentCallback` 是一个回调接口，用于监听全屏广告的各种事件。它包含以下回调方法：

- `onAdShowedFullScreenContent`：广告显示全屏内容时触发
- `onAdImpression`：广告产生展示时触发
- `onAdFailedToShowFullScreenContent`：广告显示失败时触发
- `onAdDismissedFullScreenContent`：广告关闭时触发
- `onAdClicked`：用户点击广告时触发

## 设置 FullScreenContentCallback

在广告加载成功后，我们应该立即设置 `FullScreenContentCallback`：

```dart
RewardedInterstitialAd.load(
  adUnitId: _adUnitId,
  request: const AdRequest(),
  rewardedInterstitialAdLoadCallback: RewardedInterstitialAdLoadCallback(
    onAdLoaded: (ad) {
      // 设置全屏内容回调
      ad.fullScreenContentCallback = FullScreenContentCallback(
        onAdShowedFullScreenContent: (ad) {
          debugPrint('Ad showed full screen content.');
        },
        onAdImpression: (ad) {
          debugPrint('Ad recorded an impression.');
        },
        onAdFailedToShowFullScreenContent: (ad, err) {
          debugPrint('Ad failed to show: $err');
          ad.dispose();
        },
        onAdDismissedFullScreenContent: (ad) {
          debugPrint('Ad dismissed.');
          ad.dispose();
        },
        onAdClicked: (ad) {
          debugPrint('Ad was clicked.');
        },
      );

      _rewardedInterstitialAd = ad;
    },
    onAdFailedToLoad: (LoadAdError error) {
      debugPrint('Ad failed to load: $error');
    },
  ),
);
```

## 各个回调详解

### onAdShowedFullScreenContent

当广告成功显示全屏内容时，这个回调会被触发。通常用于：

- 记录广告显示事件
- 暂停游戏或应用逻辑
- 更新 UI 状态

```dart
onAdShowedFullScreenContent: (RewardedInterstitialAd ad) {
  debugPrint('Ad showed full screen content.');
  // 可以在这里暂停游戏逻辑
  _pauseGame();
},
```

### onAdImpression

当广告产生展示（impression）时，这个回调会被触发。展示是指广告被用户实际看到。通常用于：

- 记录广告展示统计
- 分析广告效果

```dart
onAdImpression: (RewardedInterstitialAd ad) {
  debugPrint('Ad recorded an impression.');
  // 可以在这里记录分析事件
  _analytics.logEvent('ad_impression');
},
```

### onAdFailedToShowFullScreenContent

当广告显示失败时，这个回调会被触发。**重要**：在这个回调中，你必须清理广告资源。

```dart
onAdFailedToShowFullScreenContent: (RewardedInterstitialAd ad, AdError err) {
  debugPrint('Ad failed to show full screen content: $err');
  debugPrint('Error code: ${err.code}');
  debugPrint('Error domain: ${err.domain}');
  debugPrint('Error message: ${err.message}');
  
  // 必须清理广告资源
  ad.dispose();
  _rewardedInterstitialAd = null;
  
  // 可以在这里加载新广告或显示错误消息
  _loadAd();
},
```

### onAdDismissedFullScreenContent

当用户关闭广告（通过点击关闭按钮或完成观看）时，这个回调会被触发。**重要**：在这个回调中，你必须清理广告资源，并可以加载下一个广告。

```dart
onAdDismissedFullScreenContent: (RewardedInterstitialAd ad) {
  debugPrint('Ad dismissed full screen content.');
  
  // 必须清理广告资源
  ad.dispose();
  _rewardedInterstitialAd = null;
  
  // 恢复游戏逻辑
  _resumeGame();
  
  // 加载下一个广告
  _loadAd();
},
```

### onAdClicked

当用户点击广告时，这个回调会被触发。通常用于：

- 记录用户交互
- 分析广告点击率

```dart
onAdClicked: (RewardedInterstitialAd ad) {
  debugPrint('Ad was clicked.');
  // 可以在这里记录分析事件
  _analytics.logEvent('ad_clicked');
},
```

## 完整的回调实现

以下是完整的 `FullScreenContentCallback` 实现，包含所有必要的处理逻辑：

```dart
ad.fullScreenContentCallback = FullScreenContentCallback(
  // 广告显示全屏内容
  onAdShowedFullScreenContent: (RewardedInterstitialAd ad) {
    debugPrint('Ad showed full screen content.');
    // 暂停游戏逻辑
    _pauseGame();
  },
  
  // 广告产生展示
  onAdImpression: (RewardedInterstitialAd ad) {
    debugPrint('Ad recorded an impression.');
  },
  
  // 广告显示失败
  onAdFailedToShowFullScreenContent: (RewardedInterstitialAd ad, AdError err) {
    debugPrint('Ad failed to show full screen content: $err');
    // 清理资源
    ad.dispose();
    _rewardedInterstitialAd = null;
    // 恢复游戏逻辑
    _resumeGame();
    // 加载新广告
    _loadAd();
  },
  
  // 广告关闭
  onAdDismissedFullScreenContent: (RewardedInterstitialAd ad) {
    debugPrint('Ad dismissed full screen content.');
    // 清理资源
    ad.dispose();
    _rewardedInterstitialAd = null;
    // 恢复游戏逻辑
    _resumeGame();
    // 加载下一个广告
    _loadAd();
  },
  
  // 用户点击广告
  onAdClicked: (RewardedInterstitialAd ad) {
    debugPrint('Ad was clicked.');
  },
);
```

## 资源清理

### 为什么需要清理资源？

广告对象占用内存资源。如果不及时清理，可能导致内存泄漏。在以下情况下，你必须调用 `ad.dispose()`：

1. 广告显示失败（`onAdFailedToShowFullScreenContent`）
2. 广告关闭（`onAdDismissedFullScreenContent`）
3. 应用退出或组件销毁时

### 清理资源的最佳实践

```dart
onAdDismissedFullScreenContent: (RewardedInterstitialAd ad) {
  // 1. 清理广告对象
  ad.dispose();
  
  // 2. 清空引用
  _rewardedInterstitialAd = null;
  
  // 3. 加载下一个广告（可选）
  _loadAd();
},
```

### 在 dispose() 中清理

在组件的 `dispose()` 方法中，也应该清理广告资源：

```dart
@override
void dispose() {
  _rewardedInterstitialAd?.dispose();
  _rewardedInterstitialAd = null;
  super.dispose();
}
```

## 错误处理

### AdError 对象

`AdError` 对象包含以下信息：

- `code`：错误代码（整数）
- `domain`：错误域（字符串）
- `message`：错误消息（字符串）

```dart
onAdFailedToShowFullScreenContent: (RewardedInterstitialAd ad, AdError err) {
  debugPrint('Error code: ${err.code}');
  debugPrint('Error domain: ${err.domain}');
  debugPrint('Error message: ${err.message}');
  
  // 根据错误类型采取不同处理
  if (err.code == 0) {
    // 内部错误
  } else if (err.code == 3) {
    // 没有广告填充
  }
  
  ad.dispose();
  _rewardedInterstitialAd = null;
},
```

## 实践练习

1. 实现完整的 `FullScreenContentCallback`
2. 处理所有广告事件
3. 实现资源清理逻辑
4. 添加错误处理

## 常见问题

### Q: 如果我不设置 FullScreenContentCallback 会怎样？

A: 广告仍然可以显示，但你将无法监听广告事件，也无法正确清理资源，可能导致内存泄漏。

### Q: 我可以在回调中修改 UI 吗？

A: 可以，但要注意线程安全。如果需要在回调中更新 UI，使用 `setState()` 或状态管理方案。

### Q: 广告关闭后，我应该立即加载新广告吗？

A: 建议这样做，以便在用户下次需要时广告已经准备好。但也要考虑网络和性能影响。

### Q: 如果广告显示失败，我应该重试吗？

A: 可以，但不要过于频繁。建议实现指数退避策略，或者等待用户下次触发时再加载。

## 总结与检查清单

### 本章要点

- `FullScreenContentCallback` 用于监听广告事件
- 必须处理 `onAdFailedToShowFullScreenContent` 和 `onAdDismissedFullScreenContent` 来清理资源
- 在 `dispose()` 方法中也要清理广告资源
- 正确处理错误，提供良好的用户体验

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `FullScreenContentCallback` 的所有回调方法
- [ ] 何时以及如何清理广告资源
- [ ] 如何处理广告显示失败
- [ ] 如何在 `dispose()` 中清理资源

下一章，我们将学习如何实现奖励机制，将广告奖励与游戏逻辑集成。
