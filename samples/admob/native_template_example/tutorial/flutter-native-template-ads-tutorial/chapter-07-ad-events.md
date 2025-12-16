# 第 7 章 处理广告事件

## 引言

原生模板广告在生命周期中会触发各种事件，如加载成功、加载失败、用户点击、广告展示等。通过 `NativeAdListener`，我们可以监听和处理这些事件，实现更好的用户体验和广告效果追踪。本章将详细讲解所有可用的事件回调及其用途。

## NativeAdListener 概述

### 基本概念

`NativeAdListener` 是用于监听原生广告事件的回调接口，提供了多个回调方法来处理不同的广告事件。

### 使用方式

在创建 `NativeAd` 时设置 `listener`：

```dart
_nativeAd = NativeAd(
  adUnitId: _adUnitId,
  listener: NativeAdListener(
    // 各种回调
  ),
  // ... 其他配置
)..load();
```

## 所有可用回调

### onAdLoaded：广告加载成功

当广告成功加载时触发：

```dart
onAdLoaded: (ad) {
  print('NativeAd loaded.');
  setState(() {
    _nativeAdIsLoaded = true;
  });
},
```

**用途**：

- 更新 UI 状态，显示广告
- 记录加载成功事件
- 开始追踪广告展示

### onAdFailedToLoad：广告加载失败

当广告加载失败时触发：

```dart
onAdFailedToLoad: (ad, error) {
  print('NativeAd failedToLoad: $error');
  print('Error code: ${error.code}');
  print('Error domain: ${error.domain}');
  print('Error message: ${error.message}');
  
  ad.dispose();
  setState(() {
    _nativeAdIsLoaded = false;
    _nativeAd = null;
  });
},
```

**用途**：

- 处理加载失败
- 记录错误信息
- 清理资源
- 实现重试逻辑

**错误信息**：

- `error.code`：错误代码
- `error.domain`：错误域
- `error.message`：错误消息

### onAdClicked：用户点击广告

当用户点击广告时触发：

```dart
onAdClicked: (ad) {
  print('NativeAd clicked.');
  // 可以在这里记录点击事件
},
```

**用途**：

- 追踪用户点击
- 分析广告效果
- 记录用户行为

### onAdImpression：广告展示

当广告产生展示（impression）时触发：

```dart
onAdImpression: (ad) {
  print('NativeAd impression.');
  // 可以在这里记录展示事件
},
```

**用途**：

- 追踪广告展示
- 计算展示次数
- 分析广告效果

**重要**：展示事件是计算广告收益的重要指标。

### onAdOpened：广告打开覆盖层

当广告打开覆盖层（如全屏视图）时触发：

```dart
onAdOpened: (ad) {
  print('NativeAd opened.');
  // 可以在这里暂停游戏、视频等
},
```

**用途**：

- 暂停应用活动（如游戏、视频播放）
- 记录用户交互
- 处理应用状态

### onAdClosed：广告关闭覆盖层

当广告关闭覆盖层时触发：

```dart
onAdClosed: (ad) {
  print('NativeAd closed.');
  // 可以在这里恢复游戏、视频等
},
```

**用途**：

- 恢复应用活动
- 记录用户返回
- 处理应用状态

### onAdWillDismissScreen：iOS 专用

在 iOS 上，当即将关闭全屏视图时触发：

```dart
onAdWillDismissScreen: (ad) {
  print('NativeAd will dismiss screen.');
  // iOS 专用回调
},
```

**用途**：

- iOS 平台特定的处理
- 在关闭前执行清理操作

### onPaidEvent：广告收益事件

当广告产生收益时触发：

```dart
onPaidEvent: (ad, valueMicros, precision, currencyCode) {
  print('NativeAd paid event:');
  print('  Value: $valueMicros micros');
  print('  Precision: $precision');
  print('  Currency: $currencyCode');
  
  // 可以在这里记录收益数据
},
```

**参数说明**：

- `valueMicros`：收益值（微单位，需要除以 1,000,000 得到实际金额）
- `precision`：精度类型
- `currencyCode`：货币代码（如 "USD"、"CNY"）

**用途**：

- 追踪广告收益
- 分析收益数据
- 计算 eCPM（有效每千次展示成本）

## 完整的监听器实现

以下是示例项目中的完整实现：

```dart
listener: NativeAdListener(
  onAdLoaded: (ad) {
    print('$NativeAd loaded.');
    setState(() {
      _nativeAdIsLoaded = true;
    });
  },
  onAdFailedToLoad: (ad, error) {
    print('$NativeAd failedToLoad: $error');
    ad.dispose();
  },
  onAdClicked: (ad) {
    print('NativeAd clicked.');
  },
  onAdImpression: (ad) {
    print('NativeAd impression.');
  },
  onAdClosed: (ad) {
    print('NativeAd closed.');
  },
  onAdOpened: (ad) {
    print('NativeAd opened.');
  },
  onAdWillDismissScreen: (ad) {
    print('NativeAd will dismiss screen.');
  },
  onPaidEvent: (ad, valueMicros, precision, currencyCode) {
    print('NativeAd paid event: $valueMicros $currencyCode');
  },
),
```

## 错误处理和调试

### 详细错误日志

在 `onAdFailedToLoad` 中记录详细的错误信息：

```dart
onAdFailedToLoad: (ad, error) {
  debugPrint('NativeAd failedToLoad:');
  debugPrint('  Error code: ${error.code}');
  debugPrint('  Error domain: ${error.domain}');
  debugPrint('  Error message: ${error.message}');
  
  // 根据错误代码处理
  switch (error.code) {
    case 0:
      debugPrint('  Configuration error');
      break;
    case 3:
      debugPrint('  Network error');
      break;
    case 8:
      debugPrint('  Internal error');
      break;
    default:
      debugPrint('  Unknown error');
  }
  
  ad.dispose();
},
```

### 常见错误代码

- **0**：配置错误（如应用 ID 未设置）
- **1**：无效请求
- **2**：网络错误
- **3**：网络错误
- **8**：内部错误

## 事件追踪最佳实践

### 1. 记录关键事件

记录所有关键事件，用于分析：

```dart
onAdLoaded: (ad) {
  _analytics.logEvent('native_ad_loaded');
  setState(() {
    _nativeAdIsLoaded = true;
  });
},

onAdImpression: (ad) {
  _analytics.logEvent('native_ad_impression');
},

onAdClicked: (ad) {
  _analytics.logEvent('native_ad_clicked');
},
```

### 2. 计算关键指标

使用事件数据计算关键指标：

```dart
int _impressionCount = 0;
int _clickCount = 0;

onAdImpression: (ad) {
  _impressionCount++;
  final ctr = _clickCount / _impressionCount;
  print('CTR: ${ctr.toStringAsFixed(2)}%');
},

onAdClicked: (ad) {
  _clickCount++;
},
```

### 3. 处理应用状态

在广告打开和关闭时处理应用状态：

```dart
onAdOpened: (ad) {
  // 暂停游戏、视频等
  _gameController.pause();
  _videoPlayer.pause();
},

onAdClosed: (ad) {
  // 恢复游戏、视频等
  _gameController.resume();
  _videoPlayer.resume();
},
```

## 实践练习

1. 实现所有 `NativeAdListener` 回调
2. 添加详细的错误日志
3. 实现事件追踪
4. 处理应用状态（暂停/恢复）

## 常见问题

### Q: 哪些回调是必需的？

A: 所有回调都是可选的，但建议至少实现 `onAdLoaded` 和 `onAdFailedToLoad`。

### Q: `onAdImpression` 什么时候触发？

A: 当广告在屏幕上可见并满足展示条件时触发。具体时机由 SDK 决定。

### Q: 如何计算实际收益金额？

A: 将 `valueMicros` 除以 1,000,000 得到实际金额。例如，`valueMicros = 1000000` 表示 1.0 单位货币。

### Q: `onAdWillDismissScreen` 只在 iOS 上触发吗？

A: 是的，这是 iOS 平台专用的回调。

## 总结与检查清单

### 本章要点

- `NativeAdListener` 提供了多个回调来处理广告事件
- 关键回调：`onAdLoaded`、`onAdFailedToLoad`、`onAdImpression`、`onAdClicked`
- 使用回调追踪广告效果和处理应用状态
- 记录详细的错误信息以便调试

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `NativeAdListener` 的所有回调
- [ ] 如何处理加载成功和失败
- [ ] 如何追踪广告效果
- [ ] 如何处理应用状态

下一章，我们将学习用户同意管理（UMP）的集成。
