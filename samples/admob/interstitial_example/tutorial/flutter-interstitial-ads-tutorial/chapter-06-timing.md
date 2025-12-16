# 第 6 章：显示时机选择

## 章节简介

在本章中，我们将学习如何选择合适的时机显示 Interstitial 广告。显示时机的选择对于用户体验和广告效果都非常重要。我们将详细讲解何时显示广告、如何避免打断用户操作，以及预加载策略。

## 为什么显示时机很重要

### 用户体验影响

选择合适的显示时机可以：

- **提高用户满意度**：在自然过渡点显示，不会让用户感到突兀
- **提高广告效果**：用户更愿意在合适的时机与广告交互
- **避免用户流失**：不当的显示时机可能导致用户卸载应用

### 广告效果影响

合适的显示时机可以：

- **提高点击率**：用户在合适的时机更可能点击广告
- **提高收益**：更好的用户体验带来更高的广告收益
- **提高填充率**：合适的时机可以提高广告填充率

## 好的显示时机

### 1. 游戏关卡结束

**场景**：玩家完成一个关卡后

**示例**：

```dart
void _levelCompleted() {
  // 显示关卡完成界面
  showDialog(
    context: context,
    builder: (context) => AlertDialog(
      title: Text('Level Completed!'),
      content: Text('Congratulations!'),
      actions: [
        TextButton(
          onPressed: () {
            Navigator.pop(context);
            // 在用户确认后显示广告
            _interstitialAd?.show();
            // 然后加载下一个关卡
            _loadNextLevel();
          },
          child: Text('Next Level'),
        ),
      ],
    ),
  );
}
```

**优势**：

- 自然的过渡点
- 用户已完成当前任务
- 不会打断游戏体验

### 2. 任务完成

**场景**：用户完成任务或达到里程碑

**示例**：

```dart
void _taskCompleted() {
  // 更新任务状态
  _updateTaskStatus();
  
  // 显示完成提示
  _showCompletionMessage();
  
  // 显示广告
  _interstitialAd?.show();
  
  // 继续后续操作
  _navigateToNextTask();
}
```

### 3. 内容切换

**场景**：从一个内容切换到另一个内容时

**示例**：

```dart
void _switchContent() {
  // 显示广告
  _interstitialAd?.show();
  
  // 广告关闭后切换内容
  // （通过 onAdDismissedFullScreenContent 回调处理）
}
```

### 4. 应用恢复（谨慎使用）

**场景**：从后台恢复应用时

**注意**：这个场景需要谨慎使用，因为可能会打断用户操作。更推荐使用 App Open Ads。

## 不好的显示时机

### 1. 用户正在操作时

**问题**：打断用户操作会导致糟糕的用户体验

**示例（错误）**：

```dart
void _userAction() {
  // 用户正在执行操作
  _processUserInput();
  
  // 错误：在用户操作过程中显示广告
  _interstitialAd?.show(); // 不要这样做！
}
```

### 2. 频繁显示

**问题**：在短时间内重复显示广告会让用户感到厌烦

**解决方案**：实现最小时间间隔

```dart
DateTime? _lastAdShownTime;
final Duration _minTimeBetweenAds = Duration(minutes: 5);

void _showAdIfAppropriate() {
  if (_lastAdShownTime != null &&
      DateTime.now().difference(_lastAdShownTime!) < _minTimeBetweenAds) {
    return; // 距离上次显示时间太短，不显示
  }
  
  if (_interstitialAd != null) {
    _interstitialAd!.show();
    _lastAdShownTime = DateTime.now();
  }
}
```

### 3. 应用启动时立即显示

**问题**：应用启动时立即显示广告会让用户感到突兀

**解决方案**：等待用户使用应用一段时间后再显示

```dart
int _appLaunchCount = 0;

void _onAppLaunch() {
  _appLaunchCount++;
  
  // 只在用户使用应用几次后显示广告
  if (_appLaunchCount >= 3) {
    _interstitialAd?.show();
  }
}
```

## 预加载策略

### 为什么需要预加载

Interstitial Ads 的加载通常需要 1-3 秒，如果等到需要显示时再加载，用户会看到明显的延迟。预加载可以确保在需要时广告已经准备好了。

### 何时预加载

推荐在以下时机预加载：

1. **应用启动时**：在应用初始化后立即加载第一个广告
2. **广告关闭后**：在 `onAdDismissedFullScreenContent` 回调中立即加载下一个广告
3. **任务开始前**：在用户开始新任务前预加载广告

### 预加载实现

```dart
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
        _interstitialAd = ad;
        // 设置回调
        ad.fullScreenContentCallback = FullScreenContentCallback(
          onAdDismissedFullScreenContent: (ad) {
            ad.dispose();
            _interstitialAd = null;
            // 立即加载下一个广告
            _loadAd();
          },
        );
      },
      onAdFailedToLoad: (LoadAdError error) {
        // 可以在这里实现重试逻辑
      },
    ),
  );
}
```

## 实际应用示例

### 游戏应用示例

在示例项目中，广告在游戏结束时显示：

```dart
void _startTimer() {
  _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
    setState(() => _counter--);

    if (_counter == 0) {
      _gameOver = true;
      _showAlert(context); // 显示游戏结束对话框
      timer.cancel();
    }
  });
}

void _showAlert(BuildContext context) {
  showDialog(
    context: context,
    builder: (context) => AlertDialog(
      title: const Text('Game Over'),
      content: Text('You lasted $_gameLength seconds'),
      actions: <Widget>[
        TextButton(
          onPressed: () {
            Navigator.pop(context);
            // 在用户确认后显示广告
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

- 游戏倒计时结束后显示游戏结束对话框
- 用户在对话框中点击「OK」后显示广告
- 这是一个自然的过渡点，不会打断游戏体验

### 任务完成示例

```dart
void _completeTask() {
  // 完成任务逻辑
  _updateTaskProgress();
  _saveTaskData();
  
  // 显示完成提示
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(content: Text('Task completed!')),
  );
  
  // 延迟显示广告，给用户时间看到完成提示
  Future.delayed(Duration(seconds: 1), () {
    _interstitialAd?.show();
  });
}
```

## 避免打断用户操作

### 检查用户状态

在显示广告前，检查用户是否正在执行重要操作：

```dart
bool _isUserInteracting = false;

void _showAdIfAppropriate() {
  if (_isUserInteracting) {
    return; // 用户正在操作，不显示广告
  }
  
  if (_interstitialAd != null) {
    _interstitialAd!.show();
  }
}
```

### 等待合适的时机

如果当前不是合适的时机，可以等待：

```dart
void _waitForAppropriateTime() {
  // 等待用户完成当前操作
  Future.delayed(Duration(seconds: 2), () {
    if (!_isUserInteracting && _interstitialAd != null) {
      _interstitialAd!.show();
    }
  });
}
```

## 最佳实践总结

### 推荐做法

1. **在自然过渡点显示**：关卡结束、任务完成等
2. **预加载广告**：确保在需要时广告已准备好
3. **实现时间间隔**：避免频繁显示
4. **检查用户状态**：避免打断用户操作
5. **立即加载下一个**：广告关闭后立即加载

### 避免的做法

1. **不要打断用户操作**
2. **不要频繁显示**
3. **不要在应用启动时立即显示**
4. **不要在用户正在输入时显示**

## 实践练习

完成以下练习以巩固本章内容：

1. **分析显示时机**：列出你的应用中适合显示广告的时机
2. **实现时间间隔**：添加最小时间间隔保护
3. **实现预加载**：在适当时机预加载广告
4. **测试显示时机**：在不同场景下测试广告显示

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解为什么显示时机很重要
- [ ] 识别好的显示时机
- [ ] 避免不好的显示时机
- [ ] 实现预加载策略
- [ ] 实现时间间隔保护
- [ ] 避免打断用户操作
- [ ] 在实际应用中应用这些原则

## 下一步

现在我们已经学会了如何选择合适的显示时机。在下一章中，我们将学习如何实现用户同意管理（UMP），以符合 GDPR 等隐私法规的要求。

继续学习：[第 7 章：用户同意管理（UMP）](chapter-07-consent-management.md)
