# 第 3 章 模板类型选择

## 引言

原生模板广告提供两种预定义模板类型：`small` 和 `medium`。选择合适的模板类型对于广告的展示效果和用户体验至关重要。本章将详细讲解两种模板类型的特点、尺寸、使用场景，以及如何选择合适的模板类型。

## TemplateType 枚举

### 可用类型

`TemplateType` 枚举定义了两种模板类型：

```dart
enum TemplateType {
  small,   // 小型模板
  medium,  // 中型模板
}
```

### 使用方式

在创建 `NativeTemplateStyle` 时指定模板类型：

```dart
nativeTemplateStyle: NativeTemplateStyle(
  templateType: TemplateType.medium,  // 选择模板类型
  // ... 其他样式配置
)
```

## Small 模板

### 特点

- **紧凑型布局**：占用空间较小
- **适合小屏幕**：在较小的展示区域中表现良好
- **简洁设计**：布局简洁，信息密度适中

### 尺寸规格

- **宽高比**：91/355（约 0.256）
- **实际尺寸**：根据容器宽度自动计算高度

### 代码示例

```dart
// Small 模板的宽高比
final double _adAspectRatioSmall = 91 / 355;

// 使用 Small 模板
nativeTemplateStyle: NativeTemplateStyle(
  templateType: TemplateType.small,
  // ... 样式配置
)

// 设置容器尺寸
SizedBox(
  height: MediaQuery.of(context).size.width * _adAspectRatioSmall,
  width: MediaQuery.of(context).size.width,
  child: AdWidget(ad: _nativeAd!),
)
```

### 使用场景

Small 模板适合以下场景：

- **列表项**：在列表或网格中作为列表项展示
- **侧边栏**：在侧边栏或侧边面板中展示
- **小卡片**：在较小的卡片容器中展示
- **紧凑布局**：需要节省空间的布局

### 布局结构

Small 模板通常包含：

- 图标（较小）
- 标题
- 简短的描述
- 行动号召按钮

## Medium 模板

### 特点

- **标准型布局**：占用空间适中
- **信息丰富**：可以展示更多广告信息
- **视觉突出**：更容易吸引用户注意

### 尺寸规格

- **宽高比**：370/355（约 1.042）
- **实际尺寸**：根据容器宽度自动计算高度

### 代码示例

```dart
// Medium 模板的宽高比
final double _adAspectRatioMedium = 370 / 355;

// 使用 Medium 模板
nativeTemplateStyle: NativeTemplateStyle(
  templateType: TemplateType.medium,
  // ... 样式配置
)

// 设置容器尺寸
SizedBox(
  height: MediaQuery.of(context).size.width * _adAspectRatioMedium,
  width: MediaQuery.of(context).size.width,
  child: AdWidget(ad: _nativeAd!),
)
```

### 使用场景

Medium 模板适合以下场景：

- **内容流**：在内容流中作为主要展示单元
- **主页面**：在应用主页面中展示
- **大卡片**：在较大的卡片容器中展示
- **突出展示**：需要突出展示的广告位置

### 布局结构

Medium 模板通常包含：

- 图标（较大）
- 标题
- 详细描述
- 媒体视图（图片或视频）
- 行动号召按钮
- 其他元数据（如评分、价格等）

## 如何选择合适的模板类型

### 考虑因素

选择模板类型时，考虑以下因素：

1. **展示空间**：可用的展示空间大小
2. **内容重要性**：广告在页面中的重要性
3. **用户体验**：哪种模板能提供更好的用户体验
4. **设计一致性**：与应用整体设计的一致性

### 选择指南

#### 选择 Small 模板的情况

- 展示空间有限
- 广告作为次要内容
- 需要节省屏幕空间
- 在列表或网格中展示

#### 选择 Medium 模板的情况

- 有足够的展示空间
- 广告作为主要内容
- 需要突出展示
- 在内容流中展示

### 对比表格

| 特性 | Small 模板 | Medium 模板 |
|------|-----------|-------------|
| **宽高比** | 91/355 (0.256) | 370/355 (1.042) |
| **占用空间** | 小 | 中 |
| **信息量** | 较少 | 较多 |
| **视觉突出度** | 较低 | 较高 |
| **适用场景** | 列表项、侧边栏 | 内容流、主页面 |

## 模板布局结构说明

### Small 模板布局

```text
┌─────────────────────────┐
│ [Icon]  Title           │
│         Description     │
│         [CTA Button]    │
└─────────────────────────┘
```

### Medium 模板布局

```text
┌─────────────────────────┐
│ [Icon]  Title           │
│         Advertiser      │
│         Description     │
│         [Media View]    │
│         [CTA Button]    │
└─────────────────────────┘
```

## 响应式设计考虑

### 使用宽高比

无论选择哪种模板，都应该使用宽高比来设置容器尺寸，以确保在不同屏幕尺寸上正确显示：

```dart
// 根据模板类型选择宽高比
final double _adAspectRatio = 
    templateType == TemplateType.small 
        ? 91 / 355 
        : 370 / 355;

// 设置容器尺寸
SizedBox(
  height: MediaQuery.of(context).size.width * _adAspectRatio,
  width: MediaQuery.of(context).size.width,
  child: AdWidget(ad: _nativeAd!),
)
```

### 适配不同屏幕

模板会自动适配不同屏幕尺寸，但你需要：

1. **设置正确的容器尺寸**：使用宽高比计算高度
2. **考虑屏幕方向**：横屏和竖屏可能需要不同的处理
3. **测试不同设备**：在多种设备上测试显示效果

## 实践练习

1. 根据你的应用场景，选择 Small 或 Medium 模板
2. 计算并设置正确的容器尺寸
3. 在不同设备上测试显示效果
4. 根据用户体验调整选择

## 常见问题

### Q: 可以同时使用两种模板类型吗？

A: 可以。你可以在同一个应用中使用不同的模板类型，只需要创建不同的 `NativeAd` 实例，每个实例使用不同的 `TemplateType`。

### Q: 模板类型可以在运行时更改吗？

A: 不可以。模板类型在创建 `NativeAd` 时确定，不能动态更改。如果需要更改，需要创建新的 `NativeAd` 实例。

### Q: Small 和 Medium 模板的收益有区别吗？

A: 模板类型本身不影响收益，但展示位置和用户体验可能影响点击率和收益。选择适合的模板类型可以提高用户体验和点击率。

### Q: 如何知道哪种模板更适合我的应用？

A: 建议进行 A/B 测试，比较两种模板在你的应用中的表现，包括点击率、用户反馈等指标。

## 总结与检查清单

### 本章要点

- 原生模板广告提供 Small 和 Medium 两种模板类型
- Small 模板：宽高比 91/355，适合紧凑布局
- Medium 模板：宽高比 370/355，适合标准布局
- 选择模板类型需要考虑展示空间、内容重要性等因素
- 使用宽高比设置容器尺寸以确保响应式设计

### 检查清单

在继续下一章之前，确保你理解：

- [ ] Small 和 Medium 模板的特点
- [ ] 两种模板的尺寸规格和宽高比
- [ ] 如何选择合适的模板类型
- [ ] 如何使用宽高比设置容器尺寸

下一章，我们将学习如何配置模板样式。
