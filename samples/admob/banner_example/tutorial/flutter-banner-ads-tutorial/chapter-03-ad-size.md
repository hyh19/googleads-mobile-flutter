# 第 3 章：自适应横幅广告尺寸

## 章节简介

在本章中，我们将学习如何获取自适应横幅广告尺寸。自适应横幅广告是 Google Mobile Ads SDK 推荐使用的格式，它会根据设备的屏幕宽度自动调整广告尺寸，确保在不同设备上都能获得最佳的显示效果。我们将详细讲解 `AdSize` 类和 `getCurrentOrientationAnchoredAdaptiveBannerAdSize()` 方法的使用。

## 为什么需要自适应尺寸

### 设备多样性

移动设备有各种不同的屏幕尺寸：

- 小屏手机：宽度约 320-360 像素
- 中屏手机：宽度约 360-414 像素
- 大屏手机：宽度约 414-480 像素
- 平板：宽度可达 768 像素或更多

### 固定尺寸的问题

如果使用固定的广告尺寸（如 320x50），在不同设备上可能会遇到以下问题：

- **小屏设备**：广告可能过大，影响布局
- **大屏设备**：广告可能过小，浪费屏幕空间
- **横屏模式**：固定尺寸可能不适合横屏布局

### 自适应尺寸的优势

自适应横幅广告尺寸可以：

- **自动适配**：根据屏幕宽度自动调整
- **最佳效果**：在不同设备上都能获得最佳显示效果
- **简化开发**：无需手动计算不同设备的尺寸
- **提高填充率**：自适应尺寸通常有更高的广告填充率

## AdSize 类详解

### AdSize 类概述

`AdSize` 是 Google Mobile Ads SDK 提供的类，用于定义广告的尺寸。它支持多种尺寸类型：

- **固定尺寸**：如 `AdSize.banner` (320x50)
- **自适应尺寸**：根据屏幕宽度自动计算
- **自定义尺寸**：手动指定宽度和高度

### 固定尺寸常量

SDK 提供了一些常用的固定尺寸常量：

```dart
AdSize.banner          // 320x50
AdSize.largeBanner     // 320x100
AdSize.mediumRectangle // 300x250
AdSize.fullBanner      // 468x60
AdSize.leaderboard     // 728x90
```

**注意**：这些固定尺寸主要用于桌面网页，移动应用推荐使用自适应尺寸。

## 获取自适应横幅广告尺寸

### getCurrentOrientationAnchoredAdaptiveBannerAdSize() 方法

这是获取自适应横幅广告尺寸的主要方法：

```dart
final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
  MediaQuery.of(context).size.width.truncate(),
);
```

**方法参数**：

- `width`：屏幕宽度（像素），使用 `MediaQuery.of(context).size.width.truncate()` 获取

**返回值**：

- `AdSize?`：如果成功获取，返回 `AdSize` 对象；如果失败，返回 `null`

### 完整示例

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

class BannerExample extends StatefulWidget {
  @override
  State<BannerExample> createState() => _BannerExampleState();
}

class _BannerExampleState extends State<BannerExample> {
  AdSize? _adSize;

  @override
  void initState() {
    super.initState();
    _getAdSize();
  }

  Future<void> _getAdSize() async {
    // 获取当前屏幕宽度
    final width = MediaQuery.of(context).size.width.truncate();
    
    // 获取自适应广告尺寸
    final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
      width,
    );
    
    if (size != null) {
      setState(() {
        _adSize = size;
      });
      debugPrint('Ad size: ${size.width}x${size.height}');
    } else {
      debugPrint('Unable to get ad size');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Banner Example')),
      body: Center(
        child: _adSize != null
            ? Text('Ad size: ${_adSize!.width}x${_adSize!.height}')
            : CircularProgressIndicator(),
      ),
    );
  }
}
```

## 在加载广告时获取尺寸

在实际应用中，我们通常在加载广告时获取尺寸：

```dart
void _loadAd() async {
  // 获取自适应广告尺寸
  final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
    MediaQuery.sizeOf(context).width.truncate(),
  );

  if (size == null) {
    // 无法获取宽度，无法加载广告
    debugPrint('Unable to get width of anchored banner.');
    return;
  }

  // 使用获取的尺寸创建 BannerAd
  BannerAd(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    size: size, // 使用自适应尺寸
    listener: BannerAdListener(
      // ... 监听器实现
    ),
  ).load();
}
```

**代码说明**：

- `MediaQuery.sizeOf(context)`：获取当前上下文的大小信息
- `.width.truncate()`：获取宽度并截断为整数
- 检查 `size` 是否为 `null`，如果为 `null` 则无法加载广告

## 处理尺寸获取失败

### 检查返回值

`getCurrentOrientationAnchoredAdaptiveBannerAdSize()` 可能返回 `null`，因此需要检查：

```dart
final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
  MediaQuery.of(context).size.width.truncate(),
);

if (size == null) {
  debugPrint('Unable to get ad size');
  // 可以选择使用固定尺寸作为后备方案
  // 或者不加载广告
  return;
}
```

### 后备方案

如果无法获取自适应尺寸，可以使用固定尺寸作为后备：

```dart
AdSize getAdSize() async {
  final adaptiveSize = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
    MediaQuery.of(context).size.width.truncate(),
  );
  
  // 如果无法获取自适应尺寸，使用固定尺寸
  return adaptiveSize ?? AdSize.banner;
}
```

## 屏幕方向考虑

### 当前方向

`getCurrentOrientationAnchoredAdaptiveBannerAdSize()` 方法会根据**当前屏幕方向**自动计算合适的尺寸：

- **竖屏模式**：通常返回较窄的广告尺寸
- **横屏模式**：通常返回较宽的广告尺寸

### 方向变化处理

当屏幕方向改变时，需要重新获取广告尺寸。我们将在第 6 章详细讲解如何处理屏幕方向变化。

## 常见尺寸示例

不同设备上可能获得的广告尺寸示例：

| 设备类型 | 屏幕宽度 | 广告尺寸（示例） |
|---------|---------|----------------|
| 小屏手机 | 320px | 320x50 |
| 中屏手机 | 360px | 360x50 |
| 大屏手机 | 414px | 414x50 |
| 平板（竖屏） | 768px | 728x90 |
| 平板（横屏） | 1024px | 1024x90 |

**注意**：实际尺寸可能因设备而异，这些只是示例。

## 实践练习

完成以下练习以巩固本章内容：

1. **获取广告尺寸**：创建一个简单的应用，获取并显示自适应广告尺寸
2. **测试不同设备**：在不同尺寸的设备或模拟器上测试，观察尺寸变化
3. **处理空值**：实现检查 `null` 的逻辑
4. **实现后备方案**：如果无法获取自适应尺寸，使用固定尺寸作为后备

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解为什么需要自适应广告尺寸
- [ ] 了解 `AdSize` 类的作用
- [ ] 使用 `getCurrentOrientationAnchoredAdaptiveBannerAdSize()` 获取自适应尺寸
- [ ] 理解方法参数和返回值
- [ ] 处理尺寸获取失败的情况
- [ ] 在加载广告时正确获取尺寸

## 下一步

现在我们已经学会了如何获取自适应广告尺寸，在下一章中，我们将学习如何使用这个尺寸来加载 Banner 广告。

继续学习：[第 4 章：加载横幅广告](chapter-04-loading-ads.md)
