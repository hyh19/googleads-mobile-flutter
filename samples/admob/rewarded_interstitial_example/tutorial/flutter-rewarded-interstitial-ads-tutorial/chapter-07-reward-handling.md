# 第 7 章 奖励机制实现

## 引言

奖励机制是 Rewarded Interstitial Ads 的核心功能。用户观看广告后应该立即获得奖励，并且奖励应该与应用逻辑（如游戏逻辑）无缝集成。本章将详细讲解如何实现奖励机制，包括游戏逻辑集成、奖励累积和状态管理。

## 游戏逻辑示例

示例项目使用一个简单的倒计时游戏来演示奖励机制。让我们看看如何实现：

### 游戏状态

```dart
class RewardedInterstitialExampleState extends State<RewardedInterstitialExample> {
  final CountdownTimer _countdownTimer = CountdownTimer(5);
  var _coins = 0;
  var _gamePaused = false;
  var _gameOver = false;
  
  // ... 其他代码
}
```

- `_coins`：用户拥有的金币数量
- `_gamePaused`：游戏是否暂停
- `_gameOver`：游戏是否结束
- `_countdownTimer`：倒计时器

## CountdownTimer 实现

`CountdownTimer` 是一个自定义的倒计时器类，用于管理游戏倒计时：

```dart
enum CountdownState { notStarted, active, paused, ended }

class CountdownTimer extends ChangeNotifier {
  final int _countdownTime;
  late var timeLeft = _countdownTime;
  var _countdownState = CountdownState.notStarted;
  bool get isComplete => _countdownState == CountdownState.ended;
  Timer? _timer;

  CountdownTimer(this._countdownTime);

  void start() {
    timeLeft = _countdownTime;
    _startTimer();
    _countdownState = CountdownState.active;
    notifyListeners();
  }

  void resume() {
    if (_countdownState != CountdownState.paused) {
      return;
    }
    _startTimer();
    _countdownState = CountdownState.active;
  }

  void pause() {
    if (_countdownState != CountdownState.active) {
      return;
    }
    _timer?.cancel();
    _countdownState = CountdownState.paused;
  }

  void _startTimer() {
    _timer = Timer.periodic(const Duration(seconds: 1), (timer) {
      timeLeft--;
      if (timeLeft == 0) {
        _countdownState = CountdownState.ended;
        timer.cancel();
      }
      notifyListeners();
    });
  }
}
```

### 使用 CountdownTimer

在 `initState()` 中，我们设置倒计时器的监听器：

```dart
@override
void initState() {
  super.initState();
  
  // 监听倒计时器变化
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
        // 游戏结束时奖励 1 个金币
        _coins += 1;
      }
    }),
  );
}
```

## 奖励累积机制

### 在游戏结束时奖励

在示例项目中，当倒计时结束时，用户自动获得 1 个金币：

```dart
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
      // 游戏结束奖励
      _coins += 1;
    }
  }),
);
```

### 在观看广告后奖励

当用户完成观看广告后，在 `onUserEarnedReward` 回调中发放奖励：

```dart
void _showAdCallback() {
  _rewardedInterstitialAd?.show(
    onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
      debugPrint('Reward amount: ${rewardItem.amount}');
      // 将广告奖励添加到总金币数
      setState(() {
        _coins += rewardItem.amount.toInt();
      });
    },
  );
}
```

### 完整的奖励流程

1. **游戏结束**：用户完成游戏，自动获得 1 个金币
2. **显示介绍屏幕**：向用户提供观看广告获得额外奖励的机会
3. **用户观看广告**：用户选择观看广告
4. **发放广告奖励**：在 `onUserEarnedReward` 回调中发放广告奖励（如 10 个金币）
5. **更新 UI**：使用 `setState()` 更新金币显示

## 状态管理

### 使用 setState() 更新状态

当奖励发放时，使用 `setState()` 更新 UI：

```dart
setState(() {
  _coins += rewardItem.amount.toInt();
});
```

### 游戏状态管理

管理游戏的不同状态：

```dart
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
```

## UI 显示

### 显示金币数量

在 UI 中显示用户的金币数量：

```dart
Align(
  alignment: Alignment.bottomLeft,
  child: Padding(
    padding: const EdgeInsets.all(15),
    child: Text('Coins: $_coins'),
  ),
),
```

### 显示游戏状态

显示游戏倒计时和状态：

```dart
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
```

## 完整的奖励机制实现

以下是完整的奖励机制实现示例：

```dart
class RewardedInterstitialExampleState extends State<RewardedInterstitialExample> {
  final CountdownTimer _countdownTimer = CountdownTimer(5);
  var _coins = 0;
  var _gamePaused = false;
  var _gameOver = false;
  RewardedInterstitialAd? _rewardedInterstitialAd;

  @override
  void initState() {
    super.initState();
    
    // 监听倒计时器
    _countdownTimer.addListener(
      () => setState(() {
        if (_countdownTimer.isComplete) {
          // 游戏结束，奖励 1 个金币
          _coins += 1;
          
          // 显示介绍屏幕
          showDialog(
            context: context,
            builder: (context) => AdDialog(
              showAd: () {
                _gameOver = true;
                _showAdCallback();
              },
            ),
          );
        }
      }),
    );
  }

  void _startNewGame() {
    _countdownTimer.start();
    _gameOver = false;
    _gamePaused = false;
  }

  void _showAdCallback() {
    _rewardedInterstitialAd?.show(
      onUserEarnedReward: (AdWithoutView ad, RewardItem rewardItem) {
        debugPrint('Reward amount: ${rewardItem.amount}');
        // 发放广告奖励
        setState(() {
          _coins += rewardItem.amount.toInt();
        });
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          // 游戏内容
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
          // 金币显示
          Align(
            alignment: Alignment.bottomLeft,
            child: Padding(
              padding: const EdgeInsets.all(15),
              child: Text('Coins: $_coins'),
            ),
          ),
        ],
      ),
    );
  }
}
```

## 持久化奖励数据

在生产环境中，你可能需要将奖励数据持久化到本地存储：

```dart
import 'package:shared_preferences/shared_preferences.dart';

Future<void> _saveCoins() async {
  final prefs = await SharedPreferences.getInstance();
  await prefs.setInt('coins', _coins);
}

Future<void> _loadCoins() async {
  final prefs = await SharedPreferences.getInstance();
  setState(() {
    _coins = prefs.getInt('coins') ?? 0;
  });
}
```

## 实践练习

1. 实现游戏逻辑和倒计时器
2. 集成奖励累积机制
3. 实现游戏状态管理
4. 添加奖励持久化（可选）

## 常见问题

### Q: 我应该在哪里发放奖励？

A: 奖励应该在 `onUserEarnedReward` 回调中立即发放。不要延迟或要求额外操作。

### Q: 如何确保奖励数据不丢失？

A: 使用本地存储（如 SharedPreferences）或服务器端存储来持久化奖励数据。

### Q: 如果用户关闭应用，奖励会丢失吗？

A: 如果奖励只存储在内存中（如 `_coins` 变量），应用关闭后数据会丢失。建议使用持久化存储。

### Q: 我可以限制用户每天观看广告的次数吗？

A: 可以。你可以记录用户观看广告的次数和时间，并在显示介绍屏幕前检查限制。

## 总结与检查清单

### 本章要点

- 奖励应该在 `onUserEarnedReward` 回调中立即发放
- 使用 `setState()` 更新 UI 状态
- 实现游戏逻辑与奖励机制的集成
- 考虑使用持久化存储保存奖励数据

### 检查清单

在继续下一章之前，确保你理解：

- [ ] 如何实现奖励累积机制
- [ ] 如何使用 `CountdownTimer` 管理游戏逻辑
- [ ] 如何在 `onUserEarnedReward` 中发放奖励
- [ ] 如何更新 UI 显示奖励信息

下一章，我们将学习用户同意管理（UMP），这对于符合 GDPR 要求非常重要。
