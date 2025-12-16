# 第 10 章 最佳实践与常见问题

## 引言

本章将总结 Rewarded Interstitial Ads 的最佳实践，帮助你优化实现、提高用户体验、增加广告收益，并避免常见问题。我们还将讨论何时使用 Rewarded Interstitial Ads vs Rewarded Ads。

## 何时使用 Rewarded Interstitial Ads vs Rewarded Ads

### 使用 Rewarded Interstitial Ads 的场景

- **自然断点**：在应用流程的自然中断点（如游戏结束、关卡完成）
- **额外奖励机会**：向用户提供额外的奖励机会，而不是必需的奖励
- **全屏体验**：希望提供沉浸式的全屏广告体验
- **介绍屏幕**：愿意实现介绍屏幕来符合 Google 政策要求

### 使用 Rewarded Ads 的场景

- **用户主动触发**：用户明确点击"观看广告获得奖励"按钮
- **必需奖励**：奖励是用户继续使用应用所必需的
- **简单集成**：不需要介绍屏幕，集成更简单
- **用户控制**：用户完全控制何时观看广告

### 选择建议

| 场景 | 推荐格式 | 原因 |
|------|---------|------|
| 游戏结束后提供额外生命 | Rewarded Interstitial Ads | 自然断点，额外奖励 |
| 用户点击"观看广告获得金币"按钮 | Rewarded Ads | 用户主动触发 |
| 关卡完成时提供奖励 | Rewarded Interstitial Ads | 自然断点 |
| 应用内购买前的替代选项 | Rewarded Ads | 用户主动选择 |
| 内容解锁（如文章、视频） | Rewarded Ads | 用户主动寻求内容 |

## 介绍屏幕设计最佳实践

### 1. 清晰的奖励说明

确保用户清楚地知道观看广告后能获得什么：

```dart
// ✅ 好：具体明确
title: const Text('Watch an ad for 10 more coins')

// ❌ 差：模糊不清
title: const Text('Watch an ad for rewards')
```

### 2. 合理的倒计时时间

倒计时时间应该平衡用户体验和信息传达：

- **太短（< 3 秒）**：用户可能来不及阅读和理解
- **太长（> 7 秒）**：用户可能失去兴趣
- **推荐（3-5 秒）**：足够阅读，不会太慢

### 3. 明显的跳过选项

跳过按钮应该清晰可见，使用醒目的颜色：

```dart
// ✅ 好：使用红色突出显示
child: const Text('No thanks', style: TextStyle(color: Colors.red))

// ❌ 差：不够明显
child: const Text('Skip')
```

### 4. 友好的文案

使用友好、非强制性的语言：

```dart
// ✅ 好：友好、有选择权
'Watch an ad for 10 more coins'

// ❌ 差：强制性强
'You must watch this ad to continue'
```

## 广告展示时机最佳实践

### 1. 避免频繁展示

不要过于频繁地展示广告，以免影响用户体验：

- **游戏应用**：建议在游戏结束或关卡完成时展示
- **内容应用**：建议在内容消费完成后展示
- **工具应用**：建议在功能使用达到限制时展示

### 2. 预加载广告

在用户可能需要观看广告之前预先加载：

```dart
void _startNewGame() {
  _countdownTimer.start();
  _loadAd(); // 预先加载广告
}
```

### 3. 检查广告可用性

在显示介绍屏幕之前，检查广告是否已加载：

```dart
if (_rewardedInterstitialAd == null) {
  // 广告未加载，可以显示加载提示或跳过介绍屏幕
  return;
}
```

## 奖励机制最佳实践

### 1. 立即发放奖励

奖励应该在 `onUserEarnedReward` 回调中立即发放：

```dart
// ✅ 正确：立即发放
onUserEarnedReward: (ad, rewardItem) {
  setState(() {
    _coins += rewardItem.amount.toInt();
  });
}

// ❌ 错误：延迟发放
onUserEarnedReward: (ad, rewardItem) {
  Future.delayed(Duration(seconds: 5), () {
    setState(() {
      _coins += rewardItem.amount.toInt();
    });
  });
}
```

### 2. 显示奖励通知

在用户获得奖励后，显示通知以提供反馈：

```dart
onUserEarnedReward: (ad, rewardItem) {
  setState(() {
    _coins += rewardItem.amount.toInt();
  });
  
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(
      content: Text('You earned ${rewardItem.amount.toInt()} coins!'),
      duration: const Duration(seconds: 2),
    ),
  );
}
```

### 3. 持久化奖励数据

使用本地存储或服务器端存储来保存奖励数据：

```dart
import 'package:shared_preferences/shared_preferences.dart';

Future<void> _saveCoins() async {
  final prefs = await SharedPreferences.getInstance();
  await prefs.setInt('coins', _coins);
}
```

## 资源管理最佳实践

### 1. 及时清理资源

在广告关闭或失败时，立即清理资源：

```dart
onAdDismissedFullScreenContent: (ad) {
  ad.dispose();
  _rewardedInterstitialAd = null;
  _loadAd(); // 加载下一个广告
}
```

### 2. 在 dispose() 中清理

在组件销毁时清理所有资源：

```dart
@override
void dispose() {
  _rewardedInterstitialAd?.dispose();
  _countdownTimer.dispose();
  super.dispose();
}
```

### 3. 避免内存泄漏

不要保留已使用广告的引用：

```dart
// ✅ 正确：清空引用
onAdDismissedFullScreenContent: (ad) {
  ad.dispose();
  _rewardedInterstitialAd = null;
}

// ❌ 错误：保留引用
onAdDismissedFullScreenContent: (ad) {
  ad.dispose();
  // 忘记清空引用
}
```

## 错误处理最佳实践

### 1. 记录错误信息

详细记录错误信息，便于调试：

```dart
onAdFailedToLoad: (LoadAdError error) {
  debugPrint('Ad failed to load: $error');
  debugPrint('Error code: ${error.code}');
  debugPrint('Error domain: ${error.domain}');
  debugPrint('Error message: ${error.message}');
}
```

### 2. 实现重试机制

对于加载失败，可以实现重试机制，但不要过于频繁：

```dart
int _retryCount = 0;
static const int _maxRetries = 3;

void _loadAd() async {
  // ... 加载广告
  
  onAdFailedToLoad: (error) {
    if (_retryCount < _maxRetries) {
      _retryCount++;
      Future.delayed(const Duration(seconds: 2), () {
        _loadAd();
      });
    }
  },
}
```

### 3. 优雅降级

如果广告无法加载，确保应用仍然可以正常使用：

```dart
void _showAdCallback() {
  if (_rewardedInterstitialAd == null) {
    // 广告未加载，可以显示友好提示
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(
        content: Text('Ad is not ready. Please try again later.'),
      ),
    );
    _loadAd(); // 尝试加载新广告
    return;
  }
  
  _rewardedInterstitialAd?.show(/* ... */);
}
```

## 测试最佳实践

### 1. 使用测试广告单元 ID

在开发和测试时，始终使用测试广告单元 ID：

```dart
final String _adUnitId = Platform.isAndroid
    ? 'ca-app-pub-3940256099942544/5354046379' // 测试 ID
    : 'ca-app-pub-3940256099942544/6978759866'; // 测试 ID
```

### 2. 测试不同场景

测试以下场景：

- 广告加载成功
- 广告加载失败
- 广告显示成功
- 广告显示失败
- 用户完成观看
- 用户关闭广告
- 用户点击广告
- 网络中断
- 应用切换到后台

### 3. 使用 Ad Inspector

使用 Ad Inspector 来调试广告问题：

```dart
MobileAds.instance.openAdInspector((error) {
  if (error != null) {
    debugPrint('Ad Inspector error: $error');
  }
});
```

## 性能优化

### 1. 预加载广告

在用户可能需要之前预先加载广告：

```dart
void _startNewGame() {
  _countdownTimer.start();
  _loadAd(); // 预先加载
}
```

### 2. 避免重复加载

检查广告是否已加载，避免重复加载：

```dart
void _loadAd() async {
  if (_rewardedInterstitialAd != null) {
    return; // 广告已加载
  }
  
  // 加载广告...
}
```

### 3. 合理使用资源

不要同时加载多个广告，只保留一个广告引用：

```dart
// ✅ 正确：只保留一个引用
RewardedInterstitialAd? _rewardedInterstitialAd;

// ❌ 错误：保留多个引用
List<RewardedInterstitialAd> _ads = [];
```

## 常见问题解答

### Q: 为什么我的广告无法加载？

A: 可能的原因包括：

- 网络连接问题
- 广告单元 ID 配置错误
- 用户未同意（GDPR 地区）
- 没有可用的广告填充

### Q: 介绍屏幕是必须的吗？

A: 是的，对于 Rewarded Interstitial Ads，介绍屏幕是 Google 政策的强制要求。

### Q: 我可以自定义介绍屏幕吗？

A: 可以，只要满足以下要求：

- 说明即将播放广告
- 提供奖励信息
- 提供跳过选项

### Q: 如果用户拒绝同意，我还能显示广告吗？

A: 可以，但只能显示非个性化广告，收益通常较低。

### Q: 我应该多久展示一次广告？

A: 建议不要过于频繁，在自然断点处展示，避免影响用户体验。

### Q: 如何提高广告收益？

A:

- 在合适的时机展示广告
- 确保用户完成观看
- 使用个性化广告（需要用户同意）
- 优化广告单元配置

## 总结与检查清单

### 本章要点

- 根据场景选择合适的广告格式
- 设计友好的介绍屏幕
- 在合适的时机展示广告
- 立即发放奖励
- 及时清理资源
- 实现错误处理和重试机制
- 使用测试广告进行测试

### 最终检查清单

在发布应用之前，确保：

- [ ] 使用真实的 AdMob 应用 ID 和广告单元 ID
- [ ] 实现了介绍屏幕（AdDialog）
- [ ] 实现了用户同意管理（UMP）
- [ ] 正确处理所有广告事件
- [ ] 及时清理广告资源
- [ ] 实现了错误处理
- [ ] 测试了所有场景
- [ ] 奖励机制正常工作
- [ ] 用户体验流畅自然

## 下一步

恭喜你完成了 Flutter Rewarded Interstitial Ads 的完整教程！现在你应该能够：

1. 理解 Rewarded Interstitial Ads 的特点和使用场景
2. 正确配置项目并集成 Google Mobile Ads SDK
3. 实现介绍屏幕以符合 Google 政策
4. 加载、显示和处理广告事件
5. 实现奖励机制和游戏逻辑集成
6. 集成用户同意管理以符合 GDPR 要求
7. 遵循最佳实践，优化用户体验

继续探索 Google Mobile Ads 的其他功能，如 Banner Ads、Interstitial Ads 等，以构建更完整的广告集成方案。祝你开发顺利！
