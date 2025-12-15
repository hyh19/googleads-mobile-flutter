# 第 7 章：广告事件监听

## 章节简介

在本章中，我们将学习如何监听和处理 Banner 广告的各种事件。`BannerAdListener` 提供了多个回调函数，用于处理广告加载、点击、展示等事件。我们将详细讲解每个回调函数的用途和实现方法。

## BannerAdListener 概述

### BannerAdListener 类

`BannerAdListener` 是一个回调类，用于处理 Banner 广告的各种生命周期事件。它包含多个可选的回调方法，你只需要实现你关心的事件。

### 所有可用回调

`BannerAdListener` 提供以下回调：

- `onAdLoaded`：广告加载成功
- `onAdFailedToLoad`：广告加载失败
- `onAdOpened`：广告打开（用户点击）
- `onAdClosed`：广告关闭
- `onAdImpression`：广告展示（产生展示记录）
- `onAdClicked`：广告被点击
- `onAdWillDismissScreen`：即将关闭全屏视图（iOS）

## 必需的回调

### onAdLoaded

当广告成功加载时调用：

```dart
onAdLoaded: (ad) {
  debugPrint('Ad was loaded.');
  setState(() {
    _bannerAd = ad as BannerAd;
  });
}
```

**参数**：

- `ad`：成功加载的 `Ad` 对象

**用途**：

- 保存广告对象
- 更新 UI 状态
- 记录分析事件

### onAdFailedToLoad

当广告加载失败时调用：

```dart
onAdFailedToLoad: (ad, err) {
  debugPrint('Ad failed to load with error: $err');
  debugPrint('Error code: ${err.code}');
  debugPrint('Error message: ${err.message}');
  ad.dispose();
}
```

**参数**：

- `ad`：失败的 `Ad` 对象
- `err`：`LoadAdError` 对象，包含错误信息

**用途**：

- 记录错误信息
- 释放广告对象
- 实现重试逻辑

## 用户交互回调

### onAdOpened

当用户点击广告并打开全屏内容时调用：

```dart
onAdOpened: (Ad ad) {
  debugPrint('Ad was opened.');
  // 可以在这里暂停应用功能，如暂停游戏
}
```

**参数**：

- `ad`：被打开的 `Ad` 对象

**用途**：

- 暂停应用功能（如游戏、视频播放）
- 记录分析事件
- 更新 UI 状态

### onAdClosed

当用户关闭全屏内容返回应用时调用：

```dart
onAdClosed: (Ad ad) {
  debugPrint('Ad was closed.');
  // 可以在这里恢复应用功能
}
```

**参数**：

- `ad`：被关闭的 `Ad` 对象

**用途**：

- 恢复应用功能
- 记录分析事件
- 更新 UI 状态

## 展示和点击回调

### onAdImpression

当广告产生展示记录时调用：

```dart
onAdImpression: (Ad ad) {
  debugPrint('Ad recorded an impression.');
  // 记录展示事件用于分析
}
```

**参数**：

- `ad`：产生展示的 `Ad` 对象

**用途**：

- 记录展示事件
- 用于分析和统计
- 计算广告效果

**注意**：展示记录由 SDK 自动处理，你只需要监听这个事件即可。

### onAdClicked

当用户点击广告时调用：

```dart
onAdClicked: (Ad ad) {
  debugPrint('Ad was clicked.');
  // 记录点击事件
}
```

**参数**：

- `ad`：被点击的 `Ad` 对象

**用途**：

- 记录点击事件
- 用于分析用户参与度
- 计算点击率

**注意**：点击事件会在 `onAdOpened` 之前触发。

## iOS 特定回调

### onAdWillDismissScreen

仅在 iOS 上调用，在关闭全屏视图之前触发：

```dart
onAdWillDismissScreen: (Ad ad) {
  debugPrint('Ad will be dismissed.');
  // iOS 特定处理
}
```

**参数**：

- `ad`：即将关闭的 `Ad` 对象

**用途**：

- iOS 特定的清理工作
- 准备恢复应用状态

**注意**：这个回调只在 iOS 上触发，Android 上不会调用。

## 完整的监听器实现

以下是包含所有回调的完整实现：

```dart
listener: BannerAdListener(
  // 必需回调
  onAdLoaded: (ad) {
    debugPrint('Ad was loaded.');
    setState(() {
      _bannerAd = ad as BannerAd;
    });
  },
  
  onAdFailedToLoad: (ad, err) {
    debugPrint('Ad failed to load with error: $err');
    debugPrint('Error code: ${err.code}');
    debugPrint('Error message: ${err.message}');
    ad.dispose();
  },
  
  // 用户交互回调
  onAdOpened: (Ad ad) {
    debugPrint('Ad was opened.');
    // 暂停应用功能
  },
  
  onAdClosed: (Ad ad) {
    debugPrint('Ad was closed.');
    // 恢复应用功能
  },
  
  // 展示和点击回调
  onAdImpression: (Ad ad) {
    debugPrint('Ad recorded an impression.');
    // 记录展示事件
  },
  
  onAdClicked: (Ad ad) {
    debugPrint('Ad was clicked.');
    // 记录点击事件
  },
  
  // iOS 特定回调
  onAdWillDismissScreen: (Ad ad) {
    debugPrint('Ad will be dismissed.');
    // iOS 特定处理
  },
),
```

## 实际应用场景

### 游戏应用

在游戏应用中，你可能需要在广告打开时暂停游戏：

```dart
bool _isGamePaused = false;

onAdOpened: (Ad ad) {
  debugPrint('Ad was opened - pausing game');
  setState(() {
    _isGamePaused = true;
  });
  // 暂停游戏逻辑
  _pauseGame();
},

onAdClosed: (Ad ad) {
  debugPrint('Ad was closed - resuming game');
  setState(() {
    _isGamePaused = false;
  });
  // 恢复游戏逻辑
  _resumeGame();
},
```

### 视频播放应用

在视频播放应用中，暂停和恢复视频：

```dart
onAdOpened: (Ad ad) {
  debugPrint('Ad was opened - pausing video');
  _videoPlayerController.pause();
},

onAdClosed: (Ad ad) {
  debugPrint('Ad was closed - resuming video');
  _videoPlayerController.play();
},
```

### 分析统计

记录广告事件用于分析：

```dart
onAdImpression: (Ad ad) {
  // 记录展示事件
  _analytics.logEvent('ad_impression', parameters: {
    'ad_type': 'banner',
    'ad_unit_id': _adUnitId,
  });
},

onAdClicked: (Ad ad) {
  // 记录点击事件
  _analytics.logEvent('ad_click', parameters: {
    'ad_type': 'banner',
    'ad_unit_id': _adUnitId,
  });
},
```

## 事件触发顺序

了解事件的触发顺序有助于正确实现逻辑：

1. **加载阶段**：
   - `onAdLoaded` 或 `onAdFailedToLoad`

2. **用户交互**：
   - `onAdClicked`（用户点击）
   - `onAdOpened`（打开全屏内容）
   - `onAdWillDismissScreen`（iOS，即将关闭）
   - `onAdClosed`（关闭全屏内容）

3. **展示记录**：
   - `onAdImpression`（可能在加载后或显示时触发）

## 常见问题

### 哪些回调是必需的？

只有 `onAdLoaded` 和 `onAdFailedToLoad` 是必需的，其他都是可选的。

### onAdImpression 什么时候触发？

`onAdImpression` 通常在广告实际显示给用户时触发，具体时机由 SDK 决定。

### onAdClicked 和 onAdOpened 的区别？

- `onAdClicked`：用户点击广告时立即触发
- `onAdOpened`：全屏内容打开时触发（在点击之后）

### 为什么需要 onAdWillDismissScreen？

这是 iOS 特定的回调，用于在关闭全屏视图之前进行清理工作。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现所有回调**：在 `BannerAdListener` 中实现所有回调函数
2. **添加日志**：为每个回调添加详细的日志记录
3. **实现暂停/恢复**：在广告打开时暂停应用功能，关闭时恢复
4. **记录分析事件**：使用回调记录广告事件用于分析

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解 `BannerAdListener` 的所有回调
- [ ] 实现 `onAdLoaded` 和 `onAdFailedToLoad` 回调
- [ ] 实现 `onAdOpened` 和 `onAdClosed` 回调
- [ ] 实现 `onAdImpression` 和 `onAdClicked` 回调
- [ ] 理解 `onAdWillDismissScreen`（iOS 特定）
- [ ] 在实际场景中应用这些回调
- [ ] 理解事件的触发顺序

## 下一步

现在我们已经学会了如何监听和处理广告事件。在下一章中，我们将学习如何集成用户同意管理（UMP），以符合 GDPR 等隐私法规的要求。

继续学习：[第 8 章：用户同意管理（UMP）](chapter-08-consent-management.md)
