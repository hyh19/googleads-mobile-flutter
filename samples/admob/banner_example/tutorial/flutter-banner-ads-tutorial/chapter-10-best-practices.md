# 第 10 章：最佳实践与常见问题

## 章节简介

在本章中，我们将总结 Banner Ads 的最佳实践，帮助你避免常见错误，优化应用性能，并提供良好的用户体验。我们还将讨论常见问题的解决方案和调试技巧。

## 最佳实践

### 1. 广告位置选择

#### 底部显示（推荐）

底部是最常见的 Banner 广告位置，推荐使用：

```dart
Align(
  alignment: Alignment.bottomCenter,
  child: SafeArea(
    child: SizedBox(
      width: _bannerAd!.size.width.toDouble(),
      height: _bannerAd!.size.height.toDouble(),
      child: AdWidget(ad: _bannerAd!),
    ),
  ),
)
```

**优势**：

- 不遮挡主要内容
- 用户习惯底部广告位置
- 不影响用户操作

#### 避免的位置

- **内容中间**：不要将广告放在内容中间，会打断用户体验
- **关键操作区域**：不要遮挡重要的按钮或操作区域

### 2. 使用自适应尺寸

#### 推荐使用自适应尺寸

始终使用自适应横幅广告尺寸：

```dart
final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
  MediaQuery.sizeOf(context).width.truncate(),
);
```

**优势**：

- 自动适配不同设备
- 最佳显示效果
- 更高的填充率

#### 避免固定尺寸

除非有特殊需求，否则避免使用固定尺寸：

```dart
// 不推荐
size: AdSize.banner, // 固定 320x50
```

### 3. 测试广告使用

#### 始终使用测试广告

在开发和测试阶段，始终使用测试广告单元 ID：

```dart
final String _adUnitId = Platform.isAndroid
    ? 'ca-app-pub-3940256099942544/9214589741'  // 测试 ID
    : 'ca-app-pub-3940256099942544/2435281174'; // 测试 ID
```

**为什么重要**：

- 避免账号被暂停
- 确保测试环境的一致性
- 避免产生无效的广告展示

### 4. 资源管理

#### 及时释放资源

在以下情况下应该释放广告资源：

1. **方向改变时**：释放旧广告，加载新广告
2. **Widget 销毁时**：在 `dispose()` 中清理

```dart
@override
void dispose() {
  _bannerAd?.dispose();
  super.dispose();
}
```

#### 检查 mounted 状态

在异步操作后更新状态时，检查 Widget 是否仍然挂载：

```dart
void _loadAd() async {
  // ... 异步操作
  
  if (!mounted) {
    return;
  }
  
  setState(() {
    // 更新状态
  });
}
```

### 5. 方向变化处理

#### 正确处理方向变化

使用 `OrientationBuilder` 检测方向变化：

```dart
OrientationBuilder(
  builder: (context, orientation) {
    if (_currentOrientation != orientation) {
      if (_currentOrientation != null) {
        _bannerAd?.dispose();
        _bannerAd = null;
        _loadAd();
      }
      _currentOrientation = orientation;
    }
    return Widget();
  },
)
```

**关键点**：

- 只在方向真正改变时重新加载
- 释放旧广告资源
- 使用新的屏幕宽度获取尺寸

### 6. 错误处理

#### 记录详细错误信息

```dart
onAdFailedToLoad: (ad, err) {
  debugPrint('Ad failed to load: $err');
  debugPrint('Error code: ${err.code}');
  debugPrint('Error message: ${err.message}');
  ad.dispose();
}
```

#### 根据错误类型处理

```dart
onAdFailedToLoad: (ad, err) {
  switch (err.code) {
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
  ad.dispose();
}
```

### 7. 用户体验优化

#### 使用 SafeArea

始终使用 `SafeArea` 确保广告不被系统 UI 遮挡：

```dart
SafeArea(
  child: AdWidget(ad: _bannerAd!),
)
```

#### 避免遮挡内容

确保广告不会遮挡重要的应用内容：

- 使用 `Stack` 和 `Align` 精确定位
- 为主内容留出足够的空间
- 考虑不同设备的屏幕尺寸

## 常见问题与解决方案

### 问题 1：广告不显示

**可能原因**：

1. SDK 未初始化
2. 同意未获取
3. 广告未加载
4. 网络问题
5. 尺寸获取失败

**解决方案**：

```dart
// 1. 确保 SDK 已初始化
await MobileAds.instance.initialize();

// 2. 检查同意状态
if (await _consentManager.canRequestAds()) {
  // 3. 检查广告是否加载
  if (_bannerAd != null) {
    // 4. 检查尺寸
    final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
      MediaQuery.sizeOf(context).width.truncate(),
    );
    if (size != null) {
      // 显示广告
    }
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

### 问题 3：方向改变时广告不更新

**可能原因**：

1. 没有使用 `OrientationBuilder`
2. 没有检测方向变化
3. 没有重新加载广告

**解决方案**：

```dart
OrientationBuilder(
  builder: (context, orientation) {
    if (_currentOrientation != orientation) {
      if (_currentOrientation != null) {
        _bannerAd?.dispose();
        _bannerAd = null;
        _loadAd(); // 重新加载
      }
      _currentOrientation = orientation;
    }
    return Widget();
  },
)
```

### 问题 4：广告尺寸不正确

**可能原因**：

1. 使用固定尺寸而不是自适应尺寸
2. 尺寸设置错误

**解决方案**：

```dart
// 使用自适应尺寸
final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
  MediaQuery.sizeOf(context).width.truncate(),
);

// 使用实际尺寸设置 SizedBox
SizedBox(
  width: _bannerAd!.size.width.toDouble(),
  height: _bannerAd!.size.height.toDouble(),
  child: AdWidget(ad: _bannerAd!),
)
```

### 问题 5：广告被系统 UI 遮挡

**解决方案**：使用 `SafeArea`：

```dart
SafeArea(
  child: AdWidget(ad: _bannerAd!),
)
```

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
onAdLoaded: (ad) {
  debugPrint('Ad loaded: ${ad.adUnitId}');
  debugPrint('Ad size: ${(ad as BannerAd).size.width}x${(ad as BannerAd).size.height}');
}

onAdFailedToLoad: (ad, err) {
  debugPrint('Ad load failed:');
  debugPrint('  Code: ${err.code}');
  debugPrint('  Domain: ${err.domain}');
  debugPrint('  Message: ${err.message}');
}
```

### 3. 检查网络请求

使用网络监控工具检查广告请求：

- Android: 使用 Android Studio 的 Network Profiler
- iOS: 使用 Xcode 的 Network Instruments

### 4. 测试不同场景

测试以下场景：

- 不同设备尺寸
- 竖屏和横屏
- 网络断开
- 同意拒绝
- 方向变化

## 性能优化

### 1. 减少内存占用

及时释放不再需要的广告对象：

```dart
@override
void dispose() {
  _bannerAd?.dispose();
  super.dispose();
}
```

### 2. 避免频繁重新加载

只在必要时重新加载广告：

```dart
if (_currentOrientation != orientation) {
  if (_currentOrientation != null) {
    // 只在方向真正改变时重新加载
    _loadAd();
  }
}
```

### 3. 优化布局

使用高效的布局方式：

```dart
Stack(
  children: [
    // 主要内容
    // 广告（使用 Align 定位）
  ],
)
```

## 安全检查清单

在发布应用前，检查以下事项：

- [ ] 使用生产环境的广告单元 ID
- [ ] 更新 AdMob App ID
- [ ] 移除所有测试代码和调试设置
- [ ] 测试同意流程
- [ ] 测试广告加载和显示
- [ ] 测试各种网络条件
- [ ] 测试竖屏和横屏
- [ ] 测试方向变化
- [ ] 检查错误处理
- [ ] 验证资源释放

## 实践练习

完成以下练习以巩固本章内容：

1. **优化广告位置**：选择最佳的广告位置
2. **增强错误处理**：实现详细的错误处理和重试机制
3. **优化用户体验**：确保广告不遮挡重要内容
4. **使用 Ad Inspector**：学习使用 Ad Inspector 调试广告问题
5. **性能优化**：优化内存使用和加载时机

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 选择最佳的广告位置
- [ ] 使用自适应广告尺寸
- [ ] 始终使用测试广告进行开发
- [ ] 正确管理资源
- [ ] 正确处理方向变化
- [ ] 实现详细的错误处理
- [ ] 优化用户体验
- [ ] 解决常见问题
- [ ] 使用调试工具
- [ ] 优化应用性能

## 教程总结

恭喜你完成了 Flutter Banner Ads 完整教程！通过本教程，你学习了：

1. **基础知识**：Banner Ads 的概念和使用场景
2. **项目配置**：如何配置 Flutter 项目以支持 Google Mobile Ads
3. **自适应尺寸**：如何获取和使用自适应横幅广告尺寸
4. **加载广告**：如何加载 Banner 广告
5. **显示广告**：如何显示 Banner 广告
6. **方向处理**：如何处理屏幕方向变化
7. **事件监听**：如何监听和处理广告事件
8. **同意管理**：如何实现用户同意管理以符合隐私法规
9. **完整集成**：如何将所有组件整合在一起
10. **最佳实践**：如何避免常见错误并优化性能

现在你已经掌握了 Banner Ads 的完整实现流程。建议你：

1. **实践应用**：在实际项目中应用所学知识
2. **持续学习**：关注 Google Mobile Ads SDK 的更新
3. **优化改进**：根据实际使用情况不断优化实现
4. **分享经验**：与其他开发者分享你的经验和最佳实践

祝你开发顺利！
