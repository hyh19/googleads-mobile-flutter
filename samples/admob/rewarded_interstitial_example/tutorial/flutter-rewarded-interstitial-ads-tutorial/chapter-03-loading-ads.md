# 第 3 章 加载激励插屏广告

## 引言

加载广告是集成 Rewarded Interstitial Ads 的第一步。在本章中，我们将学习如何使用 `RewardedInterstitialAd.load()` 方法加载广告，以及如何处理加载成功和失败的情况。

## RewardedInterstitialAd.load() 方法

`RewardedInterstitialAd.load()` 是加载激励插屏广告的核心方法。它接受以下参数：

- `adUnitId`：广告单元 ID（字符串）
- `request`：广告请求对象（`AdRequest`）
- `rewardedInterstitialAdLoadCallback`：加载回调（`RewardedInterstitialAdLoadCallback`）

## 基本加载实现

让我们看看如何实现广告加载：

```dart
void _loadAd() async {
  // 检查是否可以请求广告（需要先处理用户同意）
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  RewardedInterstitialAd.load(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    rewardedInterstitialAdLoadCallback: RewardedInterstitialAdLoadCallback(
      onAdLoaded: (ad) {
        // 广告加载成功
        debugPrint('RewardedInterstitialAd loaded.');
        _rewardedInterstitialAd = ad;
      },
      onAdFailedToLoad: (LoadAdError error) {
        // 广告加载失败
        debugPrint('RewardedInterstitialAd failed to load: $error');
      },
    ),
  );
}
```

## 加载回调详解

### onAdLoaded 回调

当广告成功加载时，`onAdLoaded` 回调会被触发。在这个回调中，你应该：

1. **保存广告引用**：将加载的广告保存到变量中，以便后续显示
2. **设置事件回调**：设置 `FullScreenContentCallback` 来处理广告事件（我们将在第 6 章详细讲解）

```dart
onAdLoaded: (RewardedInterstitialAd ad) {
  debugPrint('RewardedInterstitialAd loaded.');
  
  // 保存广告引用
  _rewardedInterstitialAd = ad;
  
  // 设置全屏内容回调（将在第 6 章详细讲解）
  ad.fullScreenContentCallback = FullScreenContentCallback(
    onAdShowedFullScreenContent: (ad) {},
    onAdImpression: (ad) {},
    onAdFailedToShowFullScreenContent: (ad, err) {
      ad.dispose();
    },
    onAdDismissedFullScreenContent: (ad) {
      ad.dispose();
    },
    onAdClicked: (ad) {},
  );
},
```

### onAdFailedToLoad 回调

当广告加载失败时，`onAdFailedToLoad` 回调会被触发。在这个回调中，你应该：

1. **记录错误信息**：使用 `debugPrint` 或日志系统记录错误
2. **处理错误**：根据错误类型采取适当的处理措施（如重试、显示错误消息等）

```dart
onAdFailedToLoad: (LoadAdError error) {
  debugPrint('RewardedInterstitialAd failed to load: $error');
  debugPrint('Error code: ${error.code}');
  debugPrint('Error domain: ${error.domain}');
  debugPrint('Error message: ${error.message}');
  
  // 可以在这里实现重试逻辑
  // 例如：延迟后重新加载
},
```

## 错误处理和重试机制

### 常见错误代码

`LoadAdError` 对象包含以下信息：

- `code`：错误代码（整数）
- `domain`：错误域（字符串）
- `message`：错误消息（字符串）

常见的错误代码包括：

- `0`：内部错误
- `1`：无效请求
- `2`：网络错误
- `3`：没有广告填充

### 实现重试机制

如果广告加载失败，你可能想要实现重试机制：

```dart
int _retryCount = 0;
static const int _maxRetries = 3;

void _loadAd() async {
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  RewardedInterstitialAd.load(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    rewardedInterstitialAdLoadCallback: RewardedInterstitialAdLoadCallback(
      onAdLoaded: (ad) {
        _retryCount = 0; // 重置重试计数
        debugPrint('RewardedInterstitialAd loaded.');
        _rewardedInterstitialAd = ad;
        // 设置回调...
      },
      onAdFailedToLoad: (LoadAdError error) {
        debugPrint('RewardedInterstitialAd failed to load: $error');
        
        if (_retryCount < _maxRetries) {
          _retryCount++;
          // 延迟 2 秒后重试
          Future.delayed(const Duration(seconds: 2), () {
            _loadAd();
          });
        } else {
          debugPrint('Max retries reached. Giving up.');
          _retryCount = 0;
        }
      },
    ),
  );
}
```

## 广告请求配置

`AdRequest` 对象允许你配置广告请求的详细信息。目前我们使用默认配置：

```dart
request: const AdRequest()
```

在生产环境中，你可能需要配置：

- 内容定位
- 关键词
- 测试设备 ID
- 其他定位选项

## 完整示例

以下是完整的 `_loadAd()` 方法实现，包含错误处理和回调设置：

```dart
RewardedInterstitialAd? _rewardedInterstitialAd;

void _loadAd() async {
  // 检查是否可以请求广告
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  RewardedInterstitialAd.load(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    rewardedInterstitialAdLoadCallback: RewardedInterstitialAdLoadCallback(
      onAdLoaded: (ad) {
        debugPrint('RewardedInterstitialAd loaded.');
        
        // 设置全屏内容回调
        ad.fullScreenContentCallback = FullScreenContentCallback(
          onAdShowedFullScreenContent: (ad) {
            debugPrint('Ad showed full screen content.');
          },
          onAdImpression: (ad) {
            debugPrint('Ad recorded an impression.');
          },
          onAdFailedToShowFullScreenContent: (ad, err) {
            debugPrint('Ad failed to show full screen content: $err');
            ad.dispose();
            _rewardedInterstitialAd = null;
          },
          onAdDismissedFullScreenContent: (ad) {
            debugPrint('Ad dismissed full screen content.');
            ad.dispose();
            _rewardedInterstitialAd = null;
            // 加载下一个广告
            _loadAd();
          },
          onAdClicked: (ad) {
            debugPrint('Ad was clicked.');
          },
        );

        // 保存广告引用
        _rewardedInterstitialAd = ad;
      },
      onAdFailedToLoad: (LoadAdError error) {
        debugPrint('RewardedInterstitialAd failed to load: $error');
        _rewardedInterstitialAd = null;
      },
    ),
  );
}
```

## 实践练习

1. 实现基本的 `_loadAd()` 方法
2. 添加错误处理逻辑
3. 实现简单的重试机制（可选）
4. 测试广告加载功能

## 常见问题

### Q: 广告加载需要多长时间？

A: 广告加载时间取决于网络状况和广告可用性，通常在几秒内完成。如果长时间无法加载，可能是网络问题或没有可用的广告。

### Q: 我应该在什么时候加载广告？

A: 建议在应用启动时或用户可能触发广告展示之前预先加载广告。这样可以确保广告在需要时立即可用。

### Q: 如果广告加载失败，我应该怎么办？

A: 你可以实现重试机制，或者向用户显示友好的错误消息。确保不要过于频繁地重试，以免影响用户体验。

## 总结与检查清单

### 本章要点

- 使用 `RewardedInterstitialAd.load()` 加载广告
- 处理 `onAdLoaded` 和 `onAdFailedToLoad` 回调
- 实现错误处理和重试机制
- 保存广告引用以便后续显示

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `RewardedInterstitialAd.load()` 方法的使用
- [ ] 如何处理加载成功和失败的情况
- [ ] 如何设置 `FullScreenContentCallback`（基础部分）
- [ ] 如何实现基本的错误处理

下一章，我们将学习如何实现介绍屏幕（AdDialog），这是显示 Rewarded Interstitial Ad 之前的必要步骤。
