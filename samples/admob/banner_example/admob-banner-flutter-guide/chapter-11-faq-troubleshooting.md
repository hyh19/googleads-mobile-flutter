# 第 11 章 常见问题与排查

本章列举开发中常见问题及处理思路，帮助你快速定位并解决 Banner 加载异常。

## 简介

问题多集中在同意未就绪、网络异常、广告单元配置错误或 SDK 初始化不完整。合理使用回调与日志能显著缩短排查时间。

## 核心步骤

1. 同意未完成：检查 `canRequestAds` 返回值并确认表单是否展示。
2. 无填充：确认地区、流量质量及测试单元是否误用于生产。
3. 初始化未完成：确保未重复调用初始化且在 `initState` 中执行。
4. 尺寸为空：当 `AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize` 返回 null 时，检查 `MediaQuery` 是否可用或布局是否在首帧后执行。

## 代码示例

```dart
if (!await _consentManager.canRequestAds()) {
  debugPrint("Consent not ready, skip loading ad.");
  return;
}

if (size == null) {
  debugPrint("Anchored banner size is null, skip loading.");
  return;
}
```

## 练习与检查

- 刻意阻断网络，观察错误码并确认失败回调被触发。
- 使用无效广告单元测试，确认错误日志能提示配置问题。
- 在布局尚未完成时调用 `_loadAd`，验证尺寸返回空的场景并补充防护。

## 小结

通过系统化的检查列表，你可以快速定位加载问题并提升稳定性。
