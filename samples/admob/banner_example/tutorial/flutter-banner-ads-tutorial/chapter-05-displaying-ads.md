# 第 5 章：显示横幅广告

## 章节简介

在本章中，我们将学习如何显示已加载的 Banner 广告。我们将详细讲解如何使用 `AdWidget` 显示广告、在 `Stack` 中定位广告、使用 `SafeArea` 确保安全显示，以及如何正确设置广告的尺寸和布局。

## AdWidget 简介

### AdWidget 类

`AdWidget` 是 Google Mobile Ads SDK 提供的 Widget，用于在 Flutter 应用中显示广告。它将 `Ad` 对象包装成一个 Flutter Widget，可以直接在 Widget 树中使用。

### 基本用法

```dart
AdWidget(ad: _bannerAd!)
```

**参数**：

- `ad`：要显示的 `Ad` 对象（必须是 `BannerAd` 类型）

## 在 Stack 中显示广告

### 为什么使用 Stack

`Stack` 允许我们将广告叠加在其他内容之上，这是显示 Banner 广告的常用方式。通常将广告放在底部或顶部。

### 基础实现

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: Stack(
      children: [
        // 应用的主要内容
        Center(
          child: Text('Your app content here'),
        ),
        
        // 广告显示在底部
        if (_bannerAd != null)
          Align(
            alignment: Alignment.bottomCenter,
            child: SafeArea(
              child: SizedBox(
                width: _bannerAd!.size.width.toDouble(),
                height: _bannerAd!.size.height.toDouble(),
                child: AdWidget(ad: _bannerAd!),
              ),
            ),
          ),
      ],
    ),
  );
}
```

### 代码详解

让我们逐行分析这个实现：

#### 1. Stack 布局

```dart
Stack(
  children: [
    // 子组件
  ],
)
```

- `Stack` 允许子组件重叠
- 后面的子组件会显示在前面

#### 2. 主要内容

```dart
Center(
  child: Text('Your app content here'),
),
```

- 这是应用的主要内容
- 广告会叠加在这个内容之上

#### 3. 条件显示广告

```dart
if (_bannerAd != null)
  // 广告 Widget
```

- 只有当广告已加载（`_bannerAd` 不为 `null`）时才显示
- 使用 `if` 条件表达式

#### 4. Align 定位

```dart
Align(
  alignment: Alignment.bottomCenter,
  child: // 广告内容
)
```

- `Align` 用于定位子组件
- `Alignment.bottomCenter` 将广告放在底部中央

#### 5. SafeArea 保护

```dart
SafeArea(
  child: // 广告内容
)
```

- `SafeArea` 确保内容不会被系统 UI（如状态栏、导航栏）遮挡
- 这对于广告显示非常重要

#### 6. SizedBox 设置尺寸

```dart
SizedBox(
  width: _bannerAd!.size.width.toDouble(),
  height: _bannerAd!.size.height.toDouble(),
  child: AdWidget(ad: _bannerAd!),
)
```

**代码说明**：

- `width`：使用广告的实际宽度
- `height`：使用广告的实际高度
- `.toDouble()`：将整数转换为浮点数（`SizedBox` 需要 `double` 类型）
- `AdWidget`：显示广告的 Widget

## 完整的显示实现

以下是完整的显示广告的实现：

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: const Text('Banner Example'),
    ),
    body: Stack(
      children: [
        // 应用的主要内容
        Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('Your app content here'),
              // 其他内容...
            ],
          ),
        ),
        
        // 广告显示在底部
        if (_bannerAd != null)
          Align(
            alignment: Alignment.bottomCenter,
            child: SafeArea(
              child: SizedBox(
                width: _bannerAd!.size.width.toDouble(),
                height: _bannerAd!.size.height.toDouble(),
                child: AdWidget(ad: _bannerAd!),
              ),
            ),
          ),
      ],
    ),
  );
}
```

## 广告位置选择

### 底部显示（推荐）

底部是最常见的 Banner 广告位置：

```dart
Align(
  alignment: Alignment.bottomCenter,
  child: SafeArea(
    child: SizedBox(
      width: _bannerAd!.size.width.toDouble(),
      height: _bannerAd!.size.height.toDouble(),
      child: AdWidget(ad: _bannerAd!),
    ),
  ),
)
```

**优势**：

- 不遮挡主要内容
- 用户习惯底部广告位置
- 不影响用户操作

### 顶部显示

也可以将广告放在顶部：

```dart
Align(
  alignment: Alignment.topCenter,
  child: SafeArea(
    child: SizedBox(
      width: _bannerAd!.size.width.toDouble(),
      height: _bannerAd!.size.height.toDouble(),
      child: AdWidget(ad: _bannerAd!),
    ),
  ),
)
```

**注意**：顶部广告可能会遮挡应用内容，需要确保有足够的空间。

## SafeArea 的重要性

### 为什么需要 SafeArea

移动设备的屏幕可能被系统 UI 占用：

- **状态栏**：顶部显示时间、电池等信息
- **导航栏**：底部显示导航按钮（某些 Android 设备）
- **刘海屏**：某些设备有刘海或挖孔

### SafeArea 的作用

`SafeArea` 会自动调整内容位置，确保不被系统 UI 遮挡：

```dart
SafeArea(
  child: AdWidget(ad: _bannerAd!),
)
```

**效果**：

- 在底部时，自动避开导航栏
- 在顶部时，自动避开状态栏
- 在有刘海的设备上，自动避开刘海区域

## 广告尺寸设置

### 使用实际尺寸

必须使用广告的实际尺寸来设置 `SizedBox`：

```dart
SizedBox(
  width: _bannerAd!.size.width.toDouble(),
  height: _bannerAd!.size.height.toDouble(),
  child: AdWidget(ad: _bannerAd!),
)
```

**为什么重要**：

- 尺寸不匹配可能导致广告显示异常
- 正确的尺寸确保广告正常渲染
- 自适应尺寸需要动态获取

### 尺寸类型转换

注意 `AdSize` 的 `width` 和 `height` 是 `int` 类型，而 `SizedBox` 需要 `double` 类型：

```dart
width: _bannerAd!.size.width.toDouble(),
height: _bannerAd!.size.height.toDouble(),
```

## 完整示例

以下是完整的示例，展示如何加载和显示广告：

```dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

class BannerExample extends StatefulWidget {
  const BannerExample({super.key});

  @override
  State<BannerExample> createState() => _BannerExampleState();
}

class _BannerExampleState extends State<BannerExample> {
  BannerAd? _bannerAd;
  
  final String _adUnitId = Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/9214589741'
      : 'ca-app-pub-3940256099942544/2435281174';

  @override
  void initState() {
    super.initState();
    _loadAd();
  }

  void _loadAd() async {
    final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
      MediaQuery.sizeOf(context).width.truncate(),
    );

    if (size == null) {
      return;
    }

    BannerAd(
      adUnitId: _adUnitId,
      request: const AdRequest(),
      size: size,
      listener: BannerAdListener(
        onAdLoaded: (ad) {
          setState(() {
            _bannerAd = ad as BannerAd;
          });
        },
        onAdFailedToLoad: (ad, err) {
          ad.dispose();
        },
      ),
    ).load();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Banner Example'),
      ),
      body: Stack(
        children: [
          Center(
            child: Text('Your app content'),
          ),
          if (_bannerAd != null)
            Align(
              alignment: Alignment.bottomCenter,
              child: SafeArea(
                child: SizedBox(
                  width: _bannerAd!.size.width.toDouble(),
                  height: _bannerAd!.size.height.toDouble(),
                  child: AdWidget(ad: _bannerAd!),
                ),
              ),
            ),
        ],
      ),
    );
  }

  @override
  void dispose() {
    _bannerAd?.dispose();
    super.dispose();
  }
}
```

## 常见问题

### 广告不显示

可能的原因：

1. **广告未加载**：检查 `_bannerAd` 是否为 `null`
2. **尺寸错误**：确保使用正确的广告尺寸
3. **布局问题**：检查 `Stack` 和 `Align` 的配置

### 广告被遮挡

**解决方案**：使用 `SafeArea` 确保广告不被系统 UI 遮挡。

### 广告尺寸不正确

**解决方案**：确保使用 `_bannerAd!.size.width` 和 `_bannerAd!.size.height` 设置尺寸。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现广告显示**：在 `build` 方法中实现广告显示逻辑
2. **测试不同位置**：尝试将广告放在顶部和底部
3. **测试 SafeArea**：移除 `SafeArea` 观察效果
4. **测试尺寸设置**：使用错误的尺寸观察效果

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解 `AdWidget` 的作用
- [ ] 在 `Stack` 中显示广告
- [ ] 使用 `Align` 定位广告
- [ ] 使用 `SafeArea` 确保安全显示
- [ ] 正确设置广告尺寸
- [ ] 理解广告位置的选择

## 下一步

现在我们已经实现了广告的加载和显示功能。在下一章中，我们将学习如何处理屏幕方向变化，确保在设备旋转时广告能够正确显示。

继续学习：[第 6 章：处理屏幕方向变化](chapter-06-orientation-handling.md)
