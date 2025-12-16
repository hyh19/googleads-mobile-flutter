# 第 9 章 完整集成示例

## 引言

本章将展示完整的 `main.dart` 实现，将所有组件整合在一起，包括 SDK 初始化、同意管理、广告加载、介绍屏幕、广告显示和奖励处理。通过这个完整的示例，你将看到所有组件如何协同工作。

## 完整的 main.dart 实现

以下是完整的 `main.dart` 实现，包含所有必要的功能：

```dart
import 'dart:io';

import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

import 'ad_dialog.dart';
import 'app_bar_item.dart';
import 'countdown_timer.dart';
import 'consent_manager.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const MaterialApp(home: RewardedInterstitialExample()));
}

/// 一个加载并显示激励插屏广告的示例应用
class RewardedInterstitialExample extends StatefulWidget {
  const RewardedInterstitialExample({super.key});

  @override
  RewardedInterstitialExampleState createState() =>
      RewardedInterstitialExampleState();
}

class RewardedInterstitialExampleState
    extends State<RewardedInterstitialExample> {
  final _consentManager = ConsentManager();
  final CountdownTimer _countdownTimer = CountdownTimer(5);
  var _coins = 0;
  var _gamePaused = false;
  var _gameOver = false;
  var _isMobileAdsInitializeCalled = false;
  var _isPrivacyOptionsRequired = false;
  RewardedInterstitialAd? _rewardedInterstitialAd;

  final String _adUnitId = Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/5354046379'
      : 'ca-app-pub-3940256099942544/6978759866';

  @override
  void initState() {
    super.initState();

    // 收集用户同意
    _consentManager.gatherConsent((consentGatheringError) {
      if (consentGatheringError != null) {
        debugPrint(
          "${consentGatheringError.errorCode}: ${consentGatheringError.message}",
        );
      }

      // 开始第一局游戏
      _startNewGame();

      // 检查是否需要显示隐私选项入口
      _getIsPrivacyOptionsRequired();

      // 尝试初始化 Mobile Ads SDK
      _initializeMobileAdsSDK();
    });

    // 尝试使用之前会话中获得的同意来加载广告
    _initializeMobileAdsSDK();

    // 当倒计时器达到零时显示警告对话框
    _countdownTimer.addListener(
      () => setState(() {
        if (_countdownTimer.isComplete) {
          showDialog(
            context: context,
            builder: (context) => AdDialog(
              showAd: () {
                _gameOver = true;
                _showAdCallback();
              },
            ),
          );
          _coins += 1;
        }
      }),
    );
  }

  void _startNewGame() {
    _countdownTimer.start();
    _gameOver = false;
    _gamePaused = false;
  }

  void _pauseGame() {
    if (_gameOver || _gamePaused) {
      return;
    }
    _countdownTimer.pause();
    _gamePaused = true;
  }

  void _resumeGame() {
    if (_gameOver || !_gamePaused) {
      return;
    }
    _countdownTimer.resume();
    _gamePaused = false;
  }

  void _showAdCallback() {
    _rewardedInterstitialAd?.show(
      onUserEarnedReward: (AdWithoutView view, RewardItem rewardItem) {
        debugPrint('Reward amount: ${rewardItem.amount}');
        setState(() => _coins += rewardItem.amount.toInt());
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Rewarded Interstitial Example',
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Rewarded Interstitial Example'),
          actions: _appBarActions(),
        ),
        body: Stack(
          children: [
            const Align(
              alignment: Alignment.topCenter,
              child: Padding(
                padding: EdgeInsets.all(15),
                child: Text(
                  'The Impossible Game',
                  style: TextStyle(fontSize: 25, fontWeight: FontWeight.bold),
                ),
              ),
            ),
            Align(
              alignment: Alignment.center,
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text(
                    _countdownTimer.isComplete
                        ? 'Game over!'
                        : '${_countdownTimer.timeLeft} seconds left!',
                  ),
                  Visibility(
                    visible: _countdownTimer.isComplete,
                    child: TextButton(
                      onPressed: () {
                        _startNewGame();
                        _loadAd();
                      },
                      child: const Text('Play Again'),
                    ),
                  ),
                ],
              ),
            ),
            Align(
              alignment: Alignment.bottomLeft,
              child: Padding(
                padding: const EdgeInsets.all(15),
                child: Text('Coins: $_coins'),
              ),
            ),
          ],
        ),
      ),
    );
  }

  List<Widget> _appBarActions() {
    var array = [AppBarItem(AppBarItem.adInpsectorText, 0)];

    if (_isPrivacyOptionsRequired) {
      array.add(AppBarItem(AppBarItem.privacySettingsText, 1));
    }

    return <Widget>[
      PopupMenuButton<AppBarItem>(
        itemBuilder: (context) => array
            .map(
              (item) => PopupMenuItem<AppBarItem>(
                value: item,
                child: Text(item.label),
              ),
            )
            .toList(),
        onSelected: (item) {
          _pauseGame();
          switch (item.value) {
            case 0:
              MobileAds.instance.openAdInspector((error) {
                _resumeGame();
              });
            case 1:
              _consentManager.showPrivacyOptionsForm((formError) {
                if (formError != null) {
                  debugPrint("${formError.errorCode}: ${formError.message}");
                }
                _resumeGame();
              });
          }
        },
      ),
    ];
  }

  /// 加载激励插屏广告
  void _loadAd() async {
    // 只有在 Mobile Ads SDK 已收集与应用配置消息一致的同意时，才加载广告
    var canRequestAds = await _consentManager.canRequestAds();
    if (!canRequestAds) {
      return;
    }

    RewardedInterstitialAd.load(
      adUnitId: _adUnitId,
      request: const AdRequest(),
      rewardedInterstitialAdLoadCallback: RewardedInterstitialAdLoadCallback(
        onAdLoaded: (ad) {
          ad.fullScreenContentCallback = FullScreenContentCallback(
            // 当广告显示全屏内容时调用
            onAdShowedFullScreenContent: (ad) {},
            // 当广告产生展示时调用
            onAdImpression: (ad) {},
            // 当广告显示全屏内容失败时调用
            onAdFailedToShowFullScreenContent: (ad, err) {
              ad.dispose();
            },
            // 当广告关闭全屏内容时调用
            onAdDismissedFullScreenContent: (ad) {
              ad.dispose();
            },
            // 当广告被点击时调用
            onAdClicked: (ad) {},
          );

          // 保存广告引用，以便稍后显示
          _rewardedInterstitialAd = ad;
        },
        onAdFailedToLoad: (LoadAdError error) {
          print('RewardedInterstitialAd failed to load: $error');
        },
      ),
    );
  }

  /// 如果隐私选项入口是必需的，则重绘应用栏操作
  void _getIsPrivacyOptionsRequired() async {
    if (await _consentManager.isPrivacyOptionsRequired()) {
      setState(() {
        _isPrivacyOptionsRequired = true;
      });
    }
  }

  /// 如果 SDK 已收集与应用配置消息一致的同意，则初始化 Mobile Ads SDK
  void _initializeMobileAdsSDK() async {
    if (_isMobileAdsInitializeCalled) {
      return;
    }

    if (await _consentManager.canRequestAds()) {
      _isMobileAdsInitializeCalled = true;

      // 初始化 Mobile Ads SDK
      MobileAds.instance.initialize();

      // 加载广告
      _loadAd();
    }
  }

  @override
  void dispose() {
    _rewardedInterstitialAd?.dispose();
    _countdownTimer.dispose();
    super.dispose();
  }
}
```

## 代码流程解析

### 1. 应用启动流程

```
main() 
  → WidgetsFlutterBinding.ensureInitialized()
  → runApp()
  → RewardedInterstitialExample.initState()
    → gatherConsent() (收集用户同意)
    → _initializeMobileAdsSDK() (初始化 SDK)
    → _loadAd() (加载广告)
    → _countdownTimer.addListener() (监听游戏结束)
```

### 2. 用户同意流程

```
gatherConsent()
  → requestConsentInfoUpdate()
    → loadAndShowConsentFormIfRequired()
      → 用户选择同意/拒绝
      → canRequestAds() 返回 true/false
      → _initializeMobileAdsSDK()
```

### 3. 广告加载流程

```
_loadAd()
  → canRequestAds() (检查是否可以请求广告)
  → RewardedInterstitialAd.load()
    → onAdLoaded (成功)
      → 设置 FullScreenContentCallback
      → 保存广告引用
    → onAdFailedToLoad (失败)
      → 记录错误
```

### 4. 游戏和广告展示流程

```
游戏倒计时结束
  → _countdownTimer.isComplete
    → 显示 AdDialog (介绍屏幕)
      → 用户确认或倒计时结束
        → _showAdCallback()
          → _rewardedInterstitialAd?.show()
            → onUserEarnedReward (用户获得奖励)
              → 更新 _coins
            → onAdDismissedFullScreenContent (广告关闭)
              → ad.dispose()
              → _loadAd() (加载下一个广告)
```

## 关键组件集成

### 1. 同意管理与 SDK 初始化

```dart
_consentManager.gatherConsent((consentGatheringError) {
  // 处理同意收集结果
  _getIsPrivacyOptionsRequired();
  _initializeMobileAdsSDK();
});

_initializeMobileAdsSDK(); // 尝试使用之前的同意
```

### 2. 游戏逻辑与广告触发

```dart
_countdownTimer.addListener(
  () => setState(() {
    if (_countdownTimer.isComplete) {
      // 游戏结束，显示介绍屏幕
      showDialog(
        context: context,
        builder: (context) => AdDialog(
          showAd: () {
            _gameOver = true;
            _showAdCallback();
          },
        ),
      );
      // 游戏结束奖励
      _coins += 1;
    }
  }),
);
```

### 3. 广告显示与奖励处理

```dart
void _showAdCallback() {
  _rewardedInterstitialAd?.show(
    onUserEarnedReward: (AdWithoutView view, RewardItem rewardItem) {
      debugPrint('Reward amount: ${rewardItem.amount}');
      setState(() => _coins += rewardItem.amount.toInt());
    },
  );
}
```

### 4. 资源清理

```dart
@override
void dispose() {
  _rewardedInterstitialAd?.dispose();
  _countdownTimer.dispose();
  super.dispose();
}
```

## UI 结构

### 应用栏

- 标题："Rewarded Interstitial Example"
- 操作菜单：
  - Ad Inspector（广告检查器）
  - Privacy Settings（隐私设置，如果需要）

### 主界面

- **顶部**：游戏标题 "The Impossible Game"
- **中心**：游戏倒计时或游戏结束消息
- **底部**：金币显示 "Coins: X"

## 实践练习

1. 运行完整的示例代码
2. 测试用户同意流程
3. 测试广告加载和显示
4. 测试奖励机制
5. 测试游戏逻辑

## 常见问题

### Q: 为什么在 initState() 中调用两次 _initializeMobileAdsSDK()？

A: 第一次调用尝试使用之前会话中获得的同意，第二次调用在收集新的同意后执行。这确保了无论同意状态如何，SDK 都能正确初始化。

### Q: 如果用户拒绝同意，应用还能正常工作吗？

A: 可以，但无法加载和显示广告。游戏逻辑仍然可以正常工作。

### Q: 如何测试完整的流程？

A: 运行应用，等待游戏倒计时结束，查看介绍屏幕，观看广告，验证奖励是否正确发放。

## 总结与检查清单

### 本章要点

- 完整的集成包括：同意管理、SDK 初始化、广告加载、介绍屏幕、广告显示和奖励处理
- 所有组件需要正确协同工作
- 资源清理很重要，避免内存泄漏
- 用户体验应该流畅自然

### 检查清单

在继续下一章之前，确保你理解：

- [ ] 完整的应用启动流程
- [ ] 用户同意管理流程
- [ ] 广告加载和显示流程
- [ ] 游戏逻辑与广告的集成
- [ ] 资源清理的重要性

下一章，我们将学习最佳实践，帮助你优化实现并避免常见问题。
