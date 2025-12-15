# 第 5 章 UI 结构与 AppBar 导航

本章解释示例的 UI 组成、AppBar 菜单以及如何在界面中容纳 Banner 广告。

## 简介

UI 由 `MaterialApp + Scaffold` 构成，主体使用 `OrientationBuilder` 监听方向变化，底部通过 `Align + SafeArea` 展示 `AdWidget`。

## 核心步骤

1. 在 `build` 中创建 `MaterialApp`，`Scaffold` 提供 `AppBar` 与主体。
2. `OrientationBuilder` 监听横竖屏切换，必要时重新加载 Banner。
3. `_appBarActions` 根据是否需要隐私入口动态生成菜单项。

## 代码示例

```dart
Widget build(BuildContext context) {
  return MaterialApp(
    title: 'Banner Example',
    home: Scaffold(
      appBar: AppBar(
        title: const Text('Banner Example'),
        actions: _appBarActions(),
      ),
      body: OrientationBuilder(
        builder: (context, orientation) {
          if (_currentOrientation != orientation) {
            if (_currentOrientation != null) {
              _bannerAd?.dispose();
              _bannerAd = null;
              _loadAd();
            }
            _currentOrientation = orientation;
          }
          return Stack(
            children: [
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
```

`AppBar` 菜单项来自 `app_bar_item.dart`，动态插入隐私入口：

```dart
var array = [AppBarItem(AppBarItem.adInpsectorText, 0)];
if (_isPrivacyOptionsRequired) {
  array.add(AppBarItem(AppBarItem.privacySettingsText, 1));
}
```

## 练习与检查

- 切换横竖屏，确认 Banner 尺寸与定位正确。
- 打开菜单，验证 Ad Inspector 与隐私入口可选。
- 观察重新加载时是否有闪烁，必要时添加占位或过渡。

## 小结

你已掌握示例 UI 的组织方式，理解如何在布局中放置 Banner。下一章关注 Banner 请求参数与定向。
