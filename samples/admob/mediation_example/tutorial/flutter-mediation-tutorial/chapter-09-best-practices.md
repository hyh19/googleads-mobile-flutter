# 第 9 章 最佳实践与常见问题

## 引言

本章将总结广告中介的最佳实践，帮助你优化实现、提高广告填充率和收益，并避免常见问题。我们还将提供调试技巧和常见问题的解答。

## 初始化最佳实践

### 1. 在应用启动时初始化

在 `main()` 函数中初始化 SDK，确保所有适配器都准备好：

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  
  // 尽早初始化
  MobileAds.instance.initialize().then((initializationStatus) {
    initializationStatus.adapterStatuses.forEach((key, value) {
      debugPrint('Adapter status for $key: ${value.description}');
    });
  });
  
  runApp(MyApp());
}
```

### 2. 等待初始化完成

在加载广告之前，等待初始化完成：

```dart
Future<void> _initializeAndLoadAds() async {
  final initializationStatus = await MobileAds.instance.initialize();
  
  // 检查所有适配器是否准备好
  final readyAdapters = initializationStatus.adapterStatuses.entries
      .where((entry) => 
          entry.value.initializationState == AdapterInitializationState.READY)
      .length;
  
  debugPrint('$readyAdapters adapters are ready.');
  
  // 开始加载广告
  _loadAd();
}
```

### 3. 处理初始化失败

即使某些适配器初始化失败，也应该继续使用其他网络：

```dart
final initializationStatus = await MobileAds.instance.initialize();

initializationStatus.adapterStatuses.forEach((key, value) {
  if (value.initializationState == AdapterInitializationState.NOT_READY) {
    debugPrint('Warning: $key failed to initialize: ${value.description}');
  }
});

// 继续使用已初始化的网络
_loadAd();
```

## 适配器状态监控

### 定期检查状态

在应用运行时，可以定期检查适配器状态：

```dart
void _checkAdapterStatus() {
  MobileAds.instance.initialize().then((initializationStatus) {
    final notReadyAdapters = initializationStatus.adapterStatuses.entries
        .where((entry) => 
            entry.value.initializationState != AdapterInitializationState.READY)
        .map((entry) => entry.key)
        .toList();
    
    if (notReadyAdapters.isNotEmpty) {
      debugPrint('Warning: Some adapters are not ready:');
      notReadyAdapters.forEach((adapter) {
        debugPrint('  - $adapter');
      });
    }
  });
}
```

## 错误处理和重试

### 1. 处理加载失败

实现适当的错误处理：

```dart
BannerAdListener(
  onAdFailedToLoad: (ad, error) {
    debugPrint('Ad failed to load: ${error.message}');
    debugPrint('Error code: ${error.code}');
    debugPrint('Error domain: ${error.domain}');
    
    // 根据错误类型采取不同处理
    if (error.code == 0) {
      // 内部错误，可以重试
      _retryLoadAd();
    } else if (error.code == 3) {
      // 没有广告填充，这是正常的
      debugPrint('No ad available.');
    }
    
    ad.dispose();
  },
)
```

### 2. 实现重试机制

对于可恢复的错误，实现重试机制：

```dart
int _retryCount = 0;
static const int _maxRetries = 3;

void _loadAdWithRetry() {
  _bannerAd = BannerAd(
    // ... 配置
    listener: BannerAdListener(
      onAdLoaded: (ad) {
        _retryCount = 0; // 重置重试计数
        // ... 处理加载成功
      },
      onAdFailedToLoad: (ad, error) {
        if (_retryCount < _maxRetries && error.code == 0) {
          _retryCount++;
          Future.delayed(const Duration(seconds: 2), () {
            _loadAdWithRetry();
          });
        } else {
          debugPrint('Max retries reached or non-retryable error.');
          _retryCount = 0;
        }
        ad.dispose();
      },
    ),
  )..load();
}
```

## 性能优化建议

### 1. 预加载广告

在用户可能需要之前预先加载广告：

```dart
void _preloadAds() {
  // 预加载多个广告
  _loadBannerAd();
  _loadInterstitialAd();
  _loadRewardedAd();
}
```

### 2. 避免重复加载

检查广告是否已加载，避免重复加载：

```dart
void _loadBannerAd() {
  if (_bannerAd != null && _bannerIsLoaded) {
    return; // 广告已加载
  }
  
  // 加载新广告
  _bannerAd = BannerAd(/* ... */)..load();
}
```

### 3. 合理使用资源

及时清理不再使用的广告：

```dart
@override
void dispose() {
  _bannerAd?.dispose();
  _interstitialAd?.dispose();
  _rewardedAd?.dispose();
  super.dispose();
}
```

## 中介配置优化

### 1. 定期更新 eCPM

根据实际收益数据定期更新 eCPM：

- 查看 AdMob 控制台中的网络性能数据
- 根据填充率和 eCPM 调整网络优先级
- 使用自动优化功能

### 2. 监控网络性能

定期检查各个网络的性能：

- **填充率**：哪些网络提供了最多的广告
- **eCPM**：哪些网络的收益最高
- **错误率**：哪些网络经常失败

### 3. 调整网络优先级

根据性能数据调整网络优先级：

- 将表现好的网络提升到更高优先级
- 将表现差的网络降低优先级或移除
- 测试不同的优先级配置

## 调试技巧

### 1. 使用 Ad Inspector

使用 Ad Inspector 来调试广告问题：

```dart
MobileAds.instance.openAdInspector((error) {
  if (error != null) {
    debugPrint('Ad Inspector error: $error');
  } else {
    debugPrint('Ad Inspector closed.');
  }
});
```

### 2. 启用详细日志

在开发阶段启用详细日志：

```dart
// 在初始化时启用日志
MobileAds.instance.initialize().then((status) {
  // 打印所有适配器状态
  status.adapterStatuses.forEach((key, value) {
    debugPrint('=== Adapter Status ===');
    debugPrint('Name: $key');
    debugPrint('State: ${value.initializationState}');
    debugPrint('Description: ${value.description}');
    debugPrint('Latency: ${value.latency}ms');
  });
});
```

### 3. 记录广告来源

记录每个广告的来源，用于分析：

```dart
onAdLoaded: (ad) {
  final networkName = getNetworkName(ad.responseInfo?.mediationAdapterClassName);
  debugPrint('Ad loaded from: $networkName');
  // 记录到分析服务
  _analytics.logEvent('ad_loaded', parameters: {
    'network': networkName ?? 'unknown',
  });
}
```

## 常见问题解答

### Q: 为什么某些适配器显示为 MISSING？

A: 可能的原因：

- 适配器库未添加到项目中
- 依赖未正确同步
- 版本不兼容

**解决方案**：

1. 检查 `build.gradle`（Android）或 `Podfile`（iOS）中是否添加了依赖
2. 同步项目（`./gradlew build` 或 `pod install`）
3. 检查适配器版本是否与 SDK 版本兼容

### Q: 适配器初始化失败怎么办？

A: 可能的原因：

- 网络特定配置错误（如 AppLovin SDK Key）
- 网络连接问题
- SDK 版本不兼容

**解决方案**：

1. 检查网络特定配置是否正确
2. 查看日志了解具体错误信息
3. 确保网络连接正常
4. 更新适配器到最新版本

### Q: 如何知道应该添加哪些适配器？

A: 只添加你在 AdMob 控制台中配置的网络对应的适配器。添加不需要的适配器会增加应用大小。

### Q: 广告填充率低怎么办？

A: 可能的原因和解决方案：

1. **适配器未初始化**：确保所有适配器都已正确初始化
2. **网络配置错误**：检查 AdMob 控制台中的网络配置
3. **eCPM 设置过低**：调整网络优先级和 eCPM
4. **测试环境**：确保使用测试广告单元进行测试

### Q: 如何优化广告收益？

A: 优化建议：

1. **添加更多网络**：增加可用的广告来源
2. **优化 eCPM**：根据实际数据调整 eCPM
3. **调整优先级**：将高收益网络提升到更高优先级
4. **使用自动优化**：启用 AdMob 的自动优化功能
5. **监控性能**：定期检查网络性能并调整配置

### Q: Platform Channel 调用失败怎么办？

A: 检查：

1. 通道名称是否匹配（Dart 和原生端）
2. 方法名是否正确
3. 参数类型是否匹配
4. 原生代码是否正确实现

### Q: Network Extras 不生效怎么办？

A: 检查：

1. 是否正确实现了 `MediationNetworkExtrasProvider`（Android）或 `FLTMediationNetworkExtrasProvider`（iOS）
2. 是否正确注册了提供者
3. 适配器是否支持该 Extras
4. 参数是否正确传递

## 测试建议

### 1. 使用测试广告单元

在开发和测试阶段，始终使用测试广告单元：

```dart
final String _adUnitId = Platform.isAndroid
    ? 'ca-app-pub-3940256099942544/6300978111' // 测试 ID
    : 'ca-app-pub-3940256099942544/2934735716'; // 测试 ID
```

### 2. 测试不同场景

测试以下场景：

- 所有适配器都初始化成功
- 某些适配器初始化失败
- 网络连接中断
- 应用切换到后台
- 不同的广告格式

### 3. 验证中介配置

在 AdMob 控制台中验证：

- 中介组配置正确
- 网络账户信息正确
- 广告单元映射正确
- eCPM 和优先级设置合理

## 总结与检查清单

### 本章要点

- 在应用启动时初始化 SDK
- 等待初始化完成再加载广告
- 实现适当的错误处理和重试机制
- 定期监控和优化中介配置
- 使用调试工具排查问题

### 最终检查清单

在发布应用之前，确保：

- [ ] SDK 在应用启动时正确初始化
- [ ] 所有适配器都已正确添加和配置
- [ ] 适配器状态已检查并处理
- [ ] AdMob 控制台中的中介配置正确
- [ ] Platform Channel 正确实现（如需要）
- [ ] Network Extras 正确配置（如需要）
- [ ] 错误处理已实现
- [ ] 资源清理已实现
- [ ] 测试了所有场景
- [ ] 使用了真实的 AdMob 应用 ID 和广告单元 ID

## 下一步

恭喜你完成了 Flutter 广告中介的完整教程！现在你应该能够：

1. 理解广告中介的概念和工作原理
2. 正确配置项目以支持广告中介
3. 初始化 SDK 并检查适配器状态
4. 在 AdMob 控制台中配置中介
5. 添加和管理适配器库
6. 使用 Platform Channel 调用第三方 SDK API
7. 配置网络特定参数
8. 检测和分析广告来源
9. 遵循最佳实践，优化广告收益

继续探索 Google Mobile Ads 的其他功能，如不同的广告格式、高级定位选项等，以构建更完整的广告集成方案。祝你开发顺利！
