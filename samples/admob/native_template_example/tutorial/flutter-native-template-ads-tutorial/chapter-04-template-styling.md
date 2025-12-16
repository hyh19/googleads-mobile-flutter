# 第 4 章 模板样式配置

## 引言

虽然原生模板广告使用预定义模板，但你仍然可以通过 `NativeTemplateStyle` 自定义广告的外观，包括背景颜色、文本颜色、字体样式和大小等。本章将详细讲解如何配置模板样式，让你的广告与应用设计完美融合。

## NativeTemplateStyle 类概述

### 类定义

`NativeTemplateStyle` 是用于配置原生模板广告样式的类：

```dart
class NativeTemplateStyle {
  final TemplateType templateType;
  final Color? mainBackgroundColor;
  final NativeTemplateTextStyle? callToActionTextStyle;
  final NativeTemplateTextStyle? primaryTextStyle;
  final NativeTemplateTextStyle? secondaryTextStyle;
  final NativeTemplateTextStyle? tertiaryTextStyle;
  
  const NativeTemplateStyle({
    required this.templateType,
    this.mainBackgroundColor,
    this.callToActionTextStyle,
    this.primaryTextStyle,
    this.secondaryTextStyle,
    this.tertiaryTextStyle,
  });
}
```

### 必需参数

- `templateType`：模板类型（`TemplateType.small` 或 `TemplateType.medium`）

### 可选参数

- `mainBackgroundColor`：主背景颜色
- `callToActionTextStyle`：行动号召按钮文本样式
- `primaryTextStyle`：主要文本样式（标题）
- `secondaryTextStyle`：次要文本样式
- `tertiaryTextStyle`：第三级文本样式

## 主背景颜色

### mainBackgroundColor

设置广告的主背景颜色：

```dart
nativeTemplateStyle: NativeTemplateStyle(
  templateType: TemplateType.medium,
  mainBackgroundColor: const Color(0xfffffbed),  // 浅黄色背景
  // ... 其他样式
)
```

### 颜色选择建议

- **与应用主题一致**：选择与应用整体设计风格一致的颜色
- **对比度**：确保文本颜色与背景颜色有足够的对比度
- **可读性**：避免使用过于鲜艳或刺眼的颜色

### 示例

```dart
// 浅色背景
mainBackgroundColor: const Color(0xfffffbed)

// 白色背景
mainBackgroundColor: Colors.white

// 深色背景
mainBackgroundColor: const Color(0xff1a1a1a)
```

## 文本样式配置

### NativeTemplateTextStyle

`NativeTemplateTextStyle` 用于配置文本样式：

```dart
class NativeTemplateTextStyle {
  final Color textColor;
  final NativeTemplateFontStyle style;
  final double size;
  
  const NativeTemplateTextStyle({
    required this.textColor,
    required this.style,
    required this.size,
  });
}
```

### 文本样式类型

原生模板广告支持四种文本样式：

1. **callToActionTextStyle**：行动号召按钮文本样式
2. **primaryTextStyle**：主要文本样式（标题）
3. **secondaryTextStyle**：次要文本样式
4. **tertiaryTextStyle**：第三级文本样式

## 字体样式

### NativeTemplateFontStyle 枚举

`NativeTemplateFontStyle` 定义了可用的字体样式：

```dart
enum NativeTemplateFontStyle {
  normal,     // 正常字体
  bold,       // 粗体
  italic,     // 斜体
  monospace,  // 等宽字体
}
```

### 使用示例

```dart
// 粗体标题
primaryTextStyle: NativeTemplateTextStyle(
  textColor: Colors.black,
  style: NativeTemplateFontStyle.bold,
  size: 16.0,
)

// 斜体描述
secondaryTextStyle: NativeTemplateTextStyle(
  textColor: Colors.black,
  style: NativeTemplateFontStyle.italic,
  size: 16.0,
)

// 等宽字体按钮
callToActionTextStyle: NativeTemplateTextStyle(
  textColor: Colors.white,
  style: NativeTemplateFontStyle.monospace,
  size: 16.0,
)
```

## 完整样式配置示例

以下是示例项目中的完整样式配置：

```dart
nativeTemplateStyle: NativeTemplateStyle(
  templateType: TemplateType.medium,
  mainBackgroundColor: const Color(0xfffffbed),
  callToActionTextStyle: NativeTemplateTextStyle(
    textColor: Colors.white,
    style: NativeTemplateFontStyle.monospace,
    size: 16.0,
  ),
  primaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.black,
    style: NativeTemplateFontStyle.bold,
    size: 16.0,
  ),
  secondaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.black,
    style: NativeTemplateFontStyle.italic,
    size: 16.0,
  ),
  tertiaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.black,
    style: NativeTemplateFontStyle.normal,
    size: 16.0,
  ),
)
```

## 样式配置最佳实践

### 1. 与应用主题一致

选择与应用整体设计风格一致的颜色和字体：

```dart
// 如果你的应用使用深色主题
mainBackgroundColor: const Color(0xff1a1a1a),
primaryTextStyle: NativeTemplateTextStyle(
  textColor: Colors.white,
  style: NativeTemplateFontStyle.bold,
  size: 16.0,
)
```

### 2. 确保可读性

确保文本颜色与背景颜色有足够的对比度：

```dart
// ✅ 好：深色文本配浅色背景
mainBackgroundColor: Colors.white,
primaryTextStyle: NativeTemplateTextStyle(
  textColor: Colors.black,
  // ...
)

// ❌ 差：浅色文本配浅色背景（对比度不足）
mainBackgroundColor: Colors.white,
primaryTextStyle: NativeTemplateTextStyle(
  textColor: Colors.lightGray,
  // ...
)
```

### 3. 突出行动号召按钮

使用醒目的颜色和样式突出行动号召按钮：

```dart
callToActionTextStyle: NativeTemplateTextStyle(
  textColor: Colors.white,  // 白色文本
  style: NativeTemplateFontStyle.bold,  // 粗体
  size: 16.0,
)
// 注意：按钮背景颜色由 SDK 自动设置
```

### 4. 合理的字体大小

选择适合的字体大小，确保在不同设备上都能清晰阅读：

```dart
// 标题：稍大
primaryTextStyle: NativeTemplateTextStyle(
  size: 18.0,  // 或 16.0
)

// 描述：标准大小
secondaryTextStyle: NativeTemplateTextStyle(
  size: 14.0,  // 或 16.0
)
```

## 不同场景的样式配置

### 浅色主题

```dart
nativeTemplateStyle: NativeTemplateStyle(
  templateType: TemplateType.medium,
  mainBackgroundColor: Colors.white,
  callToActionTextStyle: NativeTemplateTextStyle(
    textColor: Colors.white,
    style: NativeTemplateFontStyle.bold,
    size: 16.0,
  ),
  primaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.black87,
    style: NativeTemplateFontStyle.bold,
    size: 18.0,
  ),
  secondaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.black54,
    style: NativeTemplateFontStyle.normal,
    size: 14.0,
  ),
  tertiaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.black54,
    style: NativeTemplateFontStyle.normal,
    size: 14.0,
  ),
)
```

### 深色主题

```dart
nativeTemplateStyle: NativeTemplateStyle(
  templateType: TemplateType.medium,
  mainBackgroundColor: const Color(0xff1a1a1a),
  callToActionTextStyle: NativeTemplateTextStyle(
    textColor: Colors.white,
    style: NativeTemplateFontStyle.bold,
    size: 16.0,
  ),
  primaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.white,
    style: NativeTemplateFontStyle.bold,
    size: 18.0,
  ),
  secondaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.white70,
    style: NativeTemplateFontStyle.normal,
    size: 14.0,
  ),
  tertiaryTextStyle: NativeTemplateTextStyle(
    textColor: Colors.white70,
    style: NativeTemplateFontStyle.normal,
    size: 14.0,
  ),
)
```

## 实践练习

1. 根据你的应用主题，配置合适的背景颜色
2. 为不同类型的文本设置合适的样式
3. 确保文本颜色与背景颜色有足够的对比度
4. 测试样式在不同设备上的显示效果

## 常见问题

### Q: 可以只配置部分样式吗？

A: 可以。所有样式参数都是可选的，你可以只配置需要的部分。未配置的样式将使用默认值。

### Q: 字体大小可以使用小数吗？

A: 可以。`size` 参数是 `double` 类型，可以使用小数，如 `16.5`。

### Q: 如何知道哪些文本使用哪种样式？

A: 主要文本（标题）使用 `primaryTextStyle`，次要文本使用 `secondaryTextStyle`，第三级文本使用 `tertiaryTextStyle`，行动号召按钮使用 `callToActionTextStyle`。

### Q: 可以动态更改样式吗？

A: 不可以。样式在创建 `NativeAd` 时确定，不能动态更改。如果需要更改，需要创建新的 `NativeAd` 实例。

## 总结与检查清单

### 本章要点

- `NativeTemplateStyle` 用于配置原生模板广告的样式
- 可以配置背景颜色和四种文本样式
- `NativeTemplateFontStyle` 提供四种字体样式（normal、bold、italic、monospace）
- 样式配置应该与应用主题一致，确保可读性

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `NativeTemplateStyle` 的用法
- [ ] 如何配置背景颜色
- [ ] 如何配置文本样式
- [ ] 四种字体样式的区别
- [ ] 样式配置的最佳实践

下一章，我们将学习如何加载原生模板广告。
