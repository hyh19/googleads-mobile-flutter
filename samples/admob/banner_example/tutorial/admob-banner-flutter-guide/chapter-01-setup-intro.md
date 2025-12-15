# 第 1 章 环境与项目概览

本章说明示例所需的工具链、依赖和目录结构，帮助你快速跑通 `banner_example`。完成后你将知道核心文件位置和运行方式。

## 简介

`google_mobile_ads` 提供 Flutter 端的 AdMob SDK 封装。示例位于 `lib` 目录，重点文件是 `main.dart`（广告逻辑）与 `consent_manager.dart`（同意管理）。

## 核心步骤

1. 安装 Flutter（3.x 及以上）与 Dart，配置 Android SDK 与 Xcode。
2. 在项目根目录执行 `flutter pub get` 安装依赖。
3. 使用模拟器或真机运行：`flutter run`。
4. 熟悉目录：`lib/main.dart` 管理 Banner，`lib/consent_manager.dart` 处理 GDPR，同意入口在 AppBar 菜单。

## 代码示例

最小的入口在 `main.dart`：

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const MaterialApp(home: BannerExample()));
}
```

`BannerExample` 负责请求同意并初始化 Mobile Ads。

## 练习与检查

- 运行 `flutter doctor` 确认无缺失组件。
- 在项目根执行 `flutter pub get` 成功且无错误。
- `flutter run` 能在模拟器或真机启动示例。

## 小结

你已准备好开发环境，理解了示例的入口与关键文件位置。下一章将配置 AdMob 账户与测试广告单元。
