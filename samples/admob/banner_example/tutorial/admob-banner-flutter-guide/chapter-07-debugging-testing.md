# 第 7 章 调试与测试策略

本章提供真机测试、日志调试和常见错误处理方法，确保 Banner 在开发阶段稳定可验证。

## 简介

使用测试广告单元、开启详细日志，并在真机上验证网络与同意流程是关键。Ad Inspector 可快速查看填充、创意与错误。

## 核心步骤

1. 保持测试广告单元，必要时设置测试设备 ID。
2. 调试日志：监听 `BannerAdListener`，输出加载与错误信息。
3. 使用 AppBar 的 Ad Inspector 菜单项查看实时状态。
4. 真机测试网络与 VPN，确保同意表单能正常拉取。

## 代码示例

```dart
onAdFailedToLoad: (ad, err) {
  debugPrint("Ad failed to load with error: $err");
  ad.dispose();
},

MobileAds.instance.openAdInspector((error) {
  if (error != null) {
    debugPrint("Ad Inspector error: ${error.message}");
  }
});
```

## 练习与检查

- 真机切换 Wi-Fi/蜂窝网络，验证广告能加载。
- 触发 `onAdFailedToLoad`，记录错误码并查阅官方对照表。
- 打开 Ad Inspector，确认请求状态与响应详情。

## 小结

通过日志与 Ad Inspector，你可以快速定位加载问题，为上线前的质量保障打下基础。
