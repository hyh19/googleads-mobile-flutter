# 第 6 章 iOS 布局设计（XIB）

## 引言

iOS 原生广告的布局使用 XIB 文件定义。XIB 文件是 Interface Builder 使用的 XML 格式文件，用于可视化设计界面。布局文件必须使用 `GADNativeAdView` 作为根视图，并包含所有需要的视图组件。本章将详细讲解如何设计原生广告的 XIB 布局，包括创建 XIB、连接 IBOutlet、设置约束等。

## XIB 文件概述

### 什么是 XIB？

XIB（XML Interface Builder）是 iOS 开发中用于定义用户界面的文件格式。它允许你使用 Interface Builder 可视化设计界面，也可以直接编辑 XML。

### 为什么使用 XIB？

- **可视化设计**：可以在 Xcode 的 Interface Builder 中拖拽设计
- **代码分离**：布局与逻辑代码分离
- **易于维护**：修改布局不需要修改代码

## 创建 XIB 文件

### 在 Xcode 中创建

1. 在 Xcode 中，右键点击 `Runner` 文件夹
2. 选择 "New File..."
3. 选择 "User Interface" > "View"
4. 命名为 `NativeAdView.xib`
5. 点击 "Create"

### 文件位置

XIB 文件应放在 `ios/Runner/` 目录下，与 `AppDelegate.swift` 同级。

## 设置根视图类

### 设置自定义类

1. 在 Interface Builder 中，选择根视图
2. 在 Identity Inspector 中，将 "Custom Class" 设置为 `GADNativeAdView`

```xml
<view contentMode="scaleToFill" id="iN0-l3-epB" customClass="GADNativeAdView">
    <!-- 布局内容 -->
</view>
```

## 添加必需的视图

### Headline View（标题）

添加一个 `UILabel` 作为标题视图：

```xml
<label opaque="NO" userInteractionEnabled="NO" contentMode="left" 
    text="Headline" lineBreakMode="tailTruncation" 
    translatesAutoresizingMaskIntoConstraints="NO" id="beR-eV-DX1">
    <fontDescription key="fontDescription" type="system" pointSize="17"/>
    <color key="textColor" systemColor="darkTextColor"/>
</label>
```

### Media View（媒体视图，必需）

添加一个 `GADMediaView` 作为媒体视图：

```xml
<view contentMode="scaleAspectFit" 
    translatesAutoresizingMaskIntoConstraints="NO" 
    id="fNp-yu-K4i" customClass="GADMediaView">
    <constraints>
        <constraint firstAttribute="height" constant="150" id="71m-kn-7Ug"/>
        <constraint firstAttribute="width" constant="250" id="e3T-fD-di4"/>
    </constraints>
</view>
```

## 添加可选视图

### Icon View（图标）

添加一个 `UIImageView` 作为图标视图：

```xml
<imageView userInteractionEnabled="NO" contentMode="scaleAspectFit" 
    translatesAutoresizingMaskIntoConstraints="NO" id="iNa-bH-h1m">
    <constraints>
        <constraint firstAttribute="height" constant="40" id="ICz-3W-FQf"/>
        <constraint firstAttribute="width" constant="40" id="vY6-8D-xIn"/>
    </constraints>
</imageView>
```

### Body View（正文）

添加一个 `UILabel` 作为正文视图：

```xml
<label opaque="NO" userInteractionEnabled="NO" contentMode="left" 
    text="Body text" lineBreakMode="tailTruncation" numberOfLines="0" 
    translatesAutoresizingMaskIntoConstraints="NO" id="PEQ-D9-2Vv">
    <fontDescription key="fontDescription" type="system" pointSize="14"/>
</label>
```

### Call to Action View（行动号召按钮）

添加一个 `UIButton` 作为行动号召按钮：

```xml
<button opaque="NO" contentMode="scaleToFill" 
    contentHorizontalAlignment="center" contentVerticalAlignment="center" 
    buttonType="system" translatesAutoresizingMaskIntoConstraints="NO" 
    id="E5w-YA-UY8">
    <constraints>
        <constraint firstAttribute="height" constant="30" id="rup-e7-1CR"/>
    </constraints>
    <state key="normal" title="Button"/>
</button>
```

### 其他可选视图

类似地，可以添加：

- **Price View**：价格标签（`UILabel`）
- **Store View**：商店名称标签（`UILabel`）
- **Star Rating View**：星级评分图片（`UIImageView`）
- **Advertiser View**：广告主名称标签（`UILabel`）

## 连接 IBOutlet

### 什么是 IBOutlet？

IBOutlet 是 Interface Builder 与代码之间的连接。在 XIB 文件中，通过 `<outlet>` 标签定义这些连接。

### 必需的 Outlet 连接

在 XIB 文件的 `<connections>` 部分，连接所有需要的视图：

```xml
<connections>
    <outlet property="headlineView" destination="beR-eV-DX1" id="d1E-ed-yel"/>
    <outlet property="mediaView" destination="fNp-yu-K4i" id="624-ZP-L04"/>
    <outlet property="bodyView" destination="PEQ-D9-2Vv" id="Gpd-Q6-Byv"/>
    <outlet property="iconView" destination="iNa-bH-h1m" id="gIe-xy-iwm"/>
    <outlet property="callToActionView" destination="E5w-YA-UY8" id="RCf-yK-s1x"/>
    <outlet property="priceView" destination="Ysb-of-cat" id="L6Q-hd-uaJ"/>
    <outlet property="storeView" destination="hwF-UL-Q8H" id="hRl-23-ce1"/>
    <outlet property="starRatingView" destination="2Of-AP-0h9" id="zCO-9D-S0V"/>
    <outlet property="advertiserView" destination="GTT-Yh-eSq" id="bY8-5O-6fF"/>
</connections>
```

### 在代码中定义 Outlet

在 `NativeAdView` 类中（如果使用自定义类），需要定义这些属性：

```swift
@IBOutlet weak var headlineView: UIView!
@IBOutlet weak var mediaView: GADMediaView!
@IBOutlet weak var bodyView: UIView!
@IBOutlet weak var iconView: UIView!
@IBOutlet weak var callToActionView: UIView!
@IBOutlet weak var priceView: UIView!
@IBOutlet weak var storeView: UIView!
@IBOutlet weak var starRatingView: UIView!
@IBOutlet weak var advertiserView: UIView!
```

**注意**：如果使用 `GADNativeAdView` 作为根视图类，这些属性已经定义，只需要连接即可。

## 布局约束设置

### 使用 Auto Layout

XIB 文件使用 Auto Layout 来设置视图约束。示例约束：

```xml
<constraints>
    <!-- Headline 约束 -->
    <constraint firstItem="beR-eV-DX1" firstAttribute="leading" 
        secondItem="iN0-l3-epB" secondAttribute="leading" constant="63" id="..."/>
    <constraint firstItem="beR-eV-DX1" firstAttribute="top" 
        secondItem="iN0-l3-epB" secondAttribute="top" constant="10" id="..."/>
    
    <!-- MediaView 约束 -->
    <constraint firstItem="fNp-yu-K4i" firstAttribute="centerX" 
        secondItem="iN0-l3-epB" secondAttribute="centerX" id="..."/>
    <constraint firstItem="fNp-yu-K4i" firstAttribute="top" 
        secondItem="beR-eV-DX1" secondAttribute="bottom" constant="20" id="..."/>
</constraints>
```

### 约束最佳实践

- **使用相对约束**：尽量使用相对位置约束，而不是固定坐标
- **设置优先级**：为某些约束设置优先级，以便在不同屏幕尺寸下自适应
- **测试不同设备**：在 Interface Builder 中测试不同设备尺寸

## 响应式设计

### 使用 Stack View

可以使用 `UIStackView` 来创建响应式布局：

```xml
<stackView opaque="NO" contentMode="scaleToFill" axis="vertical" 
    spacing="8" translatesAutoresizingMaskIntoConstraints="NO" id="...">
    <subviews>
        <!-- 子视图 -->
    </subviews>
    <constraints>
        <!-- 约束 -->
    </constraints>
</stackView>
```

### 适配不同屏幕尺寸

- 使用 Size Classes 来为不同设备设置不同的布局
- 使用约束优先级来处理不同屏幕尺寸
- 测试 iPhone 和 iPad 的显示效果

## 完整的 XIB 文件结构

以下是 XIB 文件的基本结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<document type="com.apple.InterfaceBuilder3.CocoaTouch.XIB" version="3.0">
    <dependencies>
        <deployment version="2048" identifier="iOS"/>
    </dependencies>
    <objects>
        <!-- File's Owner 和 First Responder -->
        <placeholder placeholderIdentifier="IBFilesOwner" id="-1"/>
        <placeholder placeholderIdentifier="IBFirstResponder" id="-2"/>
        
        <!-- 根视图：GADNativeAdView -->
        <view contentMode="scaleToFill" id="iN0-l3-epB" customClass="GADNativeAdView">
            <rect key="frame" x="0.0" y="0.0" width="375" height="667"/>
            <subviews>
                <!-- 所有子视图 -->
            </subviews>
            <constraints>
                <!-- 所有约束 -->
            </constraints>
            <connections>
                <!-- 所有 Outlet 连接 -->
            </connections>
        </view>
    </objects>
</document>
```

## 在 Interface Builder 中设计

### 可视化设计步骤

1. **打开 XIB 文件**：在 Xcode 中双击 `NativeAdView.xib`
2. **设置根视图类**：选择根视图，在 Identity Inspector 中设置 Custom Class
3. **添加视图**：从 Object Library 拖拽视图到画布
4. **设置约束**：为每个视图设置 Auto Layout 约束
5. **连接 Outlet**：在 Connections Inspector 中连接 Outlet
6. **设置属性**：在 Attributes Inspector 中设置视图属性

### 视图属性设置

- **Headline**：设置字体大小、颜色、样式
- **Body**：设置 `numberOfLines` 为 0 以支持多行
- **MediaView**：设置 `contentMode` 为 `scaleAspectFit` 或 `scaleAspectFill`
- **Button**：设置按钮样式和标题

## 实践练习

1. 在 Xcode 中创建 `NativeAdView.xib` 文件
2. 设置根视图类为 `GADNativeAdView`
3. 添加必需的视图（headline、mediaView）
4. 添加可选视图（body、icon、callToAction 等）
5. 设置 Auto Layout 约束
6. 连接所有 Outlet
7. 测试布局在不同设备上的显示

## 常见问题

### Q: 如果我不连接某个 Outlet 会怎样？

A: 如果工厂代码中引用了某个视图，但 XIB 中没有连接对应的 Outlet，会导致运行时错误。

### Q: 可以使用 Storyboard 吗？

A: 可以，但 XIB 更灵活，因为可以创建多个独立的视图文件。

### Q: MediaView 的尺寸应该设置多少？

A: 建议根据你的应用设计设置合适的尺寸。常见尺寸包括 250x150、match_parent 等。

### Q: 如何处理不同屏幕尺寸？

A: 使用 Auto Layout 约束和 Size Classes 来适配不同屏幕尺寸。

## 总结与检查清单

### 本章要点

- XIB 文件用于定义 iOS 原生广告布局
- 根视图必须是 `GADNativeAdView`
- 必须包含 `headlineView` 和 `mediaView`
- 可选视图可以根据需要添加
- 必须连接所有需要的 Outlet
- 使用 Auto Layout 设置约束

### 检查清单

在继续下一章之前，确保你已完成：

- [ ] 创建了 `NativeAdView.xib` 文件
- [ ] 设置根视图类为 `GADNativeAdView`
- [ ] 添加了必需的视图（headline、mediaView）
- [ ] 添加了需要的可选视图
- [ ] 设置了 Auto Layout 约束
- [ ] 连接了所有需要的 Outlet
- [ ] 测试了布局在不同设备上的显示

下一章，我们将学习如何在 Flutter 层集成原生广告。
