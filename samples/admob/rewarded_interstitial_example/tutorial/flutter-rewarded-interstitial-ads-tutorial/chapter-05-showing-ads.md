# 第 5 章 显示广告和处理奖励

## 引言

在用户通过介绍屏幕确认观看广告后，我们需要显示广告并处理用户获得的奖励。本章将详细讲解如何使用 `RewardedInterstitialAd.show()` 方法显示广告，以及如何处理 `onUserEarnedReward` 回调来发放奖励。

## 显示广告

### show() 方法

`RewardedInterstitialAd.show()` 方法用于显示已加载的广告。它接受一个 `onUserEarnedReward` 回调，当用户完成观看广告并获得奖励时触发。

```dart
void _showAdCallback() {
  _rewardedInterstitialAd?.show(
    onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
      debugPrint('Reward amount: ${rewardItem.amount}');
      setState(() => _coins += rewardItem.amount.toInt());
    },
  );
}
```

### 检查广告是否可用

在显示广告之前，应该检查广告是否已加载并可用：

```dart
void _showAdCallback() {
  if (_rewardedInterstitialAd == null) {
    debugPrint('RewardedInterstitialAd is not loaded.');
    return;
  }

  _rewardedInterstitialAd?.show(
    onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
      debugPrint('Reward amount: ${rewardItem.amount}');
      setState(() => _coins += rewardItem.amount.toInt());
    },
  );
}
```

## 处理奖励

### onUserEarnedReward 回调

当用户完成观看广告并获得奖励时，`onUserEarnedReward` 回调会被触发。这个回调提供两个参数：

1. `AdWithoutView ad`：广告对象
2. `RewardItem rewardItem`：奖励信息对象

### RewardItem 对象

`RewardItem` 对象包含以下信息：

- `amount`：奖励数量（`double` 类型）
- `type`：奖励类型（`String` 类型）

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  debugPrint('Reward amount: ${rewardItem.amount}');
  debugPrint('Reward type: ${rewardItem.type}');
  
  // 将奖励添加到用户账户
  setState(() => _coins += rewardItem.amount.toInt());
},
```

### 奖励发放时机

**重要**：奖励应该在 `onUserEarnedReward` 回调中立即发放。不要延迟奖励发放，也不要要求用户执行额外操作。

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  // ✅ 正确：立即发放奖励
  setState(() => _coins += rewardItem.amount.toInt());
  
  // ❌ 错误：延迟发放或要求额外操作
  // Future.delayed(Duration(seconds: 5), () {
  //   setState(() => _coins += rewardItem.amount.toInt());
  // });
},
```

## 完整的显示流程

以下是显示广告和处理奖励的完整流程：

```dart
void _showAdCallback() {
  // 检查广告是否可用
  if (_rewardedInterstitialAd == null) {
    debugPrint('RewardedInterstitialAd is not loaded.');
    // 可以在这里加载新广告
    _loadAd();
    return;
  }

  // 显示广告
  _rewardedInterstitialAd?.show(
    onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
      debugPrint('Reward amount: ${rewardItem.amount}');
      debugPrint('Reward type: ${rewardItem.type}');
      
      // 立即发放奖励
      setState(() {
        _coins += rewardItem.amount.toInt();
      });
      
      // 可以在这里显示奖励通知
      _showRewardNotification(rewardItem.amount.toInt());
    },
  );
}
```

## 奖励通知

为了提供更好的用户体验，你可以在用户获得奖励后显示通知：

```dart
void _showRewardNotification(int amount) {
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(
      content: Text('You earned $amount coins!'),
      duration: const Duration(seconds: 2),
      backgroundColor: Colors.green,
    ),
  );
}
```

或者使用更丰富的通知：

```dart
void _showRewardNotification(int amount) {
  showDialog(
    context: context,
    builder: (context) => AlertDialog(
      title: const Text('Reward Earned!'),
      content: Text('You earned $amount coins for watching the ad.'),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: const Text('OK'),
        ),
      ],
    ),
  );
}
```

## 与介绍屏幕的集成

在介绍屏幕（AdDialog）中，当用户确认观看广告或倒计时结束时，我们调用 `_showAdCallback()`：

```dart
showDialog(
  context: context,
  builder: (context) => AdDialog(
    showAd: () {
      _gameOver = true;
      _showAdCallback();
    },
  ),
);
```

## 错误处理

如果广告显示失败，`FullScreenContentCallback` 的 `onAdFailedToShowFullScreenContent` 回调会被触发（我们将在第 6 章详细讲解）。你应该在那里处理错误：

```dart
ad.fullScreenContentCallback = FullScreenContentCallback(
  // ... 其他回调
  onAdFailedToShowFullScreenContent: (ad, err) {
    debugPrint('Ad failed to show: $err');
    ad.dispose();
    _rewardedInterstitialAd = null;
    // 可以在这里显示错误消息或加载新广告
  },
);
```

## 完整示例

以下是完整的显示广告和处理奖励的实现：

```dart
class RewardedInterstitialExampleState extends State<RewardedInterstitialExample> {
  RewardedInterstitialAd? _rewardedInterstitialAd;
  var _coins = 0;

  void _showAdCallback() {
    if (_rewardedInterstitialAd == null) {
      debugPrint('RewardedInterstitialAd is not loaded.');
      _loadAd();
      return;
    }

    _rewardedInterstitialAd?.show(
      onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
        debugPrint('Reward amount: ${rewardItem.amount}');
        debugPrint('Reward type: ${rewardItem.type}');
        
        setState(() {
          _coins += rewardItem.amount.toInt();
        });
        
        // 显示奖励通知
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('You earned ${rewardItem.amount.toInt()} coins!'),
            duration: const Duration(seconds: 2),
          ),
        );
      },
    );
  }

  // 在游戏结束时显示介绍屏幕
  void _onGameOver() {
    showDialog(
      context: context,
      builder: (context) => AdDialog(
        showAd: () {
          _showAdCallback();
        },
      ),
    );
  }
}
```

## 实践练习

1. 实现 `_showAdCallback()` 方法
2. 处理 `onUserEarnedReward` 回调
3. 实现奖励发放逻辑
4. 添加奖励通知功能（可选）

## 常见问题

### Q: 如果用户没有完成观看广告，会获得奖励吗？

A: 不会。只有当用户完成观看广告后，`onUserEarnedReward` 回调才会被触发。

### Q: 奖励数量是由我控制的吗？

A: 奖励数量由你在 AdMob 控制台中配置的广告单元设置决定。你可以在 `onUserEarnedReward` 回调中读取 `rewardItem.amount` 来获取奖励数量。

### Q: 我可以在显示广告前再次检查广告是否可用吗？

A: 可以，而且应该这样做。在调用 `show()` 之前检查 `_rewardedInterstitialAd` 是否为 `null`。

### Q: 如果广告显示失败，我应该重新加载吗？

A: 是的。在 `onAdFailedToShowFullScreenContent` 回调中，你应该清理当前广告并加载新广告。

## 总结与检查清单

### 本章要点

- 使用 `RewardedInterstitialAd.show()` 显示广告
- 在 `onUserEarnedReward` 回调中处理奖励
- 立即发放奖励，不要延迟
- 检查广告是否可用后再显示

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `show()` 方法的使用
- [ ] `onUserEarnedReward` 回调的处理
- [ ] `RewardItem` 对象的使用
- [ ] 如何立即发放奖励
- [ ] 如何显示奖励通知

下一章，我们将详细学习如何处理广告的各种事件，包括显示、关闭、点击等。
