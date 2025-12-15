# 第 4 章：加载横幅广告

## 章节简介

在本章中，我们将学习如何加载 Banner 广告。我们将详细讲解 `BannerAd` 类的使用、`BannerAdListener` 回调处理、成功和失败场景的处理，以及如何集成用户同意检查。

## BannerAd 类概述

### BannerAd 类

`BannerAd` 是 Google Mobile Ads SDK 提供的类，用于加载和显示横幅广告。它继承自 `Ad` 基类，提供了加载、显示和管理横幅广告的功能。

### 创建 BannerAd 对象

创建 `BannerAd` 对象需要以下参数：

- `adUnitId`：广告单元 ID
- `request`：`AdRequest` 对象
- `size`：广告尺寸（`AdSize` 对象）
- `listener`：`BannerAdListener` 对象，处理广告事件

## 实现加载方法

### 基础实现

让我们创建一个加载广告的方法：

```dart
BannerAd? _bannerAd;

void _loadAd() async {
  // 获取自适应广告尺寸
  final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
    MediaQuery.sizeOf(context).width.truncate(),
  );

  if (size == null) {
    debugPrint('Unable to get width of anchored banner.');
    return;
  }

  // 创建并加载 BannerAd
  BannerAd(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    size: size,
    listener: BannerAdListener(
      onAdLoaded: (ad) {
        debugPrint('Ad was loaded.');
        setState(() {
          _bannerAd = ad as BannerAd;
        });
      },
      onAdFailedToLoad: (ad, err) {
        debugPrint('Ad failed to load with error: $err');
        ad.dispose();
      },
    ),
  ).load();
}
```

### 代码详解

让我们逐行分析这个实现：

#### 1. 获取广告尺寸

```dart
final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
  MediaQuery.sizeOf(context).width.truncate(),
);
```

- 使用我们在第 3 章学到的方法获取自适应广告尺寸
- 必须等待异步操作完成

#### 2. 检查尺寸

```dart
if (size == null) {
  debugPrint('Unable to get width of anchored banner.');
  return;
}
```

- 如果无法获取尺寸，无法加载广告，直接返回

#### 3. 创建 BannerAd 对象

```dart
BannerAd(
  adUnitId: _adUnitId,
  request: const AdRequest(),
  size: size,
  listener: BannerAdListener(...),
)
```

**参数说明**：

- `adUnitId`：广告单元 ID（字符串）
- `request`：`AdRequest` 对象，包含广告请求配置
- `size`：广告尺寸（`AdSize` 对象）
- `listener`：`BannerAdListener` 对象，处理广告事件

#### 4. 调用 load() 方法

```dart
.load();
```

- 调用 `load()` 方法开始加载广告
- 这是一个异步操作，结果通过 `listener` 回调通知

## BannerAdListener 详解

### BannerAdListener 类

`BannerAdListener` 是一个回调类，用于处理 Banner 广告的各种事件。它包含多个可选的回调方法。

### 必需的回调

#### onAdLoaded

当广告成功加载时调用：

```dart
onAdLoaded: (ad) {
  debugPrint('Ad was loaded.');
  setState(() {
    _bannerAd = ad as BannerAd;
  });
}
```

**参数**：

- `ad`：成功加载的 `Ad` 对象，需要转换为 `BannerAd`

**处理**：

- 保存广告对象到状态变量
- 更新 UI（通过 `setState`）

#### onAdFailedToLoad

当广告加载失败时调用：

```dart
onAdFailedToLoad: (ad, err) {
  debugPrint('Ad failed to load with error: $err');
  ad.dispose();
}
```

**参数**：

- `ad`：失败的 `Ad` 对象
- `err`：`LoadAdError` 对象，包含错误信息

**处理**：

- 记录错误信息
- 释放广告对象（调用 `dispose()`）

### 完整的监听器实现

```dart
listener: BannerAdListener(
  onAdLoaded: (ad) {
    debugPrint('Ad was loaded.');
    setState(() {
      _bannerAd = ad as BannerAd;
    });
  },
  onAdFailedToLoad: (ad, err) {
    debugPrint('Ad failed to load with error: $err');
    debugPrint('Error code: ${err.code}');
    debugPrint('Error message: ${err.message}');
    ad.dispose();
  },
  // 其他可选回调将在第 7 章详细讲解
),
```

## 集成用户同意检查

在实际应用中，应该在加载广告前检查用户是否同意：

```dart
void _loadAd() async {
  // 检查是否可以请求广告
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  // 检查组件是否仍然挂载
  if (!mounted) {
    return;
  }

  // 获取广告尺寸
  final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
    MediaQuery.sizeOf(context).width.truncate(),
  );

  if (size == null) {
    debugPrint('Unable to get width of anchored banner.');
    return;
  }

  // 创建并加载广告
  BannerAd(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    size: size,
    listener: BannerAdListener(
      // ... 监听器实现
    ),
  ).load();
}
```

**代码说明**：

- `canRequestAds()`：检查用户是否同意（需要 `ConsentManager` 实例）
- `mounted`：检查 Widget 是否仍然挂载，避免在已销毁的 Widget 上更新状态

## 错误处理

### LoadAdError 对象

`LoadAdError` 对象包含以下有用信息：

```dart
onAdFailedToLoad: (ad, err) {
  debugPrint('Error code: ${err.code}');
  debugPrint('Error domain: ${err.domain}');
  debugPrint('Error message: ${err.message}');
  debugPrint('Response info: ${err.responseInfo}');
}
```

### 常见错误代码

- `0`：`ERROR_CODE_INTERNAL_ERROR` - 内部错误
- `1`：`ERROR_CODE_INVALID_REQUEST` - 无效请求
- `2`：`ERROR_CODE_NETWORK_ERROR` - 网络错误
- `3`：`ERROR_CODE_NO_FILL` - 无广告填充

### 改进的错误处理

```dart
onAdFailedToLoad: (ad, err) {
  debugPrint('Ad failed to load: $err');
  
  switch (err.code) {
    case 0: // 内部错误
      debugPrint('Internal error occurred');
      break;
    case 1: // 无效请求
      debugPrint('Invalid ad request');
      break;
    case 2: // 网络错误
      debugPrint('Network error - will retry later');
      // 可以实现重试逻辑
      break;
    case 3: // 无广告填充
      debugPrint('No ad available');
      break;
    default:
      debugPrint('Unknown error: ${err.code}');
  }
  
  ad.dispose();
}
```

## 完整的加载方法实现

以下是完整的 `_loadAd()` 方法实现：

```dart
BannerAd? _bannerAd;

void _loadAd() async {
  // 检查用户同意
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  // 检查组件是否挂载
  if (!mounted) {
    return;
  }

  // 获取自适应广告尺寸
  final size = await AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(
    MediaQuery.sizeOf(context).width.truncate(),
  );

  if (size == null) {
    debugPrint('Unable to get width of anchored banner.');
    return;
  }

  // 创建并加载广告
  BannerAd(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    size: size,
    listener: BannerAdListener(
      onAdLoaded: (ad) {
        debugPrint('Ad was loaded.');
        setState(() {
          _bannerAd = ad as BannerAd;
        });
      },
      onAdFailedToLoad: (ad, err) {
        debugPrint('Ad failed to load with error: $err');
        debugPrint('Error code: ${err.code}');
        debugPrint('Error message: ${err.message}');
        ad.dispose();
      },
    ),
  ).load();
}
```

## 测试广告加载

### 在应用中测试

你可以在应用的初始化代码中调用 `_loadAd()` 来测试：

```dart
@override
void initState() {
  super.initState();
  _loadAd();
}
```

### 检查日志

运行应用后，查看控制台输出。如果广告加载成功，你应该看到：

```text
Ad was loaded.
```

如果加载失败，你会看到错误信息：

```text
Ad failed to load with error: LoadAdError(...)
Error code: 3
Error message: No ad available
```

## 常见问题

### 为什么广告加载失败？

可能的原因包括：

1. **网络问题**：设备没有网络连接或网络不稳定
2. **无广告填充**：当前没有可用的广告（错误代码 3）
3. **配置错误**：广告单元 ID 不正确或 AdMob App ID 未正确配置
4. **用户未同意**：用户未同意显示广告

### 如何确保广告在显示前已加载？

在显示广告之前，检查 `_bannerAd` 是否为 `null`：

```dart
if (_bannerAd != null) {
  // 显示广告
}
```

### 加载需要多长时间？

广告加载通常需要 1-3 秒，但可能因网络状况而异。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现 _loadAd() 方法**：在应用中实现完整的 `_loadAd()` 方法
2. **添加错误处理**：实现详细的错误处理和日志记录
3. **集成同意检查**：在加载广告前检查用户同意
4. **测试加载功能**：运行应用并检查控制台输出

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解 `BannerAd` 类的作用
- [ ] 创建 `BannerAd` 对象
- [ ] 实现 `_loadAd()` 方法
- [ ] 理解 `BannerAdListener` 的回调
- [ ] 处理 `onAdLoaded` 成功回调
- [ ] 处理 `onAdFailedToLoad` 失败回调
- [ ] 集成用户同意检查
- [ ] 实现基本的错误处理

## 下一步

现在我们已经实现了广告加载功能，在下一章中，我们将学习如何显示已加载的广告。

继续学习：[第 5 章：显示横幅广告](chapter-05-displaying-ads.md)
