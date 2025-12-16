# 第 4 章 Android 布局设计

## 引言

Android 原生广告的布局使用 XML 文件定义。布局文件必须使用 `NativeAdView` 作为根视图，并包含所有需要的视图组件。本章将详细讲解如何设计原生广告的 XML 布局，包括必需视图、可选视图和布局最佳实践。

## 原生广告布局要求

### 根视图要求

布局文件的根视图必须是 `com.google.android.gms.ads.nativead.NativeAdView`：

```xml
<com.google.android.gms.ads.nativead.NativeAdView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <!-- 布局内容 -->
</com.google.android.gms.ads.nativead.NativeAdView>
```

### 必需视图

布局中必须包含以下视图：

- **headlineView**：用于显示标题的 `TextView`
- **mediaView**：用于显示媒体内容的 `MediaView`

### 可选视图

布局中可以包含以下可选视图：

- **bodyView**：正文 `TextView`
- **iconView**：图标 `ImageView`
- **callToActionView**：行动号召按钮（`Button` 或 `TextView`）
- **priceView**：价格 `TextView`
- **storeView**：商店名称 `TextView`
- **starRatingView**：星级评分 `RatingBar`
- **advertiserView**：广告主名称 `TextView`

## 创建 XML 布局文件

### 文件位置

布局文件应放在 `android/app/src/main/res/layout/` 目录下，例如 `my_native_ad.xml`。

### 基本结构

```xml
<com.google.android.gms.ads.nativead.NativeAdView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <!-- 布局内容 -->
    
</com.google.android.gms.ads.nativead.NativeAdView>
```

## 完整布局示例

以下是示例项目中的完整布局文件：

```xml
<com.google.android.gms.ads.nativead.NativeAdView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

  <LinearLayout
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:layout_gravity="center"
      android:background="#FFFFFF"
      android:minHeight="50dp"
      android:orientation="vertical">
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:paddingLeft="20dp"
        android:paddingRight="20dp"
        android:paddingTop="3dp">

      <!-- 图标和标题区域 -->
      <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:orientation="horizontal">

        <!-- 应用图标 -->
        <ImageView
            android:id="@+id/ad_app_icon"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:adjustViewBounds="true"
            android:paddingBottom="5dp"
            android:paddingEnd="5dp"
            android:paddingRight="5dp"/>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

          <!-- 标题 -->
          <TextView
              android:id="@+id/ad_headline"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:textColor="#0000FF"
              android:textSize="16sp"
              android:textStyle="bold" />

          <!-- 广告主和评分 -->
          <LinearLayout
              android:layout_width="match_parent"
              android:layout_height="wrap_content">

            <TextView
                android:id="@+id/ad_advertiser"
                android:layout_width="wrap_content"
                android:layout_height="match_parent"
                android:gravity="bottom"
                android:textSize="14sp"
                android:textStyle="bold"/>

            <RatingBar
                android:id="@+id/ad_stars"
                style="?android:attr/ratingBarStyleSmall"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:isIndicator="true"
                android:numStars="5"
                android:stepSize="0.5" />
          </LinearLayout>

        </LinearLayout>
      </LinearLayout>

      <!-- 正文和媒体内容区域 -->
      <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:orientation="vertical">

        <!-- 正文 -->
        <TextView
            android:id="@+id/ad_body"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginRight="20dp"
            android:layout_marginEnd="20dp"
            android:textSize="12sp" />

        <!-- 媒体视图（必需） -->
        <com.google.android.gms.ads.nativead.MediaView
            android:id="@+id/ad_media"
            android:layout_gravity="center_horizontal"
            android:layout_width="250dp"
            android:layout_height="175dp"
            android:layout_marginTop="5dp" />

        <!-- 价格、商店和行动号召按钮 -->
        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="end"
            android:orientation="horizontal"
            android:paddingBottom="10dp"
            android:paddingTop="10dp">

          <TextView
              android:id="@+id/ad_price"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:paddingLeft="5dp"
              android:paddingStart="5dp"
              android:paddingRight="5dp"
              android:paddingEnd="5dp"
              android:textSize="12sp" />

          <TextView
              android:id="@+id/ad_store"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:paddingLeft="5dp"
              android:paddingStart="5dp"
              android:paddingRight="5dp"
              android:paddingEnd="5dp"
              android:textSize="12sp" />

          <Button
              android:id="@+id/ad_call_to_action"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:gravity="center"
              android:textSize="12sp" />
        </LinearLayout>
      </LinearLayout>
    </LinearLayout>
  </LinearLayout>
</com.google.android.gms.ads.nativead.NativeAdView>
```

## 视图 ID 要求

### ID 命名

每个视图必须使用特定的 ID，这些 ID 在工厂代码中通过 `findViewById()` 引用：

- `@+id/ad_headline` - 标题
- `@+id/ad_media` - 媒体视图（必需）
- `@+id/ad_body` - 正文
- `@+id/ad_app_icon` - 图标
- `@+id/ad_call_to_action` - 行动号召按钮
- `@+id/ad_price` - 价格
- `@+id/ad_store` - 商店名称
- `@+id/ad_stars` - 星级评分
- `@+id/ad_advertiser` - 广告主名称

**注意**：ID 名称可以自定义，但必须与工厂代码中的 ID 匹配。

## MediaView 的特殊要求

### MediaView 是必需的

`MediaView` 是原生广告的必需组件，必须包含在布局中：

```xml
<com.google.android.gms.ads.nativead.MediaView
    android:id="@+id/ad_media"
    android:layout_width="250dp"
    android:layout_height="175dp" />
```

### MediaView 特性

- **显示图片或视频**：`MediaView` 可以显示静态图片或视频内容
- **自动处理**：SDK 会自动处理媒体内容的加载和显示
- **尺寸建议**：建议设置固定尺寸，避免布局问题

## 布局设计最佳实践

### 1. 响应式设计

使用 `match_parent` 和 `wrap_content` 来创建响应式布局：

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">
    <!-- 内容 -->
</LinearLayout>
```

### 2. 适当的间距

使用 padding 和 margin 来创建视觉层次：

```xml
<LinearLayout
    android:paddingLeft="20dp"
    android:paddingRight="20dp"
    android:paddingTop="3dp">
    <!-- 内容 -->
</LinearLayout>
```

### 3. 文本样式

为不同类型的文本设置合适的样式：

```xml
<!-- 标题：大号、粗体 -->
<TextView
    android:id="@+id/ad_headline"
    android:textSize="16sp"
    android:textStyle="bold" />

<!-- 正文：小号、普通 -->
<TextView
    android:id="@+id/ad_body"
    android:textSize="12sp" />
```

### 4. 颜色和主题

使用与应用主题一致的颜色：

```xml
<LinearLayout
    android:background="#FFFFFF">
    <!-- 内容 -->
</LinearLayout>
```

## 简化布局示例

如果你只需要基本的原生广告，可以使用这个简化布局：

```xml
<com.google.android.gms.ads.nativead.NativeAdView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

  <LinearLayout
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:orientation="vertical"
      android:padding="16dp">

    <!-- 标题（必需） -->
    <TextView
        android:id="@+id/ad_headline"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:textStyle="bold" />

    <!-- 媒体视图（必需） -->
    <com.google.android.gms.ads.nativead.MediaView
        android:id="@+id/ad_media"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:layout_marginTop="8dp" />

    <!-- 正文（可选） -->
    <TextView
        android:id="@+id/ad_body"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:textSize="14sp" />

    <!-- 行动号召按钮（可选） -->
    <Button
        android:id="@+id/ad_call_to_action"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="end"
        android:layout_marginTop="8dp" />
  </LinearLayout>
</com.google.android.gms.ads.nativead.NativeAdView>
```

## 实践练习

1. 创建 `my_native_ad.xml` 布局文件
2. 使用 `NativeAdView` 作为根视图
3. 添加必需的视图（headline、mediaView）
4. 添加可选视图（body、icon、callToAction 等）
5. 设置合适的样式和布局

## 常见问题

### Q: 我可以使用 ConstraintLayout 吗？

A: 可以。只要根视图是 `NativeAdView`，内部可以使用任何布局方式。

### Q: MediaView 的尺寸应该设置多少？

A: 建议根据你的应用设计设置合适的尺寸。常见尺寸包括 250x175dp、match_parent 等。

### Q: 如果我不包含某个可选视图会怎样？

A: 如果布局中没有某个视图，在工厂代码中不要设置对应的视图引用即可。

### Q: 可以自定义视图的样式吗？

A: 可以。你可以完全自定义视图的外观，只要保持 ID 正确即可。

## 总结与检查清单

### 本章要点

- 布局文件的根视图必须是 `NativeAdView`
- 必须包含 `headlineView` 和 `mediaView`
- 可选视图可以根据需要添加
- 视图 ID 必须与工厂代码中的 ID 匹配
- 遵循布局设计最佳实践

### 检查清单

在继续下一章之前，确保你已完成：

- [ ] 创建了 XML 布局文件
- [ ] 使用 `NativeAdView` 作为根视图
- [ ] 添加了必需的视图（headline、mediaView）
- [ ] 添加了需要的可选视图
- [ ] 设置了合适的样式和布局
- [ ] 视图 ID 与工厂代码匹配

下一章，我们将学习如何在 iOS 平台实现原生广告工厂。
