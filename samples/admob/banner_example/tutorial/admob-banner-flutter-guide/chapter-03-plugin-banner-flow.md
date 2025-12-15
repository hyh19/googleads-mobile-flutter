# 第 3 章 插件与基础 Banner 加载流程

本章梳理 `google_mobile_ads` 的初始化、Banner 请求与回调处理，基于示例 `main.dart` 讲解核心流程。

## 简介

流程顺序：收集同意 → 初始化 Mobile Ads → 计算自适应尺寸 → 构建 `BannerAd` → 监听回调 → 将 `AdWidget` 挂到页面底部。

## 核心步骤

1. 在 `initState` 中收集同意并尝试初始化 SDK。
2. 通过 `AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize` 获取当前方向的自适应尺寸。
3. 创建 `BannerAd`，传入 `adUnitId`、`AdRequest`、`size` 和 `BannerAdListener`。
4. 加载完成后将 `AdWidget` 放入 `Align + SafeArea`，展示在底部。

## 代码示例

```dart
final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
  MediaQuery.sizeOf(context).width.truncate(),
);

BannerAd(
  adUnitId: _adUnitId,
  request: const AdRequest(),
  size: size!,
  listener: BannerAdListener(
    onAdLoaded: (ad) {
      debugPrint("Ad was loaded.");
      setState(() => _bannerAd = ad as BannerAd);
    },
    onAdFailedToLoad: (ad, err) {
      debugPrint("Ad failed to load with error: $err");
      ad.dispose();
    },
  ),
).load();
```

## 练习与检查

- 在加载前打印 `size`，确认自适应尺寸非空。
- 观察控制台日志，确认 `onAdLoaded` 与失败回调输出。
- 手动旋转设备，验证横竖屏切换后重新加载 Banner。

## 小结

你已掌握基础加载流程与关键 API。下一章将加入同意管理与 GDPR 处理。
