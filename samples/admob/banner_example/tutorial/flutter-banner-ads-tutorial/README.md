# Flutter Banner Ads 完整教程

## 教程简介

本教程面向熟悉 Flutter 开发但未接触过 Google Mobile Ads 的中级开发者，通过详细的步骤讲解和完整的代码示例，帮助你掌握 Banner Ads（横幅广告）的完整实现流程。

Banner Ads 是最常见的移动广告格式之一，通常显示在应用界面的顶部或底部。通过本教程，你将学习如何从零开始集成 Banner Ads，包括自适应广告尺寸、加载和显示广告、处理屏幕方向变化、监听广告事件等核心功能。

## 学习目标

完成本教程后，你将能够：

- 理解 Banner Ads 的概念和使用场景
- 配置 Flutter 项目以支持 Google Mobile Ads
- 实现自适应横幅广告尺寸
- 加载和显示 Banner Ads
- 处理屏幕方向变化
- 监听和处理各种广告事件
- 集成用户同意管理（UMP）以符合 GDPR 要求
- 遵循最佳实践优化用户体验

## 前置要求

在学习本教程之前，你需要：

- 熟悉 Flutter 和 Dart 编程语言
- 了解 Flutter 应用的基本结构和布局
- 具备 Android 和 iOS 平台的基础知识
- 拥有 Google AdMob 账号（用于获取广告单元 ID）

## 章节列表

1. [第 1 章：Banner Ads 简介](chapter-01-introduction.md)
2. [第 2 章：项目配置与依赖](chapter-02-project-setup.md)
3. [第 3 章：自适应横幅广告尺寸](chapter-03-ad-size.md)
4. [第 4 章：加载横幅广告](chapter-04-loading-ads.md)
5. [第 5 章：显示横幅广告](chapter-05-displaying-ads.md)
6. [第 6 章：处理屏幕方向变化](chapter-06-orientation-handling.md)
7. [第 7 章：广告事件监听](chapter-07-ad-events.md)
8. [第 8 章：用户同意管理（UMP）](chapter-08-consent-management.md)
9. [第 9 章：完整集成示例](chapter-09-complete-integration.md)
10. [第 10 章：最佳实践与常见问题](chapter-10-best-practices.md)

## 示例项目

本教程基于 `googleads-mobile-flutter` 项目中的 `banner_example` 示例。你可以参考该示例项目的完整代码实现。

## 重要提示

**始终使用测试广告进行开发和测试**

在构建和测试应用时，请确保使用测试广告而不是生产环境的真实广告。未遵循此要求可能导致账号被暂停。

测试广告单元 ID：

- Android: `ca-app-pub-3940256099942544/9214589741`
- iOS: `ca-app-pub-3940256099942544/2435281174`

## 开始学习

建议按照章节顺序学习，从 [第 1 章：Banner Ads 简介](chapter-01-introduction.md) 开始。

每个章节都包含：

- 核心概念讲解
- 详细的代码示例
- 实践练习
- 总结检查清单

祝你学习愉快！
