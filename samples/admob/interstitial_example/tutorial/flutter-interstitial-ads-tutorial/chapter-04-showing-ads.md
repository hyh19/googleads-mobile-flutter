# 第 4 章：显示插屏广告

## 章节简介

在本章中，我们将学习如何显示已加载的 Interstitial 广告。我们将详细讲解 `show()` 方法的使用、显示前的检查、显示时机选择，以及如何正确管理广告资源。

## 显示广告的流程

显示 Interstitial 广告的基本流程如下：

1. **检查广告可用性**：确保广告已加载且可用
2. **调用 show() 方法**：显示广告
3. **处理广告事件**：通过回调处理广告的展示、关闭等事件

## show() 方法

### 基本用法

显示 Interstitial 广告非常简单，只需要调用 `show()` 方法：

```dart
_interstitialAd?.show();
```

**代码说明**：

- `_interstitialAd` 是已加载的 `InterstitialAd` 对象
- 使用 `?.` 安全调用，如果广告为 `null` 则不会调用
- `show()` 方法会立即显示广告

### 显示前检查

在显示广告之前，应该检查广告是否已加载：

```dart
void _showAd() {
  if (_interstitialAd != null) {
    _interstitialAd!.show();
  } else {
    debugPrint('Ad not loaded yet');
  }
}
```

**代码说明**：

- 检查 `_interstitialAd` 是否为 `null`
- 如果不为 `null`，使用 `!` 断言非空并调用 `show()`
- 如果为 `null`，记录日志提示广告尚未加载

## 完整的显示实现

以下是完整的显示广告的实现，包含检查和处理：

```dart
void _showAd() {
  if (_interstitialAd != null) {
    _interstitialAd!.show();
  } else {
    debugPrint('Ad not loaded yet. Loading ad...');
    _loadAd();
  }
}
```

**代码说明**：

- 如果广告已加载，直接显示
- 如果广告未加载，尝试加载广告（但不会立即显示，因为加载是异步的）

## 在实际场景中使用

### 游戏结束场景

在游戏结束或关卡完成时显示广告：

```dart
void _gameOver() {
  // 显示游戏结束对话框
  showDialog(
    context: context,
    builder: (context) => AlertDialog(
      title: const Text('Game Over'),
      content: Text('You lasted $_gameLength seconds'),
      actions: <Widget>[
        TextButton(
          onPressed: () {
            Navigator.pop(context);
            // 显示广告
            _interstitialAd?.show();
          },
          child: const Text('OK'),
        ),
      ],
    ),
  );
}
```

**代码说明**：

- 在用户确认游戏结束对话框后显示广告
- 这是一个自然的过渡点，不会打断用户操作

### 任务完成场景

在任务完成时显示广告：

```dart
void _taskCompleted() {
  // 完成任务逻辑
  _updateProgress();
  
  // 显示广告
  _interstitialAd?.show();
  
  // 继续后续操作
  _navigateToNextScreen();
}
```

## 显示时机的重要性

### 好的显示时机

- **游戏关卡结束**：玩家完成关卡后
- **任务完成**：用户完成任务后
- **内容切换**：从一个内容切换到另一个内容时
- **里程碑达成**：用户达到某个里程碑时

### 不好的显示时机

- **用户正在操作时**：不要打断用户操作
- **频繁显示**：不要在短时间内重复显示
- **应用启动时**：不要立即显示（除非是 App Open Ad）

## 资源管理

### 为什么需要资源管理

Interstitial 广告对象占用系统资源，如果不及时清理，可能导致内存泄漏。每个广告对象只能显示一次，显示后必须释放资源。

### 释放时机

广告资源应该在以下情况下释放：

1. **广告关闭后**：在 `onAdDismissedFullScreenContent` 回调中
2. **广告显示失败后**：在 `onAdFailedToShowFullScreenContent` 回调中
3. **Widget 销毁时**：在 `dispose()` 方法中

### dispose() 方法

`dispose()` 方法会：

- 释放广告占用的内存
- 取消所有待处理的请求
- 清理内部状态

**重要**：调用 `dispose()` 后，广告对象不能再使用。如果需要再次显示广告，必须加载新的广告。

## 完整的显示流程

以下是包含资源管理的完整显示流程：

```dart
InterstitialAd? _interstitialAd;

void _loadAd() async {
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  InterstitialAd.load(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    adLoadCallback: InterstitialAdLoadCallback(
      onAdLoaded: (InterstitialAd ad) {
        debugPrint('Ad was loaded.');
        _interstitialAd = ad;
        
        // 设置事件回调（我们将在下一章详细讲解）
        ad.fullScreenContentCallback = FullScreenContentCallback(
          onAdDismissedFullScreenContent: (ad) {
            debugPrint('Ad was dismissed.');
            ad.dispose(); // 释放资源
            _interstitialAd = null;
            // 可以在这里加载下一个广告
            _loadAd();
          },
        );
      },
      onAdFailedToLoad: (LoadAdError error) {
        debugPrint('Ad failed to load: $error');
      },
    ),
  );
}

void _showAd() {
  if (_interstitialAd != null) {
    _interstitialAd!.show();
  }
}

@override
void dispose() {
  _interstitialAd?.dispose(); // 清理资源
  super.dispose();
}
```

## 测试广告显示

### 创建测试应用

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

class InterstitialTestPage extends StatefulWidget {
  @override
  _InterstitialTestPageState createState() => _InterstitialTestPageState();
}

class _InterstitialTestPageState extends State<InterstitialTestPage> {
  InterstitialAd? _interstitialAd;

  @override
  void initState() {
    super.initState();
    _loadAd();
  }

  void _loadAd() {
    // 加载广告的实现
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Interstitial Test')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            _interstitialAd?.show();
          },
          child: Text('Show Ad'),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _interstitialAd?.dispose();
    super.dispose();
  }
}
```

### 观察广告显示

运行应用并点击「Show Ad」按钮，应该看到全屏广告显示。

## 常见问题

### 广告不显示

可能的原因：

1. **广告未加载**：检查 `_interstitialAd` 是否为 `null`
2. **广告已显示过**：每个广告对象只能显示一次
3. **广告已释放**：检查是否在显示前调用了 `dispose()`

### 可以重复使用同一个广告对象吗？

不可以。每个 `InterstitialAd` 对象只能显示一次。显示完成后必须调用 `dispose()` 释放资源，然后加载新的广告。

### 什么时候加载下一个广告？

推荐在广告关闭后立即加载下一个广告，这样在需要显示时广告已经准备好了。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现 _showAd() 方法**：创建显示广告的方法
2. **添加显示前检查**：确保广告已加载
3. **测试显示功能**：创建一个测试应用来验证广告显示
4. **实现资源管理**：确保在适当时机释放资源

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解显示广告的基本流程
- [ ] 使用 `show()` 方法显示广告
- [ ] 在显示前检查广告是否可用
- [ ] 选择合适的显示时机
- [ ] 理解资源管理的重要性
- [ ] 在适当时机释放广告资源
- [ ] 理解每个广告对象只能显示一次

## 下一步

现在我们已经实现了广告的加载和显示功能。在下一章中，我们将学习如何处理广告事件，包括广告展示、关闭、点击等事件。

继续学习：[第 5 章：处理广告事件](chapter-05-ad-events.md)
