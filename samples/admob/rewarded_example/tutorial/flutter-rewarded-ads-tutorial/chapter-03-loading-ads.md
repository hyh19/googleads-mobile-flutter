# 第 3 章：加载激励广告

## 章节简介

在本章中，我们将学习如何使用 Google Mobile Ads SDK 从广告服务器加载 Rewarded 广告。我们将详细讲解 `RewardedAd.load()` 方法、`RewardedAdLoadCallback` 回调处理、成功和失败场景，以及错误处理机制。

## 广告加载流程

加载 Rewarded 广告的基本流程如下：

1. **调用加载方法**：使用 `RewardedAd.load()` 方法请求广告
2. **等待响应**：Google 广告服务器处理请求并返回广告内容
3. **处理回调**：根据加载结果执行相应的回调函数
4. **保存广告对象**：如果加载成功，保存广告对象以供后续使用

## 实现 _loadAd() 方法

### 基础实现

让我们创建一个加载广告的方法：

```dart
RewardedAd? _rewardedAd;

void _loadAd() async {
  RewardedAd.load(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    rewardedAdLoadCallback: RewardedAdLoadCallback(
      onAdLoaded: (RewardedAd ad) {
        debugPrint('Ad was loaded.');
        _rewardedAd = ad;
      },
      onAdFailedToLoad: (LoadAdError error) {
        debugPrint('RewardedAd failed to load: $error');
      },
    ),
  );
}
```

### 代码详解

让我们逐行分析这个实现：

#### RewardedAd.load() 方法

`RewardedAd.load()` 是 Google Mobile Ads SDK 提供的静态方法，用于加载 Rewarded 广告。它接受以下参数：

- `adUnitId`：广告单元 ID（字符串）
- `request`：`AdRequest` 对象，包含广告请求的配置信息
- `rewardedAdLoadCallback`：`RewardedAdLoadCallback` 对象，处理加载结果

#### AdRequest 对象

`AdRequest()` 创建一个标准的广告请求。你可以根据需要添加额外的配置：

```dart
AdRequest(
  keywords: ['game', 'entertainment'],  // 关键词（可选）
  contentUrl: 'https://example.com',    // 内容 URL（可选）
  nonPersonalizedAds: false,            // 是否禁用个性化广告（可选）
)
```

对于 Rewarded Ads，通常使用默认的 `AdRequest()` 即可。

#### RewardedAdLoadCallback

`RewardedAdLoadCallback` 是一个回调类，包含两个回调函数：

- `onAdLoaded`：广告加载成功时调用，接收加载的 `RewardedAd` 对象作为参数
- `onAdFailedToLoad`：广告加载失败时调用，接收 `LoadAdError` 对象作为参数

### 成功回调处理

当广告成功加载时，`onAdLoaded` 回调会被调用：

```dart
onAdLoaded: (RewardedAd ad) {
  debugPrint('Ad was loaded.');
  _rewardedAd = ad;
}
```

**代码说明**：

- `ad` 参数是成功加载的 `RewardedAd` 对象
- 使用 `debugPrint` 记录加载成功的日志（仅在调试模式下输出）
- 将加载的广告对象保存到 `_rewardedAd` 变量中

### 失败回调处理

当广告加载失败时，`onAdFailedToLoad` 回调会被调用：

```dart
onAdFailedToLoad: (LoadAdError error) {
  debugPrint('RewardedAd failed to load: $error');
}
```

**代码说明**：

- `error` 参数是 `LoadAdError` 对象，包含错误信息
- 记录错误日志以便调试

## 集成用户同意检查

在实际应用中，应该在加载广告前检查用户是否同意：

```dart
final _consentManager = ConsentManager();

void _loadAd() async {
  // 检查是否可以请求广告
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  RewardedAd.load(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    rewardedAdLoadCallback: RewardedAdLoadCallback(
      onAdLoaded: (RewardedAd ad) {
        debugPrint('Ad was loaded.');
        _rewardedAd = ad;
      },
      onAdFailedToLoad: (LoadAdError error) {
        debugPrint('RewardedAd failed to load: $error');
      },
    ),
  );
}
```

**代码说明**：

- 在加载广告前，先检查是否可以请求广告
- 如果用户未同意，不加载广告

## 错误处理

### LoadAdError 对象

`LoadAdError` 对象包含以下有用信息：

```dart
onAdFailedToLoad: (LoadAdError error) {
  debugPrint('Error code: ${error.code}');
  debugPrint('Error domain: ${error.domain}');
  debugPrint('Error message: ${error.message}');
  debugPrint('Response info: ${error.responseInfo}');
}
```

### 常见错误代码

- `0`：`ERROR_CODE_INTERNAL_ERROR` - 内部错误
- `1`：`ERROR_CODE_INVALID_REQUEST` - 无效请求
- `2`：`ERROR_CODE_NETWORK_ERROR` - 网络错误
- `3`：`ERROR_CODE_NO_FILL` - 无广告填充

### 改进的错误处理

我们可以根据错误类型采取不同的处理策略：

```dart
onAdFailedToLoad: (LoadAdError error) {
  debugPrint('RewardedAd failed to load: $error');
  
  switch (error.code) {
    case 0: // 内部错误
      debugPrint('Internal error occurred');
      break;
    case 1: // 无效请求
      debugPrint('Invalid ad request');
      break;
    case 2: // 网络错误
      debugPrint('Network error - will retry later');
      // 可以在这里实现重试逻辑
      break;
    case 3: // 无广告填充
      debugPrint('No ad available');
      break;
    default:
      debugPrint('Unknown error: ${error.code}');
  }
}
```

## 完整的 _loadAd() 实现

以下是完整的 `_loadAd()` 方法实现，包含详细的错误处理和同意检查：

```dart
RewardedAd? _rewardedAd;
final _consentManager = ConsentManager();

void _loadAd() async {
  // 检查是否可以请求广告
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  RewardedAd.load(
    adUnitId: _adUnitId,
    request: const AdRequest(),
    rewardedAdLoadCallback: RewardedAdLoadCallback(
      onAdLoaded: (RewardedAd ad) {
        debugPrint('Ad was loaded.');
        // 保存广告对象以便后续使用
        _rewardedAd = ad;
      },
      onAdFailedToLoad: (LoadAdError error) {
        debugPrint('RewardedAd failed to load: $error');
        debugPrint('Error code: ${error.code}');
        debugPrint('Error message: ${error.message}');
      },
    ),
  );
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
RewardedAd failed to load: LoadAdError(...)
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

在显示广告之前，检查 `_rewardedAd` 是否为 `null`：

```dart
if (_rewardedAd != null) {
  _rewardedAd!.show(...);
} else {
  debugPrint('Ad not loaded yet');
}
```

### 加载需要多长时间？

广告加载通常需要 1-3 秒，但可能因网络状况而异。这就是为什么需要预加载的原因。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现 _loadAd() 方法**：在应用中实现完整的 `_loadAd()` 方法
2. **添加错误处理**：实现详细的错误处理和日志记录
3. **集成同意检查**：在加载广告前检查用户同意
4. **测试加载功能**：运行应用并检查控制台输出

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解广告加载的基本流程
- [ ] 实现 `_loadAd()` 方法使用 `RewardedAd.load()`
- [ ] 处理 `onAdLoaded` 成功回调
- [ ] 处理 `onAdFailedToLoad` 失败回调
- [ ] 理解 `LoadAdError` 对象和常见错误代码
- [ ] 实现基本的错误处理和日志记录
- [ ] 集成用户同意检查
- [ ] 测试广告加载功能

## 下一步

现在我们已经实现了广告加载功能，在下一章中，我们将学习如何显示已加载的广告，并处理用户获得的奖励。

继续学习：[第 4 章：显示广告和处理奖励](chapter-04-showing-ads.md)
