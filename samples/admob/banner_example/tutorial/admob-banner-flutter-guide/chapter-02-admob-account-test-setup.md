# 第 2 章 AdMob 账户与测试配置

本章帮助你准备 AdMob 账户、创建 Banner 广告单元，并在开发阶段使用测试配置避免违规流量。

## 简介

使用官方测试广告单元可避免无效流量风险。示例内置 Android 与 iOS 的测试 Unit ID，便于即刻验证。

## 核心步骤

1. 登录 AdMob 控制台创建应用与 Banner 广告单元，记录真实 Unit ID（上线时替换）。
2. 开发阶段使用测试 Unit ID：Android `ca-app-pub-3940256099942544/9214589741`，iOS `ca-app-pub-3940256099942544/2435281174`。
3. 如需自测设备，调用 `MobileAds.instance.updateRequestConfiguration` 添加测试设备 ID。

## 代码示例

`main.dart` 中的内置测试 Unit ID：

```dart
final String _adUnitId = Platform.isAndroid
    ? 'ca-app-pub-3940256099942544/9214589741'
    : 'ca-app-pub-3940256099942544/2435281174';
```

切换到生产单元时，将上述字符串替换为你的真实 Unit ID。

## 练习与检查

- 在 AdMob 创建一个 Banner 单元，确认已获得 Unit ID。
- 保持示例使用测试 ID 运行，确保广告展示正常。
- 可选：打印 `RequestConfiguration().testDeviceIds` 确认测试设备设置生效。

## 小结

你已准备好测试配置并了解上线前需替换的单元 ID。接下来会深入 Banner 加载流程与插件 API。
