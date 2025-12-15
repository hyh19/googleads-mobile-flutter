# 第 6 章：处理屏幕方向变化

## 章节简介

在本章中，我们将学习如何处理屏幕方向变化。当设备从竖屏旋转到横屏（或相反）时，屏幕宽度会发生变化，需要重新获取广告尺寸并重新加载广告。我们将详细讲解如何使用 `OrientationBuilder` 检测方向变化，以及如何正确清理旧广告并加载新广告。

## 为什么需要处理方向变化

### 屏幕方向变化的影响

当设备旋转时：

- **屏幕宽度变化**：竖屏和横屏的宽度不同
- **广告尺寸变化**：需要根据新的宽度重新计算广告尺寸
- **布局变化**：应用布局可能需要调整

### 不处理方向变化的问题

如果不处理方向变化：

- 广告尺寸可能不适合新的屏幕方向
- 广告可能显示不正确
- 用户体验差

## OrientationBuilder 简介

### OrientationBuilder 类

`OrientationBuilder` 是 Flutter 提供的 Widget，用于根据屏幕方向构建不同的 UI。它会在方向变化时自动重建。

### 基本用法

```dart
OrientationBuilder(
  builder: (context, orientation) {
    if (orientation == Orientation.portrait) {
      // 竖屏布局
    } else {
      // 横屏布局
    }
    return Widget();
  },
)
```

**参数**：

- `builder`：构建函数，接收 `BuildContext` 和 `Orientation` 参数

## 检测方向变化

### 跟踪当前方向

我们需要跟踪当前的方向，以便检测变化：

```dart
class _BannerExampleState extends State<BannerExample> {
  BannerAd? _bannerAd;
  Orientation? _currentOrientation;

  @override
  Widget build(BuildContext context) {
    return OrientationBuilder(
      builder: (context, orientation) {
        // 检测方向变化
        if (_currentOrientation != orientation) {
          if (_currentOrientation != null) {
            // 方向已改变，需要重新加载广告
            _bannerAd?.dispose();
            _bannerAd = null;
            _loadAd();
          }
          _currentOrientation = orientation;
        }
        
        return Scaffold(
          // ... UI 内容
        );
      },
    );
  }
}
```

### 代码详解

让我们逐行分析这个实现：

#### 1. 跟踪方向

```dart
Orientation? _currentOrientation;
```

- 使用可空变量跟踪当前方向
- 初始值为 `null`（首次构建时）

#### 2. OrientationBuilder

```dart
OrientationBuilder(
  builder: (context, orientation) {
    // 构建逻辑
  },
)
```

- `OrientationBuilder` 会在方向变化时重建
- `orientation` 参数表示当前方向

#### 3. 检测变化

```dart
if (_currentOrientation != orientation) {
  // 方向已改变
}
```

- 比较当前方向和之前的方向
- 如果不相同，说明方向已改变

#### 4. 处理首次构建

```dart
if (_currentOrientation != null) {
  // 不是首次构建，需要重新加载
}
```

- 如果 `_currentOrientation` 为 `null`，说明是首次构建
- 只有在方向真正改变时才重新加载

#### 5. 清理旧广告

```dart
_bannerAd?.dispose();
_bannerAd = null;
```

- 释放旧的广告对象
- 清空引用

#### 6. 重新加载

```dart
_loadAd();
```

- 调用加载方法重新加载广告
- 新广告会根据新的屏幕宽度自动调整尺寸

#### 7. 更新方向

```dart
_currentOrientation = orientation;
```

- 更新当前方向
- 避免重复触发

## 完整的实现

以下是完整的处理方向变化的实现：

```dart
class _BannerExampleState extends State<BannerExample> {
  BannerAd? _bannerAd;
  Orientation? _currentOrientation;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Banner Example'),
        ),
        body: OrientationBuilder(
          builder: (context, orientation) {
            if (_currentOrientation != orientation) {
              if (_currentOrientation != null) {
                // 方向已改变，释放旧广告并加载新广告
                _bannerAd?.dispose();
                _bannerAd = null;
                _loadAd();
              }
              _currentOrientation = orientation;
            }
            
            return Stack(
              children: [
                // 应用内容
                Center(
                  child: Text('Your app content'),
                ),
                
                // 广告
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
            );
          },
        ),
      ),
    );
  }

  void _loadAd() async {
    // 获取自适应广告尺寸（会根据当前方向自动调整）
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
  void dispose() {
    _bannerAd?.dispose();
    super.dispose();
  }
}
```

## 资源清理

### 为什么需要清理

当方向改变时，旧的广告对象：

- 尺寸不适合新的方向
- 占用内存资源
- 可能导致显示问题

### 清理时机

应该在以下时机清理广告：

1. **方向改变时**：释放旧广告，加载新广告
2. **Widget 销毁时**：在 `dispose()` 中清理

### dispose() 方法

```dart
@override
void dispose() {
  _bannerAd?.dispose();
  super.dispose();
}
```

**代码说明**：

- `_bannerAd?.dispose()`：如果广告不为 `null`，调用 `dispose()` 释放资源
- 必须在 `super.dispose()` 之前调用

## 自适应尺寸的方向感知

### getCurrentOrientationAnchoredAdaptiveBannerAdSize()

这个方法会根据**当前屏幕方向**自动计算合适的广告尺寸：

- **竖屏模式**：根据竖屏宽度计算
- **横屏模式**：根据横屏宽度计算

因此，当方向改变时，只需要重新调用这个方法，就会自动获得适合新方向的尺寸。

## 测试方向变化

### 测试步骤

1. **运行应用**：启动应用，广告应该正常显示
2. **旋转设备**：将设备从竖屏旋转到横屏
3. **观察广告**：广告应该重新加载并适应新方向
4. **再次旋转**：旋转回竖屏，广告应该再次适应

### 检查日志

在控制台中，你应该看到：

```text
Ad was loaded.  // 初始加载
Ad was loaded.  // 方向改变后重新加载
```

## 常见问题

### 方向改变时广告不更新

**可能原因**：

1. 没有使用 `OrientationBuilder`
2. 没有检测方向变化
3. 没有重新加载广告

**解决方案**：确保按照本章的示例实现方向检测和重新加载逻辑。

### 广告闪烁

**可能原因**：在重新加载时，旧广告被移除，新广告尚未加载完成。

**解决方案**：这是正常现象。可以考虑在方向改变时短暂隐藏广告，等新广告加载完成后再显示。

### 性能问题

**可能原因**：频繁的方向变化导致频繁重新加载广告。

**解决方案**：确保只在方向真正改变时才重新加载，避免不必要的操作。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现方向检测**：使用 `OrientationBuilder` 检测方向变化
2. **实现重新加载**：在方向改变时重新加载广告
3. **测试方向变化**：旋转设备测试广告是否正确更新
4. **优化性能**：确保只在必要时重新加载

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解为什么需要处理方向变化
- [ ] 使用 `OrientationBuilder` 检测方向变化
- [ ] 跟踪当前方向状态
- [ ] 在方向改变时清理旧广告
- [ ] 在方向改变时重新加载广告
- [ ] 正确实现 `dispose()` 方法
- [ ] 理解自适应尺寸的方向感知

## 下一步

现在我们已经实现了方向变化的处理。在下一章中，我们将学习如何监听和处理各种广告事件，如广告点击、展示等。

继续学习：[第 7 章：广告事件监听](chapter-07-ad-events.md)
