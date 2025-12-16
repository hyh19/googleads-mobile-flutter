# 第 5 章 iOS 原生工厂实现

## 引言

在 iOS 平台，原生广告需要通过实现 `FLTNativeAdFactory` 协议来创建原生广告视图。本章将详细讲解如何在 iOS 中实现原生广告工厂，包括注册工厂、从 XIB 加载视图、设置视图引用、填充数据和处理可选字段。

## FLTNativeAdFactory 协议概述

### 协议定义

`FLTNativeAdFactory` 是 Google Mobile Ads Flutter 插件提供的协议，用于创建原生广告视图：

```swift
protocol FLTNativeAdFactory {
    func createNativeAd(
        _ nativeAd: NativeAd,
        customOptions: [AnyHashable : Any]?
    ) -> NativeAdView?
}
```

### 协议方法

- `createNativeAd()`：创建并返回一个 `NativeAdView`，该视图包含填充了广告数据的 UI 组件

## 在 AppDelegate 中注册工厂

### 注册工厂

在 `AppDelegate` 的 `application(_:didFinishLaunchingWithOptions:)` 方法中注册工厂：

```swift
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)

    FLTGoogleMobileAdsPlugin.registerNativeAdFactory(
        self,
        factoryId: "adFactoryExample",  // factoryId，必须与 Flutter 层匹配
        nativeAdFactory: NativeAdFactoryExample()
    )

    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

### factoryId 的重要性

`factoryId` 是一个字符串标识符，用于将 Flutter 层的广告请求与原生层的工厂关联起来。**重要**：`factoryId` 必须在 Flutter 层和原生层完全匹配。

## 实现 NativeAdFactoryExample 类

### 类结构

```swift
@objc class NativeAdFactoryExample: NSObject, FLTNativeAdFactory {
  func createNativeAd(
      _ nativeAd: NativeAd,
      customOptions: [AnyHashable : Any]? = nil
  ) -> NativeAdView? {
    // 实现创建逻辑
  }
}
```

### 从 XIB 加载视图

首先，从 XIB 文件加载 `NativeAdView`：

```swift
func createNativeAd(
    _ nativeAd: NativeAd,
    customOptions: [AnyHashable : Any]? = nil
) -> NativeAdView? {
  // 从 XIB 加载视图
  let nativeAdView = Bundle.main.loadNibNamed(
      "NativeAdView",
      owner: nil,
      options: nil
  )?.first as! NativeAdView
  
  // 设置视图引用和填充数据
  // ...
  
  return nativeAdView
}
```

## 设置视图引用和广告数据

### 关联 NativeAd 对象

首先，将 `NativeAd` 对象关联到视图。这是使广告可点击的关键步骤：

```swift
nativeAdView.nativeAd = nativeAd
```

### 填充必需字段

`headline` 和 `mediaContent` 是保证存在的字段：

```swift
// headline 和 mediaContent 保证存在
(nativeAdView.headlineView as? UILabel)?.text = nativeAd.headline
nativeAdView.mediaView?.mediaContent = nativeAd.mediaContent
```

### 填充可选字段

其他字段可能为 `nil`，需要检查后再设置：

```swift
// body（可选）
(nativeAdView.bodyView as? UILabel)?.text = nativeAd.body

// icon（可选）
(nativeAdView.iconView as? UIImageView)?.image = nativeAd.icon?.image

// callToAction（可选）
(nativeAdView.callToActionView as? UIButton)?.setTitle(
    nativeAd.callToAction,
    for: .normal
)

// price（可选）
(nativeAdView.priceView as? UILabel)?.text = nativeAd.price

// store（可选）
(nativeAdView.storeView as? UILabel)?.text = nativeAd.store

// advertiser（可选）
(nativeAdView.advertiserView as? UILabel)?.text = nativeAd.advertiser
```

## 处理星级评分

### 星级评分转换

iOS 中的 `starRating` 是 `NSDecimalNumber` 类型，需要转换为图片：

```swift
private func imageOfStars(from starRating: NSDecimalNumber?) -> UIImage? {
  guard let rating = starRating?.doubleValue else {
    return nil
  }
  
  if rating >= 5 {
    return UIImage(named: "stars_5")
  } else if rating >= 4.5 {
    return UIImage(named: "stars_4_5")
  } else if rating >= 4 {
    return UIImage(named: "stars_4")
  } else if rating >= 3.5 {
    return UIImage(named: "stars_3_5")
  } else {
    return nil
  }
}
```

### 设置星级评分视图

```swift
(nativeAdView.starRatingView as? UIImageView)?.image = imageOfStars(
    from: nativeAd.starRating
)
```

## 禁用用户交互

### 重要：禁用 Call to Action 按钮的交互

为了确保 SDK 能正确处理点击事件，必须禁用 `callToActionView` 的用户交互：

```swift
// 禁用用户交互，让 SDK 处理点击事件
nativeAdView.callToActionView?.isUserInteractionEnabled = false
```

### 最终关联 NativeAd

在填充完所有数据后，再次设置 `nativeAd` 属性（这是必需的）：

```swift
// 重要：必须在填充完所有视图后设置
nativeAdView.nativeAd = nativeAd
```

## 完整的实现示例

以下是示例项目中的完整实现：

```swift
import UIKit
import Flutter

@objc class NativeAdFactoryExample: NSObject, FLTNativeAdFactory {
  func createNativeAd(
      _ nativeAd: NativeAd,
      customOptions: [AnyHashable : Any]? = nil
  ) -> NativeAdView? {
    
    // 从 XIB 加载视图
    let nativeAdView = Bundle.main.loadNibNamed(
        "NativeAdView",
        owner: nil,
        options: nil
    )?.first as! NativeAdView

    // 关联 NativeAd 对象（第一次）
    nativeAdView.nativeAd = nativeAd

    // 填充必需字段
    (nativeAdView.headlineView as? UILabel)?.text = nativeAd.headline
    nativeAdView.mediaView?.mediaContent = nativeAd.mediaContent

    // 填充可选字段
    (nativeAdView.bodyView as? UILabel)?.text = nativeAd.body
    (nativeAdView.iconView as? UIImageView)?.image = nativeAd.icon?.image
    (nativeAdView.starRatingView as? UIImageView)?.image = imageOfStars(
        from: nativeAd.starRating
    )
    (nativeAdView.storeView as? UILabel)?.text = nativeAd.store
    (nativeAdView.priceView as? UILabel)?.text = nativeAd.price
    (nativeAdView.advertiserView as? UILabel)?.text = nativeAd.advertiser
    (nativeAdView.callToActionView as? UIButton)?.setTitle(
        nativeAd.callToAction,
        for: .normal
    )

    // 禁用用户交互，让 SDK 处理点击事件
    nativeAdView.callToActionView?.isUserInteractionEnabled = false

    // 重要：再次设置 nativeAd（必须在填充完所有视图后）
    nativeAdView.nativeAd = nativeAd

    return nativeAdView
  }

  private func imageOfStars(from starRating: NSDecimalNumber?) -> UIImage? {
    guard let rating = starRating?.doubleValue else {
      return nil
    }
    if rating >= 5 {
      return UIImage(named: "stars_5")
    } else if rating >= 4.5 {
      return UIImage(named: "stars_4_5")
    } else if rating >= 4 {
      return UIImage(named: "stars_4")
    } else if rating >= 3.5 {
      return UIImage(named: "stars_3_5")
    } else {
      return nil
    }
  }
}
```

## AppDelegate 完整实现

```swift
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)

    FLTGoogleMobileAdsPlugin.registerNativeAdFactory(
        self,
        factoryId: "adFactoryExample",
        nativeAdFactory: NativeAdFactoryExample()
    )

    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}

// NativeAdFactoryExample 实现（见上文）
```

## 处理 customOptions

`customOptions` 参数允许 Flutter 层传递自定义选项到原生层：

```swift
func createNativeAd(
    _ nativeAd: NativeAd,
    customOptions: [AnyHashable : Any]? = nil
) -> NativeAdView? {
  // 使用 customOptions
  if let backgroundColor = customOptions?["backgroundColor"] as? String {
    // 根据 customOptions 自定义视图
    nativeAdView.backgroundColor = UIColor(hexString: backgroundColor)
  }
  
  // ... 其他实现
}
```

## 错误处理

### 安全的类型转换

使用可选绑定来安全地转换视图类型：

```swift
if let headlineLabel = nativeAdView.headlineView as? UILabel {
  headlineLabel.text = nativeAd.headline
}
```

### 处理 XIB 加载失败

```swift
guard let nativeAdView = Bundle.main.loadNibNamed(
    "NativeAdView",
    owner: nil,
    options: nil
)?.first as? NativeAdView else {
  print("Failed to load NativeAdView from XIB")
  return nil
}
```

## 实践练习

1. 创建 `NativeAdFactoryExample` 类
2. 在 `AppDelegate` 中注册工厂
3. 实现 `createNativeAd()` 方法
4. 从 XIB 加载视图
5. 填充必需字段和可选字段
6. 处理星级评分
7. 设置 `nativeAd` 属性

## 常见问题

### Q: 为什么需要两次设置 `nativeAd` 属性？

A: 第一次设置是为了初始化关联，第二次设置（在填充完所有视图后）是为了确保 SDK 能正确处理所有视图的点击事件。

### Q: 如果 XIB 文件不存在会怎样？

A: `loadNibNamed()` 会返回 `nil`，导致应用崩溃。应该使用可选绑定来安全处理。

### Q: 可以创建多个工厂吗？

A: 可以。每个工厂使用不同的 `factoryId`，可以在 Flutter 层选择使用哪个工厂。

### Q: 如何处理视图类型转换错误？

A: 使用可选绑定（`as?`）而不是强制转换（`as!`），并检查结果是否为 `nil`。

## 总结与检查清单

### 本章要点

- 实现 `FLTNativeAdFactory` 协议创建原生广告视图
- 在 `AppDelegate` 中注册工厂，使用唯一的 `factoryId`
- 从 XIB 文件加载 `NativeAdView`
- 设置视图引用和填充广告数据
- 处理可选字段和星级评分
- 必须设置 `nativeAd` 属性并禁用 `callToActionView` 的用户交互

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `FLTNativeAdFactory` 协议的使用
- [ ] 如何在 `AppDelegate` 中注册工厂
- [ ] 如何从 XIB 加载视图
- [ ] 如何填充必需字段和可选字段
- [ ] 如何处理星级评分
- [ ] 为什么需要设置 `nativeAd` 属性

下一章，我们将学习如何设计 iOS 原生广告的 XIB 布局文件。
