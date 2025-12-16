# 第 4 章：显示广告和处理奖励

## 章节简介

在本章中，我们将学习如何显示已加载的 Rewarded 广告，并处理用户获得的奖励。这是 Rewarded Ads 的核心功能。我们将详细讲解 `show()` 方法的使用、`onUserEarnedReward` 回调、`RewardItem` 对象，以及如何实现奖励发放逻辑。

## 显示广告的流程

显示 Rewarded 广告的基本流程如下：

1. **检查广告可用性**：确保广告已加载且可用
2. **调用 show() 方法**：显示广告并传入奖励回调
3. **用户观看广告**：用户观看完整广告
4. **处理奖励**：通过 `onUserEarnedReward` 回调发放奖励

## show() 方法

### 基本用法

显示 Rewarded 广告需要使用 `show()` 方法，并传入 `onUserEarnedReward` 回调：

```dart
_rewardedAd?.show(
  onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
    debugPrint('Reward amount: ${rewardItem.amount}');
    debugPrint('Reward type: ${rewardItem.type}');
  },
);
```

**代码说明**：

- `_rewardedAd` 是已加载的 `RewardedAd` 对象
- 使用 `?.` 安全调用，如果广告为 `null` 则不会调用
- `onUserEarnedReward` 是必需的回调，用于处理用户获得的奖励

### 显示前检查

在显示广告之前，应该检查广告是否已加载：

```dart
void _showAd() {
  if (_rewardedAd != null) {
    _rewardedAd!.show(
      onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
        // 处理奖励
      },
    );
  } else {
    debugPrint('Ad not loaded yet');
  }
}
```

## onUserEarnedReward 回调

### 回调参数

`onUserEarnedReward` 回调接收两个参数：

- `ad`：`AdWithoutView` 对象，表示触发奖励的广告
- `rewardItem`：`RewardItem` 对象，包含奖励信息

### 回调触发时机

`onUserEarnedReward` 回调在以下时机触发：

- 用户观看完整广告后
- 用户完成广告要求的所有操作后
- 广告服务器确认用户有资格获得奖励后

**重要**：只有在用户完整观看广告并满足所有条件后，才会触发此回调。

## RewardItem 对象

### RewardItem 属性

`RewardItem` 对象包含以下属性：

- `type`：奖励类型（字符串），如 "coins"、"lives" 等
- `amount`：奖励数量（数字），表示奖励的数量

### 使用 RewardItem

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  debugPrint('Reward type: ${rewardItem.type}');
  debugPrint('Reward amount: ${rewardItem.amount}');
  
  // 根据奖励类型和数量发放奖励
  if (rewardItem.type == 'coins') {
    _coins += rewardItem.amount.toInt();
  } else if (rewardItem.type == 'lives') {
    _lives += rewardItem.amount.toInt();
  }
}
```

**代码说明**：

- `rewardItem.type` 是字符串类型，表示奖励类型
- `rewardItem.amount` 是数字类型，表示奖励数量
- 使用 `toInt()` 将数量转换为整数（如果需要）

## 完整的显示和奖励处理实现

以下是完整的显示广告和处理奖励的实现：

```dart
RewardedAd? _rewardedAd;
var _coins = 0;

void _showAd() {
  if (_rewardedAd != null) {
    _rewardedAd!.show(
      onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
        debugPrint('Reward amount: ${rewardItem.amount}');
        debugPrint('Reward type: ${rewardItem.type}');
        
        // 发放奖励
        setState(() {
          _coins += rewardItem.amount.toInt();
        });
        
        // 可以显示奖励提示
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text('You earned ${rewardItem.amount} ${rewardItem.type}!'),
          ),
        );
      },
    );
  } else {
    debugPrint('Ad not loaded yet');
  }
}
```

## 奖励发放逻辑

### 基本奖励发放

最简单的奖励发放方式是直接增加奖励数量：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  setState(() {
    _coins += rewardItem.amount.toInt();
  });
}
```

### 根据奖励类型发放

可以根据不同的奖励类型发放不同的奖励：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  setState(() {
    switch (rewardItem.type) {
      case 'coins':
        _coins += rewardItem.amount.toInt();
        break;
      case 'lives':
        _lives += rewardItem.amount.toInt();
        break;
      case 'energy':
        _energy += rewardItem.amount.toInt();
        break;
      default:
        debugPrint('Unknown reward type: ${rewardItem.type}');
    }
  });
}
```

### 奖励验证

在实际应用中，你可能需要验证奖励的有效性：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  // 验证奖励数量是否合理
  if (rewardItem.amount > 0) {
    setState(() {
      _coins += rewardItem.amount.toInt();
    });
    
    // 保存奖励到持久化存储
    _saveRewards();
  } else {
    debugPrint('Invalid reward amount: ${rewardItem.amount}');
  }
}
```

## 在实际场景中使用

### 游戏币奖励场景

在示例项目中，广告用于奖励游戏币：

```dart
var _coins = 0;

Visibility(
  visible: _showWatchVideoButton,
  child: TextButton(
    onPressed: () {
      setState(() => _showWatchVideoButton = false);
      _rewardedAd?.show(
        onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
          debugPrint('Reward amount: ${rewardItem.amount}');
          setState(() {
            _coins += rewardItem.amount.toInt();
          });
        },
      );
    },
    child: const Text('Watch video for additional 10 coins'),
  ),
)
```

**代码说明**：

- 显示「观看视频获得额外 10 个金币」按钮
- 用户点击按钮后显示广告
- 用户观看完整广告后获得奖励
- 奖励数量增加到 `_coins` 变量中

### 额外生命场景

在游戏应用中，可以奖励额外生命：

```dart
var _lives = 3;

void _showAdForExtraLife() {
  if (_rewardedAd != null) {
    _rewardedAd!.show(
      onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
        setState(() {
          _lives += rewardItem.amount.toInt();
        });
        
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('You earned ${rewardItem.amount} extra lives!')),
        );
      },
    );
  }
}
```

## 奖励显示和反馈

### 显示奖励提示

在用户获得奖励后，应该提供清晰的反馈：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  setState(() {
    _coins += rewardItem.amount.toInt();
  });
  
  // 显示奖励提示
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(
      content: Text('You earned ${rewardItem.amount} ${rewardItem.type}!'),
      duration: Duration(seconds: 2),
    ),
  );
}
```

### 更新 UI

确保在获得奖励后更新 UI：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  setState(() {
    _coins += rewardItem.amount.toInt();
  });
  
  // UI 会自动更新，因为调用了 setState
}
```

## 常见问题

### 奖励没有发放

可能的原因：

1. **用户未完整观看广告**：只有完整观看广告才会触发奖励
2. **回调未实现**：必须实现 `onUserEarnedReward` 回调
3. **奖励数量为 0**：检查 `rewardItem.amount` 是否大于 0

### 奖励数量不正确

**解决方案**：检查 `rewardItem.amount` 的值，确保正确使用：

```dart
onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
  debugPrint('Reward amount: ${rewardItem.amount}');
  debugPrint('Reward type: ${rewardItem.type}');
  
  // 确保数量大于 0
  if (rewardItem.amount > 0) {
    _coins += rewardItem.amount.toInt();
  }
}
```

### 可以重复获得奖励吗？

每个广告只能触发一次奖励。如果用户再次观看同一个广告，不会再次获得奖励。需要加载新的广告才能再次获得奖励。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现 _showAd() 方法**：创建显示广告的方法
2. **实现奖励处理**：在 `onUserEarnedReward` 回调中实现奖励发放逻辑
3. **显示奖励反馈**：在用户获得奖励后显示提示
4. **测试奖励功能**：运行应用并测试奖励是否正确发放

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解显示广告的基本流程
- [ ] 使用 `show()` 方法显示广告
- [ ] 实现 `onUserEarnedReward` 回调处理奖励
- [ ] 理解 `RewardItem` 对象的属性和用法
- [ ] 实现奖励发放逻辑
- [ ] 根据奖励类型发放不同的奖励
- [ ] 显示奖励反馈给用户
- [ ] 在显示前检查广告是否可用

## 下一步

现在我们已经实现了广告的显示和奖励处理功能。在下一章中，我们将学习如何处理广告事件，包括广告展示、关闭、点击等事件。

继续学习：[第 5 章：处理广告事件](chapter-05-ad-events.md)
