# 第 3 章 Android 原生工厂实现

## 引言

在 Android 平台，原生广告需要通过实现 `NativeAdFactory` 接口来创建原生广告视图。本章将详细讲解如何在 Android 中实现原生广告工厂，包括注册工厂、创建视图、填充数据和处理可选字段。

## NativeAdFactory 接口概述

### 接口定义

`NativeAdFactory` 是 Google Mobile Ads Flutter 插件提供的接口，用于创建原生广告视图：

```kotlin
interface NativeAdFactory {
    fun createNativeAd(
        nativeAd: NativeAd?,
        customOptions: MutableMap<String, Any>?
    ): NativeAdView
}
```

### 接口方法

- `createNativeAd()`：创建并返回一个 `NativeAdView`，该视图包含填充了广告数据的 UI 组件

## 在 MainActivity 中注册工厂

### 注册工厂

在 `MainActivity` 的 `configureFlutterEngine()` 方法中注册工厂：

```kotlin
class MainActivity: FlutterActivity() {
  override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    flutterEngine.plugins.add(GoogleMobileAdsPlugin())
    super.configureFlutterEngine(flutterEngine)
    
    GoogleMobileAdsPlugin.registerNativeAdFactory(
        flutterEngine,
        "adFactoryExample",  // factoryId，必须与 Flutter 层匹配
        NativeAdFactoryExample(layoutInflater))
  }
}
```

### factoryId 的重要性

`factoryId` 是一个字符串标识符，用于将 Flutter 层的广告请求与原生层的工厂关联起来。**重要**：`factoryId` 必须在 Flutter 层和原生层完全匹配。

### 注销工厂

在 `cleanUpFlutterEngine()` 方法中注销工厂：

```kotlin
override fun cleanUpFlutterEngine(flutterEngine: FlutterEngine) {
  GoogleMobileAdsPlugin.unregisterNativeAdFactory(
      flutterEngine, 
      "adFactoryExample")
}
```

## 实现 NativeAdFactoryExample 类

### 类结构

```kotlin
class NativeAdFactoryExample: NativeAdFactory {
  private var layoutInflater: LayoutInflater

  constructor(layoutInflater: LayoutInflater) {
    this.layoutInflater = layoutInflater
  }

  override fun createNativeAd(
      nativeAd: NativeAd?,
      customOptions: MutableMap<String, Any>?
  ): NativeAdView {
    // 实现创建逻辑
  }
}
```

### 创建 NativeAdView

首先，从 XML 布局文件创建 `NativeAdView`：

```kotlin
override fun createNativeAd(
    nativeAd: NativeAd?,
    customOptions: MutableMap<String, Any>?
): NativeAdView {
  val adView = layoutInflater.inflate(
      R.layout.my_native_ad, 
      null
  ) as NativeAdView
  
  // 设置视图引用和填充数据
  // ...
  
  return adView
}
```

## 设置视图引用

### 必需的视图引用

必须设置 `mediaView`，因为它是原生广告的必需组件：

```kotlin
adView.mediaView = adView.findViewById(R.id.ad_media)
```

### 其他视图引用

设置所有需要的视图引用：

```kotlin
// 设置媒体视图（必需）
adView.mediaView = adView.findViewById(R.id.ad_media)

// 设置其他视图引用
adView.headlineView = adView.findViewById(R.id.ad_headline)
adView.bodyView = adView.findViewById(R.id.ad_body)
adView.callToActionView = adView.findViewById(R.id.ad_call_to_action)
adView.iconView = adView.findViewById(R.id.ad_app_icon)
adView.priceView = adView.findViewById(R.id.ad_price)
adView.starRatingView = adView.findViewById(R.id.ad_stars)
adView.storeView = adView.findViewById(R.id.ad_store)
adView.advertiserView = adView.findViewById(R.id.ad_advertiser)
```

## 填充广告数据

### 必需字段

`headline` 和 `mediaContent` 是保证存在的字段：

```kotlin
// headline 和 mediaContent 保证存在
(adView.headlineView as TextView).text = nativeAd?.headline
adView.mediaView?.mediaContent = nativeAd?.mediaContent
```

### 可选字段处理

其他字段可能为 `null`，需要检查后再设置：

```kotlin
// body（可选）
if (nativeAd?.body == null) {
  adView.bodyView?.visibility = View.INVISIBLE
} else {
  adView.bodyView?.visibility = View.VISIBLE
  (adView.bodyView as TextView).text = nativeAd.body
}

// callToAction（可选）
if (nativeAd?.callToAction == null) {
  adView.callToActionView?.visibility = View.INVISIBLE
} else {
  adView.callToActionView?.visibility = View.VISIBLE
  (adView.callToActionView as Button).text = nativeAd.callToAction
}

// icon（可选）
if (nativeAd?.icon == null) {
  adView.iconView?.visibility = View.GONE
} else {
  (adView.iconView as ImageView).setImageDrawable(nativeAd.icon!!.drawable)
  adView.iconView?.visibility = View.VISIBLE
}

// price（可选）
if (nativeAd?.price == null) {
  adView.priceView?.visibility = View.INVISIBLE
} else {
  adView.priceView?.visibility = View.VISIBLE
  (adView.priceView as TextView).text = nativeAd.price
}

// store（可选）
if (nativeAd?.store == null) {
  adView.storeView?.visibility = View.INVISIBLE
} else {
  adView.storeView?.visibility = View.VISIBLE
  (adView.storeView as TextView).text = nativeAd.store
}

// starRating（可选）
if (nativeAd?.starRating == null) {
  adView.starRatingView?.visibility = View.INVISIBLE
} else {
  (adView.starRatingView as RatingBar).rating = nativeAd.starRating!!.toFloat()
  adView.starRatingView?.visibility = View.VISIBLE
}

// advertiser（可选）
if (nativeAd?.advertiser == null) {
  adView.advertiserView?.visibility = View.INVISIBLE
} else {
  adView.advertiserView?.visibility = View.VISIBLE
  (adView.advertiserView as TextView).text = nativeAd.advertiser
}
```

## 调用 setNativeAd() 方法

### 重要性

在填充完所有数据后，必须调用 `setNativeAd()` 方法。这个方法告诉 Google Mobile Ads SDK 你已经完成了视图的填充，SDK 会处理点击事件等交互。

```kotlin
// 必须在填充完所有数据后调用
if (nativeAd != null) {
  adView.setNativeAd(nativeAd)
}
```

### 调用时机

`setNativeAd()` 应该在以下操作之后调用：

1. 设置所有视图引用
2. 填充所有广告数据
3. 处理所有可选字段的可见性

## 完整的实现示例

以下是完整的 `NativeAdFactoryExample` 实现：

```kotlin
class NativeAdFactoryExample: NativeAdFactory {
  private var layoutInflater: LayoutInflater

  constructor(layoutInflater: LayoutInflater) {
    this.layoutInflater = layoutInflater
  }

  override fun createNativeAd(
      nativeAd: NativeAd?,
      customOptions: MutableMap<String, Any>?
  ): NativeAdView {
    val adView = layoutInflater.inflate(
        R.layout.my_native_ad, 
        null
    ) as NativeAdView

    // 设置媒体视图（必需）
    adView.mediaView = adView.findViewById(R.id.ad_media)

    // 设置其他视图引用
    adView.headlineView = adView.findViewById(R.id.ad_headline)
    adView.bodyView = adView.findViewById(R.id.ad_body)
    adView.callToActionView = adView.findViewById(R.id.ad_call_to_action)
    adView.iconView = adView.findViewById(R.id.ad_app_icon)
    adView.priceView = adView.findViewById(R.id.ad_price)
    adView.starRatingView = adView.findViewById(R.id.ad_stars)
    adView.storeView = adView.findViewById(R.id.ad_store)
    adView.advertiserView = adView.findViewById(R.id.ad_advertiser)

    // 填充必需字段
    (adView.headlineView as TextView).text = nativeAd?.headline
    adView.mediaView?.mediaContent = nativeAd?.mediaContent

    // 处理可选字段
    if (nativeAd?.body == null) {
      adView.bodyView?.visibility = View.INVISIBLE
    } else {
      adView.bodyView?.visibility = View.VISIBLE
      (adView.bodyView as TextView).text = nativeAd.body
    }

    if (nativeAd?.callToAction == null) {
      adView.callToActionView?.visibility = View.INVISIBLE
    } else {
      adView.callToActionView?.visibility = View.VISIBLE
      (adView.callToActionView as Button).text = nativeAd.callToAction
    }

    if (nativeAd?.icon == null) {
      adView.iconView?.visibility = View.GONE
    } else {
      (adView.iconView as ImageView).setImageDrawable(nativeAd.icon!!.drawable)
      adView.iconView?.visibility = View.VISIBLE
    }

    if (nativeAd?.price == null) {
      adView.priceView?.visibility = View.INVISIBLE
    } else {
      adView.priceView?.visibility = View.VISIBLE
      (adView.priceView as TextView).text = nativeAd.price
    }

    if (nativeAd?.store == null) {
      adView.storeView?.visibility = View.INVISIBLE
    } else {
      adView.storeView?.visibility = View.VISIBLE
      (adView.storeView as TextView).text = nativeAd.store
    }

    if (nativeAd?.starRating == null) {
      adView.starRatingView?.visibility = View.INVISIBLE
    } else {
      (adView.starRatingView as RatingBar).rating = nativeAd.starRating!!.toFloat()
      adView.starRatingView?.visibility = View.VISIBLE
    }

    if (nativeAd?.advertiser == null) {
      adView.advertiserView?.visibility = View.INVISIBLE
    } else {
      adView.advertiserView?.visibility = View.VISIBLE
      (adView.advertiserView as TextView).text = nativeAd.advertiser
    }

    // 重要：调用 setNativeAd() 完成设置
    if (nativeAd != null) {
      adView.setNativeAd(nativeAd)
    }

    return adView
  }
}
```

## MainActivity 完整实现

```kotlin
package com.example.native_platform_example

import android.view.LayoutInflater
import android.view.View
import android.widget.Button
import android.widget.ImageView
import android.widget.RatingBar
import android.widget.TextView
import com.google.android.gms.ads.nativead.NativeAd
import com.google.android.gms.ads.nativead.NativeAdView
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugins.googlemobileads.GoogleMobileAdsPlugin
import io.flutter.plugins.googlemobileads.GoogleMobileAdsPlugin.NativeAdFactory

class MainActivity: FlutterActivity() {
  override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    flutterEngine.plugins.add(GoogleMobileAdsPlugin())
    super.configureFlutterEngine(flutterEngine)
    
    GoogleMobileAdsPlugin.registerNativeAdFactory(
        flutterEngine,
        "adFactoryExample",
        NativeAdFactoryExample(layoutInflater))
  }

  override fun cleanUpFlutterEngine(flutterEngine: FlutterEngine) {
    GoogleMobileAdsPlugin.unregisterNativeAdFactory(
        flutterEngine, 
        "adFactoryExample")
  }
}

// NativeAdFactoryExample 实现（见上文）
```

## 处理 customOptions

`customOptions` 参数允许 Flutter 层传递自定义选项到原生层：

```kotlin
override fun createNativeAd(
    nativeAd: NativeAd?,
    customOptions: MutableMap<String, Any>?
): NativeAdView {
  // 使用 customOptions
  val backgroundColor = customOptions?.get("backgroundColor") as? String
  if (backgroundColor != null) {
    // 根据 customOptions 自定义视图
  }
  
  // ... 其他实现
}
```

## 实践练习

1. 创建 `NativeAdFactoryExample` 类
2. 在 `MainActivity` 中注册工厂
3. 实现 `createNativeAd()` 方法
4. 处理必需字段和可选字段
5. 调用 `setNativeAd()` 方法

## 常见问题

### Q: 如果我不调用 `setNativeAd()` 会怎样？

A: 广告可能无法正确响应点击事件，SDK 无法跟踪广告交互。

### Q: 可以创建多个工厂吗？

A: 可以。每个工厂使用不同的 `factoryId`，可以在 Flutter 层选择使用哪个工厂。

### Q: 如果 `nativeAd` 为 null 怎么办？

A: 应该返回一个空的或默认的 `NativeAdView`，或者抛出异常。但通常 SDK 不会传递 null。

### Q: 如何处理视图类型转换错误？

A: 确保 XML 布局中的视图类型与代码中的类型匹配。使用安全的类型转换。

## 总结与检查清单

### 本章要点

- 实现 `NativeAdFactory` 接口创建原生广告视图
- 在 `MainActivity` 中注册工厂，使用唯一的 `factoryId`
- 设置所有视图引用，包括必需的 `mediaView`
- 填充必需字段，正确处理可选字段
- 必须调用 `setNativeAd()` 方法完成设置

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `NativeAdFactory` 接口的使用
- [ ] 如何在 `MainActivity` 中注册工厂
- [ ] 如何创建和配置 `NativeAdView`
- [ ] 如何处理必需字段和可选字段
- [ ] 为什么必须调用 `setNativeAd()` 方法

下一章，我们将学习如何设计 Android 原生广告的 XML 布局文件。
