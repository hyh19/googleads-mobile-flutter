# 第 4 章 介绍屏幕（AdDialog）实现

## 引言

介绍屏幕（Intro Screen）是 Rewarded Interstitial Ads 的**强制要求**。根据 Google 的广告政策，在显示 Rewarded Interstitial Ad 之前，你必须向用户展示一个介绍屏幕，说明即将播放广告，并提供跳过选项。本章将详细讲解如何实现这个重要的组件。

## Google 政策要求

在显示 Rewarded Interstitial Ad 之前，你必须：

1. **展示介绍屏幕**：向用户说明即将播放广告
2. **提供奖励信息**：明确告知用户观看广告后能获得什么奖励
3. **提供跳过选项**：必须提供"跳过"或"No thanks"按钮，允许用户选择不观看广告
4. **倒计时提示**（推荐）：显示倒计时，让用户知道广告即将开始

不遵守这些要求可能导致应用被 Google Play 或 App Store 拒绝。

## AdDialog 组件结构

让我们看看示例项目中的 `AdDialog` 实现：

```dart
import 'countdown_timer.dart';
import 'package:flutter/material.dart';

/// 一个简单的类，在显示广告前显示提示对话框
class AdDialog extends StatefulWidget {
  final VoidCallback showAd;

  const AdDialog({super.key, required this.showAd});

  @override
  AdDialogState createState() => AdDialogState();
}
```

`AdDialog` 是一个 `StatefulWidget`，接受一个 `showAd` 回调函数，当用户确认观看广告时调用。

## 实现 AdDialogState

### 基本结构

```dart
class AdDialogState extends State<AdDialog> {
  final CountdownTimer _countdownTimer = CountdownTimer(5);

  @override
  void initState() {
    super.initState();
    // 设置倒计时监听器
    _countdownTimer.addListener(
      () => setState(() {
        if (_countdownTimer.isComplete) {
          Navigator.pop(context);
          widget.showAd();
        }
      }),
    );
    // 启动倒计时
    _countdownTimer.start();
  }

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text('Watch an ad for 10 more coins'),
      content: Text(
        'Video starting in ${_countdownTimer.timeLeft} seconds...',
        style: const TextStyle(color: Colors.grey),
      ),
      actions: <Widget>[
        TextButton(
          onPressed: () {
            Navigator.pop(context);
          },
          child: const Text('No thanks', style: TextStyle(color: Colors.red)),
        ),
      ],
    );
  }

  @override
  void dispose() {
    _countdownTimer.dispose();
    super.dispose();
  }
}
```

## 关键功能详解

### 1. 倒计时功能

`AdDialog` 使用 `CountdownTimer` 来实现倒计时功能。倒计时从 5 秒开始，当倒计时结束时，自动关闭对话框并显示广告。

```dart
final CountdownTimer _countdownTimer = CountdownTimer(5);
```

在 `initState()` 中，我们设置监听器来监听倒计时状态：

```dart
_countdownTimer.addListener(
  () => setState(() {
    if (_countdownTimer.isComplete) {
      Navigator.pop(context);
      widget.showAd();
    }
  }),
);
_countdownTimer.start();
```

### 2. 跳过选项

用户可以通过点击"No thanks"按钮来跳过广告：

```dart
TextButton(
  onPressed: () {
    Navigator.pop(context);
  },
  child: const Text('No thanks', style: TextStyle(color: Colors.red)),
),
```

**重要**：提供跳过选项是 Google 政策的强制要求。用户必须能够选择不观看广告。

### 3. 奖励信息展示

在对话框标题中，我们明确告知用户观看广告后能获得的奖励：

```dart
title: const Text('Watch an ad for 10 more coins'),
```

这帮助用户做出明智的决定，提高用户参与度。

### 4. 倒计时显示

在对话框内容中，我们显示倒计时，让用户知道广告即将开始：

```dart
content: Text(
  'Video starting in ${_countdownTimer.timeLeft} seconds...',
  style: const TextStyle(color: Colors.grey),
),
```

## 显示 AdDialog

在需要显示广告时，我们使用 `showDialog` 来显示 `AdDialog`：

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

`showAd` 回调在以下情况下被调用：

1. 倒计时结束时（自动）
2. 用户点击确认按钮时（如果有的话）

## 完整的 AdDialog 实现

以下是完整的 `AdDialog` 实现，包含所有必要的功能：

```dart
import 'countdown_timer.dart';
import 'package:flutter/material.dart';

/// 一个简单的类，在显示广告前显示提示对话框
class AdDialog extends StatefulWidget {
  final VoidCallback showAd;

  const AdDialog({super.key, required this.showAd});

  @override
  AdDialogState createState() => AdDialogState();
}

class AdDialogState extends State<AdDialog> {
  final CountdownTimer _countdownTimer = CountdownTimer(5);

  @override
  void initState() {
    _countdownTimer.addListener(
      () => setState(() {
        if (_countdownTimer.isComplete) {
          Navigator.pop(context);
          widget.showAd();
        }
      }),
    );
    _countdownTimer.start();

    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text('Watch an ad for 10 more coins'),
      content: Text(
        'Video starting in ${_countdownTimer.timeLeft} seconds...',
        style: const TextStyle(color: Colors.grey),
      ),
      actions: <Widget>[
        TextButton(
          onPressed: () {
            Navigator.pop(context);
          },
          child: const Text('No thanks', style: TextStyle(color: Colors.red)),
        ),
      ],
    );
  }

  @override
  void dispose() {
    _countdownTimer.dispose();
    super.dispose();
  }
}
```

## 用户体验最佳实践

### 1. 清晰的奖励说明

确保用户清楚地知道观看广告后能获得什么奖励。使用具体的数字和描述：

- ✅ "Watch an ad for 10 more coins"
- ❌ "Watch an ad for rewards"

### 2. 合理的倒计时时间

倒计时时间应该足够长，让用户有时间阅读和理解，但也不要太长，以免用户失去兴趣。建议 3-5 秒。

### 3. 明显的跳过选项

跳过按钮应该清晰可见，使用醒目的颜色（如红色）来区分：

```dart
child: const Text('No thanks', style: TextStyle(color: Colors.red)),
```

### 4. 友好的文案

使用友好、非强制性的语言，让用户感觉有选择权：

- ✅ "Watch an ad for 10 more coins"
- ❌ "You must watch this ad"

## 自定义 AdDialog

你可以根据应用的设计风格自定义 `AdDialog`：

```dart
@override
Widget build(BuildContext context) {
  return AlertDialog(
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(16),
    ),
    title: Row(
      children: [
        Icon(Icons.monetization_on, color: Colors.amber),
        const SizedBox(width: 8),
        const Text('Earn Rewards'),
      ],
    ),
    content: Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        const Text('Watch an ad to earn 10 coins!'),
        const SizedBox(height: 16),
        Text(
          'Video starting in ${_countdownTimer.timeLeft} seconds...',
          style: const TextStyle(
            color: Colors.grey,
            fontSize: 14,
          ),
        ),
      ],
    ),
    actions: <Widget>[
      TextButton(
        onPressed: () {
          Navigator.pop(context);
        },
        child: const Text('No thanks'),
      ),
      ElevatedButton(
        onPressed: () {
          Navigator.pop(context);
          widget.showAd();
        },
        child: const Text('Watch Ad'),
      ),
    ],
  );
}
```

## 实践练习

1. 实现基本的 `AdDialog` 组件
2. 添加倒计时功能
3. 实现跳过选项
4. 自定义对话框样式（可选）

## 常见问题

### Q: 倒计时是必须的吗？

A: 倒计时不是强制要求，但强烈推荐。它让用户知道广告即将开始，提供更好的用户体验。

### Q: 如果用户点击跳过，我应该做什么？

A: 如果用户选择跳过，只需关闭对话框即可。不要强制用户观看广告。

### Q: 我可以在介绍屏幕中添加更多信息吗？

A: 可以，但确保核心信息（奖励、跳过选项）清晰可见。不要添加过多信息，以免分散用户注意力。

## 总结与检查清单

### 本章要点

- 介绍屏幕是 Rewarded Interstitial Ads 的强制要求
- 必须提供跳过选项
- 倒计时功能可以改善用户体验
- 清晰的奖励说明可以提高用户参与度

### 检查清单

在继续下一章之前，确保你理解：

- [ ] Google 政策对介绍屏幕的要求
- [ ] `AdDialog` 组件的基本结构
- [ ] 如何实现倒计时功能
- [ ] 如何提供跳过选项
- [ ] 如何显示和关闭对话框

下一章，我们将学习如何显示广告和处理用户获得的奖励。
