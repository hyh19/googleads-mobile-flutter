# 第 9 章 发布与政策合规

本章提醒上线前的合规要点，覆盖隐私、内容政策与平台审核要求，避免因广告违规导致下架。

## 简介

AdMob 要求展示隐私政策、遵守内容与流量质量规范。应用分发渠道（Play/App Store）也要求披露数据收集与追踪声明。

## 核心步骤

1. 提供可访问的隐私政策页面，并在应用内合适位置展示链接。
2. 保持同意流程可触达：在 AppBar 菜单暴露隐私入口。
3. 上架前替换为真实广告单元，并在测试完成后移除测试设备配置。
4. 按商店要求填写数据安全与跟踪透明度表单。

## 代码示例

隐私入口的菜单添加逻辑：

```dart
var array = [AppBarItem(AppBarItem.adInpsectorText, 0)];
if (_isPrivacyOptionsRequired) {
  array.add(AppBarItem(AppBarItem.privacySettingsText, 1));
}
```

## 练习与检查

- 确认隐私政策 URL 在应用内可见。
- 检查是否仍使用测试 Unit ID；如要上线需替换为生产 ID。
- 复核商店提交流程，填写数据安全与追踪声明。

## 小结

完成合规配置后，你的应用更容易通过审核并保持账号健康。
