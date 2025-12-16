# 第 5 章：处理广告事件

## 章节简介

在本章中，我们将学习如何监听和处理 Rewarded 广告的各种事件。`FullScreenContentCallback` 提供了多个回调函数，用于处理广告展示、关闭、点击等事件。我们将详细讲解每个回调函数的用途和实现方法。

## FullScreenContentCallback 概述

### FullScreenContentCallback 类

`FullScreenContentCallback` 是一个回调类，用于处理全屏广告（包括 Rewarded Ads）的各种生命周期事件。它包含多个可选的回调方法，你只需要实现你关心的事件。

### 所有可用回调

`FullScreenContentCallback` 提供以下回调：

- `onAdShowedFullScreenContent`：广告成功展示到全屏
- `onAdFailedToShowFullScreenContent`：广告展示失败
- `onAdDismissedFullScreenContent`：广告被关闭
- `onAdImpression`：广告产生展示记录
- `onAdClicked`：广告被点击

## 设置回调

### 在加载成功时设置

推荐在广告加载成功时设置回调：

```dart
onAdLoaded: (RewardedAd ad) {
  debugPrint('Ad was loaded.');
  _rewardedAd = ad;
  
  // 设置全屏内容回调
  ad.fullScreenContentCallback = FullScreenContentCallback(
    // 回调实现将在下面详细讲解
  );
}
```

**代码说明**：

- 在 `onAdLoaded` 回调中设置 `fullScreenContentCallback`
- 这样可以在广告加载成功后立即准备好事件处理

## 必需的回调

### onAdDismissedFullScreenContent

当用户关闭广告（点击关闭按钮或返回）时，此回调会被调用：

```dart
onAdDismissedFullScreenContent: (ad) {
  debugPrint('Ad was dismissed.');
  ad.dispose();
  _rewardedAd = null;
  // 加载下一个广告
  _loadAd();
}
```

**代码说明**：

- `ad` 参数是被关闭的 `RewardedAd` 对象
- 调用 `dispose()` 释放广告资源
- 清空 `_rewardedAd` 引用
- **重要**：立即加载下一个广告，为下次显示做准备

**为什么立即加载下一个广告？**

Rewarded Ads 需要预加载才能在需要时立即显示。如果等到需要显示时再加载，用户会看到明显的延迟，影响用户体验。

### onAdFailedToShowFullScreenContent

当广告展示失败时，此回调会被调用：

```dart
onAdFailedToShowFullScreenContent: (ad, err) {
  debugPrint('Ad failed to show full screen content with error: $err');
  ad.dispose();
  _rewardedAd = null;
}
```

**代码说明**：

- `ad` 参数是失败的 `RewardedAd` 对象
- `err` 参数是 `AdError` 对象，包含错误信息
- 释放广告资源
- 清空引用

## 其他重要回调

### onAdShowedFullScreenContent

当广告成功展示到全屏时，此回调会被调用：

```dart
onAdShowedFullScreenContent: (ad) {
  debugPrint('Ad showed full screen content.');
  // 可以在这里暂停应用功能，如暂停游戏
}
```

**代码说明**：

- `ad` 参数是正在展示的 `RewardedAd` 对象
- 可以在这里暂停应用功能（如游戏、视频播放）

### onAdImpression

当广告产生展示记录时，此回调会被调用：

```dart
onAdImpression: (ad) {
  debugPrint('Ad recorded an impression.');
  // 记录展示事件用于分析
}
```

**代码说明**：

- `ad` 参数是产生展示的 `RewardedAd` 对象
- 用于记录展示事件和分析

**注意**：展示记录由 SDK 自动处理，你只需要监听这个事件即可。

### onAdClicked

当用户点击广告时，此回调会被调用：

```dart
onAdClicked: (ad) {
  debugPrint('Ad was clicked.');
  // 记录点击事件
}
```

**代码说明**：

- `ad` 参数是被点击的 `RewardedAd` 对象
- 用于记录点击事件和分析

## 完整的回调实现

以下是包含所有回调的完整实现：

```dart
onAdLoaded: (RewardedAd ad) {
  debugPrint('Ad was loaded.');
  _rewardedAd = ad;
  
  ad.fullScreenContentCallback = FullScreenContentCallback(
    onAdShowedFullScreenContent: (ad) {
      debugPrint('Ad showed full screen content.');
      // 暂停应用功能
    },
    onAdFailedToShowFullScreenContent: (ad, err) {
      debugPrint('Ad failed to show full screen content with error: $err');
      ad.dispose();
      _rewardedAd = null;
    },
    onAdDismissedFullScreenContent: (ad) {
      debugPrint('Ad was dismissed.');
      ad.dispose();
      _rewardedAd = null;
      // 立即加载下一个广告
      _loadAd();
    },
    onAdImpression: (ad) {
      debugPrint('Ad recorded an impression.');
      // 记录展示事件
    },
    onAdClicked: (ad) {
      debugPrint('Ad was clicked.');
      // 记录点击事件
    },
  );
}
```

## 实际应用场景

### 游戏应用

在游戏应用中，你可能需要在广告显示时暂停游戏：

```dart
bool _isGamePaused = false;

onAdShowedFullScreenContent: (ad) {
  debugPrint('Ad showed - pausing game');
  setState(() {
    _isGamePaused = true;
  });
  _pauseGame();
},

onAdDismissedFullScreenContent: (ad) {
  debugPrint('Ad dismissed - resuming game');
  setState(() {
    _isGamePaused = false;
  });
  _resumeGame();
  ad.dispose();
  _rewardedAd = null;
  _loadAd();
},
```

### 分析统计

记录广告事件用于分析：

```dart
onAdImpression: (ad) {
  // 记录展示事件
  _analytics.logEvent('ad_impression', parameters: {
    'ad_type': 'rewarded',
    'ad_unit_id': _adUnitId,
  });
},

onAdClicked: (ad) {
  // 记录点击事件
  _analytics.logEvent('ad_click', parameters: {
    'ad_type': 'rewarded',
    'ad_unit_id': _adUnitId,
  });
},
```

## 事件触发顺序

了解事件的触发顺序有助于正确实现逻辑：

1. **显示阶段**：
   - `onAdShowedFullScreenContent`（广告成功展示）
   - 或 `onAdFailedToShowFullScreenContent`（广告展示失败）

2. **用户交互**：
   - `onAdClicked`（用户点击广告，可选）
   - `onAdImpression`（产生展示记录，可能在展示时触发）

3. **奖励阶段**：
   - `onUserEarnedReward`（用户获得奖励，在 `show()` 方法中定义）

4. **关闭阶段**：
   - `onAdDismissedFullScreenContent`（用户关闭广告）

## 资源清理最佳实践

### 在回调中清理

确保在所有适当的回调中清理资源：

```dart
onAdDismissedFullScreenContent: (ad) {
  ad.dispose(); // 必须调用
  _rewardedAd = null; // 清空引用
  _loadAd(); // 加载下一个
},

onAdFailedToShowFullScreenContent: (ad, err) {
  ad.dispose(); // 必须调用
  _rewardedAd = null; // 清空引用
},
```

### 在 dispose() 中清理

在 Widget 销毁时也要清理：

```dart
@override
void dispose() {
  _rewardedAd?.dispose();
  super.dispose();
}
```

## 常见问题

### 哪些回调是必需的？

`onAdDismissedFullScreenContent` 是必需的，用于释放资源。其他回调都是可选的，但建议实现 `onAdFailedToShowFullScreenContent` 来处理错误。

### onAdImpression 什么时候触发？

`onAdImpression` 通常在广告实际显示给用户时触发，具体时机由 SDK 决定。

### 为什么需要在 onAdDismissedFullScreenContent 中立即加载下一个广告？

Rewarded Ads 需要预加载才能在需要时立即显示。如果等到需要显示时再加载，用户会看到明显的延迟。立即加载下一个广告可以确保在用户下次需要时，广告已经准备好了。

### 可以重复使用同一个广告对象吗？

不可以。每个 `RewardedAd` 对象只能显示一次。显示完成后必须调用 `dispose()` 释放资源，然后加载新的广告。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现所有回调**：在 `FullScreenContentCallback` 中实现所有回调函数
2. **添加日志**：为每个回调添加详细的日志记录
3. **实现暂停/恢复**：在广告显示时暂停应用功能，关闭时恢复
4. **记录分析事件**：使用回调记录广告事件用于分析
5. **测试事件触发**：运行应用并观察不同场景下的事件触发

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解 `FullScreenContentCallback` 的所有回调
- [ ] 实现 `onAdShowedFullScreenContent` 回调
- [ ] 实现 `onAdFailedToShowFullScreenContent` 回调
- [ ] 实现 `onAdDismissedFullScreenContent` 回调（必需）
- [ ] 实现 `onAdImpression` 和 `onAdClicked` 回调
- [ ] 在回调中正确清理资源
- [ ] 在广告关闭后立即加载下一个广告
- [ ] 理解事件的触发顺序
- [ ] 在实际场景中应用这些回调

## 下一步

现在我们已经学会了如何监听和处理广告事件。在下一章中，我们将学习如何实现奖励机制，包括奖励的存储、持久化和显示。

继续学习：[第 6 章：奖励机制实现](chapter-06-reward-handling.md)
