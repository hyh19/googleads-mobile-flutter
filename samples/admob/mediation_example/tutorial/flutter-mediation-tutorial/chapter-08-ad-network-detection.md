# 第 8 章 检测广告来源网络

## 引言

在使用广告中介时，了解广告来自哪个网络非常重要。这可以帮助你分析各个网络的性能、优化中介配置，以及进行数据分析。本章将详细讲解如何使用 `ResponseInfo` 来检测广告来源网络。

## ResponseInfo 概述

### 什么是 ResponseInfo？

`ResponseInfo` 对象包含广告响应的详细信息，包括：

- `mediationAdapterClassName`：提供广告的中介适配器类名
- `responseId`：广告响应的唯一标识符
- `adapterResponses`：所有适配器的响应信息

### 为什么需要检测广告来源？

检测广告来源网络可以帮助你：

1. **性能分析**：了解哪个网络提供了最多的广告
2. **收益优化**：分析各个网络的 eCPM 和收益
3. **问题排查**：识别哪些网络可能存在问题
4. **数据统计**：记录和分析广告来源数据

## 在 Banner Ad 中检测

### 基本检测

在 Banner Ad 的 `onAdLoaded` 回调中，可以获取 `ResponseInfo`：

```dart
BannerAd(
  adUnitId: _adUnitId,
  size: AdSize.banner,
  listener: BannerAdListener(
    onAdLoaded: (ad) {
      // 获取 ResponseInfo
      final responseInfo = ad.responseInfo;
      final adapterClassName = responseInfo?.mediationAdapterClassName;
      
      debugPrint('Ad loaded from: $adapterClassName');
      
      setState(() {
        _bannerAd = ad as BannerAd;
        _bannerIsLoaded = true;
      });
    },
    onAdFailedToLoad: (ad, error) {
      debugPrint('Ad failed to load: ${error.message}');
      ad.dispose();
    },
  ),
  request: AdRequest(),
)..load();
```

### 完整示例

以下是示例项目中的完整实现：

```dart
void _loadBannerAd() {
  _bannerAd = BannerAd(
    size: AdSize.banner,
    adUnitId: '<your-ad-unit>',
    listener: BannerAdListener(
      onAdLoaded: (ad) {
        debugPrint(
          '$ad loaded: ${ad.responseInfo?.mediationAdapterClassName}',
        );
        setState(() {
          _bannerIsLoaded = true;
        });
      },
      onAdFailedToLoad: (ad, error) =>
          debugPrint('$ad failed to load: ${error.message}'),
    ),
    request: AdRequest(nonPersonalizedAds: true),
  )..load();
}
```

## 在 Interstitial Ad 中检测

### 基本检测

在 Interstitial Ad 的加载回调中检测：

```dart
InterstitialAd.load(
  adUnitId: _adUnitId,
  request: const AdRequest(),
  adLoadCallback: InterstitialAdLoadCallback(
    onAdLoaded: (InterstitialAd ad) {
      // 获取 ResponseInfo
      final responseInfo = ad.responseInfo;
      final adapterClassName = responseInfo?.mediationAdapterClassName;
      
      debugPrint('Interstitial ad loaded from: $adapterClassName');
      
      _interstitialAd = ad;
    },
    onAdFailedToLoad: (LoadAdError error) {
      debugPrint('Interstitial ad failed to load: $error');
    },
  ),
);
```

## 在 Rewarded Ad 中检测

### 基本检测

在 Rewarded Ad 的加载回调中检测：

```dart
RewardedAd.load(
  adUnitId: _adUnitId,
  request: const AdRequest(),
  rewardedAdLoadCallback: RewardedAdLoadCallback(
    onAdLoaded: (RewardedAd ad) {
      // 获取 ResponseInfo
      final responseInfo = ad.responseInfo;
      final adapterClassName = responseInfo?.mediationAdapterClassName;
      
      debugPrint('Rewarded ad loaded from: $adapterClassName');
      
      _rewardedAd = ad;
    },
    onAdFailedToLoad: (LoadAdError error) {
      debugPrint('Rewarded ad failed to load: $error');
    },
  ),
);
```

## 解析适配器类名

### 常见适配器类名

不同的适配器有不同的类名，以下是一些常见的：

- **AdMob**：`com.google.android.gms.ads.mediation.adapter.AdMobAdapter`
- **AppLovin**：`com.google.ads.mediation.applovin.ApplovinAdapter`
- **Unity Ads**：`com.google.ads.mediation.unity.UnityAdapter`
- **IronSource**：`com.google.ads.mediation.ironsource.IronSourceAdapter`
- **Vungle**：`com.google.ads.mediation.vungle.VungleAdapter`

### 提取网络名称

你可以从适配器类名中提取网络名称：

```dart
String? getNetworkName(String? adapterClassName) {
  if (adapterClassName == null) return null;
  
  // 提取最后一个点后面的部分
  final parts = adapterClassName.split('.');
  if (parts.isEmpty) return null;
  
  final adapterName = parts.last;
  
  // 移除 "Adapter" 后缀
  if (adapterName.endsWith('Adapter')) {
    return adapterName.substring(0, adapterName.length - 7);
  }
  
  return adapterName;
}

// 使用
final adapterClassName = ad.responseInfo?.mediationAdapterClassName;
final networkName = getNetworkName(adapterClassName);
debugPrint('Ad network: $networkName');
```

## 记录和分析数据

### 记录广告来源

你可以记录每个广告的来源，用于后续分析：

```dart
class AdNetworkTracker {
  final Map<String, int> _networkCounts = {};
  
  void recordAdNetwork(String? adapterClassName) {
    final networkName = getNetworkName(adapterClassName) ?? 'Unknown';
    _networkCounts[networkName] = (_networkCounts[networkName] ?? 0) + 1;
  }
  
  void printStatistics() {
    debugPrint('=== Ad Network Statistics ===');
    _networkCounts.forEach((network, count) {
      debugPrint('$network: $count ads');
    });
    debugPrint('============================');
  }
  
  Map<String, int> getStatistics() => Map.unmodifiable(_networkCounts);
}
```

### 使用示例

```dart
final _adNetworkTracker = AdNetworkTracker();

void _loadBannerAd() {
  _bannerAd = BannerAd(
    // ... 其他配置
    listener: BannerAdListener(
      onAdLoaded: (ad) {
        final adapterClassName = ad.responseInfo?.mediationAdapterClassName;
        _adNetworkTracker.recordAdNetwork(adapterClassName);
        
        debugPrint('Ad loaded from: ${getNetworkName(adapterClassName)}');
        
        setState(() {
          _bannerIsLoaded = true;
        });
      },
      // ... 其他回调
    ),
  )..load();
}
```

## 完整示例

以下是完整的广告来源检测实现：

```dart
class _MyHomePageState extends State<MyHomePage> {
  BannerAd? _bannerAd;
  bool _bannerIsLoaded = false;
  final _adNetworkTracker = AdNetworkTracker();

  void _loadBannerAd() {
    _bannerAd = BannerAd(
      size: AdSize.banner,
      adUnitId: '<your-ad-unit>',
      listener: BannerAdListener(
        onAdLoaded: (ad) {
          final responseInfo = ad.responseInfo;
          final adapterClassName = responseInfo?.mediationAdapterClassName;
          final networkName = getNetworkName(adapterClassName);
          
          debugPrint('Ad loaded from: $networkName');
          _adNetworkTracker.recordAdNetwork(adapterClassName);
          
          setState(() {
            _bannerAd = ad as BannerAd;
            _bannerIsLoaded = true;
          });
        },
        onAdFailedToLoad: (ad, error) {
          debugPrint('Ad failed to load: ${error.message}');
          ad.dispose();
        },
        onAdImpression: (ad) {
          final networkName = getNetworkName(ad.responseInfo?.mediationAdapterClassName);
          debugPrint('Ad impression from: $networkName');
        },
      ),
      request: AdRequest(nonPersonalizedAds: true),
    )..load();
  }

  String? getNetworkName(String? adapterClassName) {
    if (adapterClassName == null) return null;
    final parts = adapterClassName.split('.');
    if (parts.isEmpty) return null;
    final adapterName = parts.last;
    if (adapterName.endsWith('Adapter')) {
      return adapterName.substring(0, adapterName.length - 7);
    }
    return adapterName;
  }
}
```

## 使用广告来源信息优化

### 分析网络性能

根据记录的广告来源数据，你可以：

1. **识别表现好的网络**：哪些网络提供了最多的广告
2. **识别问题网络**：哪些网络经常失败或没有广告
3. **调整中介配置**：在 AdMob 控制台中调整网络优先级

### 示例分析

```dart
void analyzeAdNetworks() {
  final stats = _adNetworkTracker.getStatistics();
  
  if (stats.isEmpty) {
    debugPrint('No ad network data available.');
    return;
  }
  
  final totalAds = stats.values.reduce((a, b) => a + b);
  debugPrint('Total ads loaded: $totalAds');
  debugPrint('Network distribution:');
  
  stats.forEach((network, count) {
    final percentage = (count / totalAds * 100).toStringAsFixed(1);
    debugPrint('  $network: $count ($percentage%)');
  });
  
  // 找出表现最好的网络
  final topNetwork = stats.entries.reduce((a, b) => a.value > b.value ? a : b);
  debugPrint('Top network: ${topNetwork.key} (${topNetwork.value} ads)');
}
```

## 实践练习

1. 在 Banner Ad 中检测广告来源
2. 实现网络名称提取函数
3. 记录和分析广告来源数据
4. 使用数据优化中介配置

## 常见问题

### Q: 如果 `mediationAdapterClassName` 为 null 怎么办？

A: 这可能表示广告来自 AdMob 本身，或者信息不可用。你应该处理 null 情况。

### Q: 可以实时监控广告来源吗？

A: 可以。在每次广告加载时记录来源，并定期分析数据。

### Q: 如何知道哪个网络收益最高？

A: 需要结合 AdMob 控制台的数据和你的记录，分析每个网络的 eCPM 和总收益。

## 总结与检查清单

### 本章要点

- 使用 `ResponseInfo.mediationAdapterClassName` 检测广告来源
- 在所有广告格式中都可以检测来源
- 记录和分析广告来源数据有助于优化中介配置
- 根据数据调整网络优先级

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `ResponseInfo` 对象的使用
- [ ] 如何在各种广告格式中检测来源
- [ ] 如何解析适配器类名
- [ ] 如何记录和分析广告来源数据

下一章，我们将学习最佳实践和常见问题。
