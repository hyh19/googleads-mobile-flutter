# Flutter App Open Ads 完整教程

## 教程简介

本教程面向熟悉 Flutter 开发但未接触过 Google Mobile Ads 的中级开发者，通过详细的步骤讲解和完整的代码示例，帮助你掌握 App Open Ads 的完整实现流程。

App Open Ads 是一种特殊的全屏广告格式，设计用于在用户将应用切换到前台时显示。通过本教程，你将学习如何从零开始集成 App Open Ads，包括广告加载、展示、生命周期管理、用户同意处理等核心功能。

## 学习目标

完成本教程后，你将能够：

- 理解 App Open Ads 的概念和使用场景
- 配置 Flutter 项目以支持 Google Mobile Ads
- 实现完整的 App Open Ads 管理器
- 处理应用生命周期事件以适时显示广告
- 实现广告过期检测和自动重新加载
- 集成用户同意管理（UMP）以符合 GDPR 要求
- 处理冷启动场景和优化用户体验
- 遵循最佳实践避免常见错误

## 前置要求

在学习本教程之前，你需要：

- 熟悉 Flutter 和 Dart 编程语言
- 了解 Flutter 应用的基本结构和生命周期
- 具备 Android 和 iOS 平台的基础知识
- 拥有 Google AdMob 账号（用于获取广告单元 ID）

## 章节列表

1. [第 1 章：App Open Ads 简介](chapter-01-introduction.md)
2. [第 2 章：项目配置与依赖](chapter-02-project-setup.md)
3. [第 3 章：创建广告管理器基础类](chapter-03-ad-manager-basics.md)
4. [第 4 章：加载 App Open 广告](chapter-04-loading-ads.md)
5. [第 5 章：展示广告与回调处理](chapter-05-showing-ads.md)
6. [第 6 章：应用生命周期管理](chapter-06-lifecycle-management.md)
7. [第 7 章：广告过期处理](chapter-07-ad-expiration.md)
8. [第 8 章：用户同意管理（UMP）](chapter-08-consent-management.md)
9. [第 9 章：冷启动与加载屏幕](chapter-09-cold-starts.md)
10. [第 10 章：完整集成示例](chapter-10-complete-integration.md)
11. [第 11 章：最佳实践与常见问题](chapter-11-best-practices.md)

## 示例项目

本教程基于 `googleads-mobile-flutter` 项目中的 `app_open_example` 示例。你可以参考该示例项目的完整代码实现。

## 重要提示

**始终使用测试广告进行开发和测试**

在构建和测试应用时，请确保使用测试广告而不是生产环境的真实广告。未遵循此要求可能导致账号被暂停。

测试广告单元 ID：

- Android: `ca-app-pub-3940256099942544/9257395921`
- iOS: `ca-app-pub-3940256099942544/5575463023`

## 开始学习

建议按照章节顺序学习，从 [第 1 章：App Open Ads 简介](chapter-01-introduction.md) 开始。

每个章节都包含：

- 核心概念讲解
- 详细的代码示例
- 实践练习
- 总结检查清单

祝你学习愉快！
