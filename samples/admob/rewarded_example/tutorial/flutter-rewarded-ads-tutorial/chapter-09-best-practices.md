# 第 9 章：最佳实践与常见问题

## 章节简介

在本章中，我们将总结 Rewarded Ads 的最佳实践，帮助你避免常见错误，优化应用性能，并提供良好的用户体验。我们还将讨论常见问题的解决方案和调试技巧。

## 最佳实践

### 1. 显示时机选择

#### 用户主动选择

**推荐做法**：让用户主动选择观看广告以获得奖励。

**好的时机**：

- 游戏结束后提供额外奖励选项
- 用户需要额外资源时
- 用户想要跳过等待时间时
- 用户想要解锁内容时

**不好的时机**：

- 强制用户观看广告
- 在用户正在操作时弹出
- 频繁提示用户观看广告

#### 清晰的奖励说明

在显示广告按钮时，应该清楚地说明奖励内容：

```dart
TextButton(
  onPressed: () {
    _rewardedAd?.show(...);
  },
  child: const Text('Watch video for additional 10 coins'),
)
```

**代码说明**：

- 明确说明奖励类型（coins）
- 明确说明奖励数量（10）
- 让用户知道需要做什么（watch video）

### 2. 奖励机制设计

#### 合理的奖励数量

奖励数量应该：

- **有吸引力**：足够吸引用户观看广告
- **平衡游戏**：不会破坏游戏平衡
- **可持续**：不会导致经济系统崩溃

#### 奖励类型选择

根据应用类型选择合适的奖励：

- **游戏应用**：游戏币、生命、道具等
- **内容应用**：解锁内容、跳过等待等
- **工具应用**：高级功能、额外使用次数等

### 3. 预加载策略

#### 立即加载下一个广告

**推荐做法**：在广告关闭后立即加载下一个广告。

```dart
onAdDismissedFullScreenContent: (ad) {
  ad.dispose();
  _rewardedAd = null;
  _loadAd(); // 立即加载下一个广告
}
```

**好处**：

- 确保下次需要时广告已准备好
- 减少用户等待时间
- 提高广告展示率

#### 在应用启动时预加载

在应用初始化后立即加载第一个广告：

```dart
@override
void initState() {
  super.initState();
  // ... 其他初始化代码
  _initializeMobileAdsSDK(); // 这会加载第一个广告
}
```

### 4. 资源管理

#### 及时释放资源

在以下情况下应该释放广告资源：

1. **广告关闭后**：在 `onAdDismissedFullScreenContent` 中
2. **广告显示失败后**：在 `onAdFailedToShowFullScreenContent` 中
3. **Widget 销毁时**：在 `dispose()` 中

```dart
onAdDismissedFullScreenContent: (ad) {
  ad.dispose(); // 必须调用
  _rewardedAd = null; // 清空引用
  _loadAd(); // 加载下一个
}

@override
void dispose() {
  _rewardedAd?.dispose();
  super.dispose();
}
```

#### 检查 mounted 状态

在异步操作后更新状态时，检查 Widget 是否仍然挂载：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  if (mounted) {
    setState(() {
      _coins += rewardItem.amount.toInt();
    });
  }
}
```

### 5. 奖励存储

#### 使用持久化存储

使用 `SharedPreferences` 或其他持久化方案保存奖励：

```dart
Future<void> addCoins(int amount) async {
  final prefs = await SharedPreferences.getInstance();
  final currentCoins = prefs.getInt('coins') ?? 0;
  await prefs.setInt('coins', currentCoins + amount);
}
```

#### 及时保存

在获得奖励后立即保存：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) async {
  await _rewardManager.addCoins(rewardItem.amount.toInt());
  setState(() {
    _coins += rewardItem.amount.toInt();
  });
}
```

### 6. 错误处理

#### 记录详细错误信息

```dart
onAdFailedToLoad: (LoadAdError error) {
  debugPrint('Ad failed to load: $error');
  debugPrint('Error code: ${error.code}');
  debugPrint('Error domain: ${error.domain}');
  debugPrint('Error message: ${error.message}');
}
```

#### 根据错误类型处理

```dart
onAdFailedToLoad: (LoadAdError error) {
  switch (error.code) {
    case 0: // 内部错误
      // 可能需要重新初始化 SDK
      break;
    case 2: // 网络错误
      // 可以稍后重试
      _scheduleRetry();
      break;
    case 3: // 无广告填充
      // 正常情况，稍后重试
      _scheduleRetry();
      break;
  }
}
```

### 7. 测试广告使用

#### 始终使用测试广告

在开发和测试阶段，始终使用测试广告单元 ID：

```dart
final String _adUnitId = Platform.isAndroid
    ? 'ca-app-pub-3940256099942544/5224354917'  // 测试 ID
    : 'ca-app-pub-3940256099942544/1712485313'; // 测试 ID
```

**为什么重要**：

- 避免账号被暂停
- 确保测试环境的一致性
- 避免产生无效的广告展示

## 常见问题与解决方案

### 问题 1：奖励没有发放

**可能原因**：

1. 用户未完整观看广告
2. 回调未实现
3. 奖励数量为 0

**解决方案**：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  debugPrint('Reward type: ${rewardItem.type}');
  debugPrint('Reward amount: ${rewardItem.amount}');
  
  // 验证奖励
  if (rewardItem.amount > 0) {
    setState(() {
      _coins += rewardItem.amount.toInt();
    });
    
    // 保存奖励
    _saveRewards();
  } else {
    debugPrint('Invalid reward amount');
  }
}
```

### 问题 2：广告加载失败

**可能原因**：

1. 网络连接问题
2. 广告单元 ID 错误
3. AdMob App ID 未配置
4. 无广告填充

**解决方案**：

1. **检查网络**：确保设备有网络连接
2. **验证配置**：检查 `AndroidManifest.xml` 和 `Info.plist` 中的配置
3. **使用测试 ID**：确保使用正确的测试广告单元 ID
4. **实现重试**：在网络错误时实现重试机制

### 问题 3：奖励没有保存

**解决方案**：使用持久化存储保存奖励：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) async {
  await _rewardManager.addCoins(rewardItem.amount.toInt());
  await _loadCoins(); // 重新加载并更新 UI
}
```

### 问题 4：广告显示后应用崩溃

**可能原因**：

1. 在广告显示后访问已释放的资源
2. 在已销毁的 Widget 上更新状态

**解决方案**：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  if (mounted) { // 检查 Widget 是否仍然挂载
    setState(() {
      _coins += rewardItem.amount.toInt();
    });
  }
}
```

### 问题 5：可以重复获得奖励吗？

每个广告只能触发一次奖励。如果用户再次观看同一个广告，不会再次获得奖励。需要加载新的广告才能再次获得奖励。

## 调试技巧

### 1. 使用 Ad Inspector

Ad Inspector 是 Google Mobile Ads SDK 提供的调试工具：

```dart
MobileAds.instance.openAdInspector((error) {
  if (error != null) {
    debugPrint('Ad Inspector error: $error');
  }
});
```

### 2. 启用详细日志

在开发阶段，启用详细的日志记录：

```dart
onAdLoaded: (RewardedAd ad) {
  debugPrint('Ad loaded: ${ad.adUnitId}');
}

onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  debugPrint('Reward type: ${rewardItem.type}');
  debugPrint('Reward amount: ${rewardItem.amount}');
}

onAdFailedToLoad: (LoadAdError error) {
  debugPrint('Ad load failed:');
  debugPrint('  Code: ${error.code}');
  debugPrint('  Domain: ${error.domain}');
  debugPrint('  Message: ${error.message}');
}
```

### 3. 检查网络请求

使用网络监控工具检查广告请求：

- Android: 使用 Android Studio 的 Network Profiler
- iOS: 使用 Xcode 的 Network Instruments

### 4. 测试不同场景

测试以下场景：

- 不同网络条件
- 同意拒绝
- 广告加载失败
- 广告显示失败
- 奖励发放
- 应用生命周期变化

## 性能优化

### 1. 减少内存占用

及时释放不再需要的广告对象：

```dart
onAdDismissedFullScreenContent: (ad) {
  ad.dispose(); // 立即释放
  _rewardedAd = null;
}
```

### 2. 避免频繁重新加载

只在必要时重新加载广告：

```dart
void _loadAd() {
  // 如果已有广告，不重复加载
  if (_rewardedAd != null) {
    return;
  }
  
  // 加载新广告
  RewardedAd.load(...);
}
```

### 3. 优化奖励存储

批量保存奖励，减少 I/O 操作：

```dart
int _pendingCoins = 0;

onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  _pendingCoins += rewardItem.amount.toInt();
  
  // 延迟保存，批量处理
  _scheduleSave();
}
```

## 安全检查清单

在发布应用前，检查以下事项：

- [ ] 使用生产环境的广告单元 ID
- [ ] 更新 AdMob App ID
- [ ] 移除所有测试代码和调试设置
- [ ] 测试同意流程
- [ ] 测试广告加载和显示
- [ ] 测试奖励发放
- [ ] 测试奖励持久化
- [ ] 测试各种网络条件
- [ ] 检查错误处理
- [ ] 验证资源释放
- [ ] 测试应用生命周期

## 实践练习

完成以下练习以巩固本章内容：

1. **优化显示时机**：确保广告在合适的时机显示
2. **增强错误处理**：实现详细的错误处理和重试机制
3. **实现奖励持久化**：使用 SharedPreferences 保存奖励
4. **优化用户体验**：提供清晰的奖励说明和反馈
5. **使用 Ad Inspector**：学习使用 Ad Inspector 调试广告问题
6. **性能优化**：优化内存使用和加载时机

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 选择合适的显示时机
- [ ] 设计合理的奖励机制
- [ ] 实现预加载策略
- [ ] 正确管理资源
- [ ] 实现奖励持久化
- [ ] 实现详细的错误处理
- [ ] 优化用户体验
- [ ] 解决常见问题
- [ ] 使用调试工具
- [ ] 优化应用性能

## 教程总结

恭喜你完成了 Flutter Rewarded Ads 完整教程！通过本教程，你学习了：

1. **基础知识**：Rewarded Ads 的概念和使用场景
2. **项目配置**：如何配置 Flutter 项目以支持 Google Mobile Ads
3. **加载广告**：如何加载 Rewarded 广告
4. **显示广告**：如何显示广告和处理奖励
5. **事件处理**：如何监听和处理广告事件
6. **奖励机制**：如何实现奖励机制和持久化
7. **同意管理**：如何实现用户同意管理以符合隐私法规
8. **完整集成**：如何将所有组件整合在一起
9. **最佳实践**：如何避免常见错误并优化性能

现在你已经掌握了 Rewarded Ads 的完整实现流程。建议你：

1. **实践应用**：在实际项目中应用所学知识
2. **持续学习**：关注 Google Mobile Ads SDK 的更新
3. **优化改进**：根据实际使用情况不断优化实现
4. **分享经验**：与其他开发者分享你的经验和最佳实践

祝你开发顺利！
