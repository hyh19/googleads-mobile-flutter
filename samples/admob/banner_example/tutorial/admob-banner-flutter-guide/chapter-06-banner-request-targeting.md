# 第 6 章 Banner 请求参数与定位

本章介绍如何配置 Banner 请求参数、选择自适应尺寸，并理解回调事件便于调试和优化。

## 简介

`AdRequest` 支持传入关键字、内容 URL、非个性化标记等。示例使用默认请求，重点展示自适应尺寸获取与监听事件。

## 核心步骤

1. 获取自适应 Banner 尺寸，避免布局错位。
2. 构建 `AdRequest`，可按需添加关键词或内容 URL。
3. 使用 `BannerAdListener` 观察加载、展示、点击等事件。

## 代码示例

```dart
final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
  MediaQuery.sizeOf(context).width.truncate(),
);

BannerAd(
  adUnitId: _adUnitId,
  request: const AdRequest(
    keywords: ['news', 'tech'],
    nonPersonalizedAds: false,
  ),
  size: size!,
  listener: BannerAdListener(
    onAdImpression: (ad) => debugPrint("Ad recorded an impression."),
    onAdClicked: (ad) => debugPrint("Ad was clicked."),
  ),
).load();
```

如需仅请求非个性化广告，可设置 `nonPersonalizedAds: true`。

## 练习与检查

- 修改关键字并观察点击率和填充率变化（上线后评估）。
- 记录 `onAdFailedToLoad` 的错误码，结合官方文档定位问题。
- 在不同屏幕尺寸设备上验证 Banner 尺寸正确性。

## 小结

你已了解请求配置、自适应尺寸与事件回调，为后续调试和优化奠定基础。下一章将聚焦调试与测试策略。
