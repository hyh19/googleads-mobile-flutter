# 第 6 章 显示原生模板广告

## 引言

加载原生模板广告后，我们需要在 Flutter UI 中显示它。本章将详细讲解如何使用 `AdWidget` 显示原生模板广告，包括设置容器尺寸、使用响应式布局、优化显示效果等。

## 使用 AdWidget 显示广告

### 基本显示

使用 `AdWidget` 来显示原生模板广告：

```dart
if (_nativeAdIsLoaded && _nativeAd != null)
  SizedBox(
    height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
    width: MediaQuery.of(context).size.width,
    child: AdWidget(ad: _nativeAd!),
  ),
```

### AdWidget 概述

`AdWidget` 是 Flutter 中用于显示广告的组件，它可以显示：

- 横幅广告（BannerAd）
- 原生广告（NativeAd）
- 其他广告格式

对于原生模板广告，只需要传入 `NativeAd` 实例即可。

## 设置广告容器尺寸

### 使用宽高比

根据模板类型设置正确的容器尺寸：

```dart
// Medium 模板的宽高比
final double _adAspectRatioMedium = 370 / 355;

// Small 模板的宽高比
final double _adAspectRatioSmall = 91 / 355;

// 设置容器尺寸
SizedBox(
  height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
  width: MediaQuery.of(context).size.width,
  child: AdWidget(ad: _nativeAd!),
)
```

### 响应式设计

使用 `MediaQuery` 获取屏幕宽度，确保广告在不同设备上正确显示：

```dart
SizedBox(
  height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
  width: MediaQuery.of(context).size.width,
  child: AdWidget(ad: _nativeAd!),
)
```

## 使用 Stack 布局优化显示

### 基本 Stack 布局

使用 `Stack` 来更好地控制广告位置和布局：

```dart
Stack(
  children: [
    SizedBox(
      height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
      width: MediaQuery.of(context).size.width,
    ),
    if (_nativeAdIsLoaded && _nativeAd != null)
      SizedBox(
        height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
        width: MediaQuery.of(context).size.width,
        child: AdWidget(ad: _nativeAd!),
      ),
  ],
)
```

### 为什么使用 Stack？

使用 `Stack` 的好处：

1. **占位空间**：即使广告未加载，也保留空间
2. **平滑过渡**：广告加载后平滑显示
3. **布局稳定**：避免广告加载导致的布局跳动

## 完整的显示实现

以下是示例项目中的完整实现：

```dart
@override
Widget build(BuildContext context) {
  return MaterialApp(
    title: 'Native Example',
    home: Scaffold(
      appBar: AppBar(
        title: const Text('Native Example'),
        actions: _appBarActions(),
      ),
      body: SizedBox(
        height: MediaQuery.of(context).size.height,
        width: MediaQuery.of(context).size.width,
        child: Column(
          children: [
            Stack(
              children: [
                SizedBox(
                  height: MediaQuery.of(context).size.width *
                      _adAspectRatioMedium,
                  width: MediaQuery.of(context).size.width,
                ),
                if (_nativeAdIsLoaded && _nativeAd != null)
                  SizedBox(
                    height: MediaQuery.of(context).size.width *
                        _adAspectRatioMedium,
                    width: MediaQuery.of(context).size.width,
                    child: AdWidget(ad: _nativeAd!),
                  ),
              ],
            ),
            TextButton(
              onPressed: _loadAd,
              child: const Text("Refresh Ad"),
            ),
            // ... 其他内容
          ],
        ),
      ),
    ),
  );
}
```

## 条件渲染

### 检查广告状态

在显示广告之前，检查广告是否已加载：

```dart
if (_nativeAdIsLoaded && _nativeAd != null)
  AdWidget(ad: _nativeAd!),
```

### 加载状态处理

在广告加载过程中，可以显示加载指示器或占位符：

```dart
if (_nativeAdIsLoaded && _nativeAd != null)
  AdWidget(ad: _nativeAd!)
else
  Container(
    height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
    width: MediaQuery.of(context).size.width,
    color: Colors.grey[200],
    child: Center(
      child: CircularProgressIndicator(),
    ),
  ),
```

## 响应式设计考虑

### 适配不同屏幕尺寸

广告会自动适配不同屏幕尺寸，但你需要：

1. **使用宽高比**：根据模板类型使用正确的宽高比
2. **使用 MediaQuery**：获取屏幕宽度计算高度
3. **测试不同设备**：在多种设备上测试显示效果

### 横屏和竖屏

考虑横屏和竖屏的显示：

```dart
// 根据屏幕方向调整
final bool isLandscape = 
    MediaQuery.of(context).orientation == Orientation.landscape;

final double adHeight = isLandscape
    ? MediaQuery.of(context).size.height * 0.3
    : MediaQuery.of(context).size.width * _adAspectRatioMedium;
```

## 布局最佳实践

### 1. 预留空间

即使广告未加载，也预留空间，避免布局跳动：

```dart
Stack(
  children: [
    // 占位空间
    SizedBox(
      height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
      width: MediaQuery.of(context).size.width,
    ),
    // 广告（如果已加载）
    if (_nativeAdIsLoaded && _nativeAd != null)
      AdWidget(ad: _nativeAd!),
  ],
)
```

### 2. 居中显示

如果需要居中显示广告：

```dart
Center(
  child: SizedBox(
    height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
    width: MediaQuery.of(context).size.width,
    child: AdWidget(ad: _nativeAd!),
  ),
)
```

### 3. 添加间距

在广告周围添加适当的间距：

```dart
Padding(
  padding: const EdgeInsets.all(16.0),
  child: SizedBox(
    height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
    width: MediaQuery.of(context).size.width,
    child: AdWidget(ad: _nativeAd!),
  ),
)
```

## 实践练习

1. 使用 `AdWidget` 显示原生模板广告
2. 根据模板类型设置正确的容器尺寸
3. 使用 `Stack` 布局优化显示
4. 实现条件渲染和加载状态处理
5. 测试在不同设备上的显示效果

## 常见问题

### Q: 广告显示为空白怎么办？

A: 检查：

1. 广告是否已加载（`_nativeAdIsLoaded` 是否为 `true`）
2. 容器尺寸是否正确
3. `_nativeAd` 是否为 `null`

### Q: 广告尺寸不正确怎么办？

A: 确保使用正确的宽高比：

- Small 模板：91/355
- Medium 模板：370/355

### Q: 可以自定义广告尺寸吗？

A: 不可以。原生模板广告使用预定义模板，尺寸由模板类型决定。你只能设置容器尺寸。

### Q: 广告在横屏时显示异常怎么办？

A: 考虑根据屏幕方向调整容器尺寸，或使用固定宽度。

## 总结与检查清单

### 本章要点

- 使用 `AdWidget` 显示原生模板广告
- 根据模板类型设置正确的容器尺寸（使用宽高比）
- 使用 `Stack` 布局优化显示效果
- 实现条件渲染和加载状态处理

### 检查清单

在继续下一章之前，确保你理解：

- [ ] 如何使用 `AdWidget` 显示广告
- [ ] 如何设置正确的容器尺寸
- [ ] 如何使用 `Stack` 布局
- [ ] 如何实现条件渲染

下一章，我们将学习如何处理广告事件。
