# 第 8 章 性能与体验优化

本章讨论 Banner 的刷新与重建策略、资源释放以及如何减少界面闪烁，提升用户体验。

## 简介

避免过度重建和泄漏，确保在方向变化或页面退出时正确释放广告资源。自适应 Banner 可减少布局抖动。

## 核心步骤

1. 在方向变化时先 dispose 再重新加载，避免尺寸不匹配。
2. 在 `dispose` 生命周期中释放 `_bannerAd`。
3. 仅在同意允许后加载，减少无效请求。
4. 可为空状态预留占位高度，减少跳动。

## 代码示例

```dart
if (_currentOrientation != orientation) {
  if (_currentOrientation != null) {
    _bannerAd?.dispose();
    _bannerAd = null;
    _loadAd();
  }
  _currentOrientation = orientation;
}

@override
void dispose() {
  _bannerAd?.dispose();
  super.dispose();
}
```

## 练习与检查

- 在横竖屏切换时观察是否存在闪烁，可添加占位 `SizedBox` 平滑过渡。
- 压测频繁导航返回时，确认无资源泄漏警告。
- 检查加载失败时的 UI，确保不会挡住主要内容。

## 小结

通过合理的生命周期管理与占位策略，你可以在不同设备上提供稳定的 Banner 体验。
