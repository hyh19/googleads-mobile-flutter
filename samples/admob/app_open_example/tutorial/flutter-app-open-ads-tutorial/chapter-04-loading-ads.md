# 第 4 章：加载 App Open 广告

## 章节简介

在本章中，我们将实现 `loadAd()` 方法，学习如何使用 Google Mobile Ads SDK 从广告服务器加载 App Open 广告。我们将详细讲解 `AppOpenAd.load()` 方法、回调处理、成功和失败场景，以及错误处理机制。

## 广告加载流程

加载 App Open 广告的基本流程如下：

1. **调用加载方法**：使用 `AppOpenAd.load()` 方法请求广告
2. **等待响应**：Google 广告服务器处理请求并返回广告内容
3. **处理回调**：根据加载结果执行相应的回调函数
4. **保存广告对象**：如果加载成功，保存广告对象以供后续使用

## 实现 loadAd() 方法

### 基础实现

让我们更新 `AppOpenAdManager` 类中的 `loadAd()` 方法：

```dart
/// 加载一个 App Open 广告
void loadAd() {
  AppOpenAd.load(
    adUnitId: adUnitId,
    request: AdRequest(),
    adLoadCallback: AppOpenAdLoadCallback(
      onAdLoaded: (ad) {
        debugPrint('App Open Ad loaded: $ad');
        _appOpenAd = ad;
      },
      onAdFailedToLoad: (error) {
        debugPrint('App Open Ad failed to load: $error');
        // 处理加载失败的情况
      },
    ),
  );
}
```

### 代码详解

让我们逐行分析这个实现：

#### AppOpenAd.load() 方法

`AppOpenAd.load()` 是 Google Mobile Ads SDK 提供的静态方法，用于加载 App Open 广告。它接受以下参数：

- `adUnitId`：广告单元 ID（我们之前定义的 `adUnitId` 属性）
- `request`：`AdRequest` 对象，包含广告请求的配置信息
- `adLoadCallback`：`AppOpenAdLoadCallback` 对象，处理加载结果

#### AdRequest 对象

`AdRequest()` 创建一个标准的广告请求。你可以根据需要添加额外的配置：

```dart
AdRequest(
  keywords: ['game', 'entertainment'],  // 关键词（可选）
  contentUrl: 'https://example.com',    // 内容 URL（可选）
  nonPersonalizedAds: false,            // 是否禁用个性化广告（可选）
)
```

对于 App Open Ads，通常使用默认的 `AdRequest()` 即可。

#### AppOpenAdLoadCallback

`AppOpenAdLoadCallback` 是一个回调类，包含两个回调函数：

- `onAdLoaded`：广告加载成功时调用，接收加载的 `AppOpenAd` 对象作为参数
- `onAdFailedToLoad`：广告加载失败时调用，接收 `LoadAdError` 对象作为参数

### 成功回调处理

当广告成功加载时，`onAdLoaded` 回调会被调用：

```dart
onAdLoaded: (ad) {
  debugPrint('App Open Ad loaded: $ad');
  _appOpenAd = ad;
}
```

**代码说明**：

- `ad` 参数是成功加载的 `AppOpenAd` 对象
- 使用 `debugPrint` 记录加载成功的日志（仅在调试模式下输出）
- 将加载的广告对象保存到 `_appOpenAd` 变量中

### 失败回调处理

当广告加载失败时，`onAdFailedToLoad` 回调会被调用：

```dart
onAdFailedToLoad: (error) {
  debugPrint('App Open Ad failed to load: $error');
  // 处理加载失败的情况
}
```

**代码说明**：

- `error` 参数是 `LoadAdError` 对象，包含错误信息
- 记录错误日志以便调试
- 可以根据错误类型采取不同的处理策略

## 错误处理

### LoadAdError 对象

`LoadAdError` 对象包含以下有用信息：

```dart
onAdFailedToLoad: (error) {
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
onAdFailedToLoad: (error) {
  debugPrint('App Open Ad failed to load: $error');
  
  // 根据错误代码处理
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
  
  // 清理状态
  _appOpenAd = null;
}
```

## 完整的 loadAd() 实现

以下是完整的 `loadAd()` 方法实现，包含详细的错误处理：

```dart
/// 加载一个 App Open 广告
void loadAd() {
  // 如果已经有广告在加载或已加载，不重复加载
  if (_appOpenAd != null) {
    debugPrint('Ad already loaded, skipping load request');
    return;
  }

  AppOpenAd.load(
    adUnitId: adUnitId,
    request: const AdRequest(),
    adLoadCallback: AppOpenAdLoadCallback(
      onAdLoaded: (ad) {
        debugPrint('App Open Ad loaded successfully: $ad');
        _appOpenAd = ad;
      },
      onAdFailedToLoad: (error) {
        debugPrint('App Open Ad failed to load: $error');
        debugPrint('Error code: ${error.code}');
        debugPrint('Error message: ${error.message}');
        
        // 清理状态
        _appOpenAd = null;
        
        // 可以根据需要实现重试逻辑
        // 例如：延迟后重新加载
      },
    ),
  );
}
```

## 异步加载考虑

虽然 `AppOpenAd.load()` 本身是异步的，但它是基于回调的，不需要使用 `async/await`。不过，如果你需要在加载完成后执行某些操作，可以考虑使用 `Future`：

```dart
Future<void> loadAd() async {
  if (_appOpenAd != null) {
    return;
  }

  final completer = Completer<void>();
  
  AppOpenAd.load(
    adUnitId: adUnitId,
    request: const AdRequest(),
    adLoadCallback: AppOpenAdLoadCallback(
      onAdLoaded: (ad) {
        _appOpenAd = ad;
        completer.complete();
      },
      onAdFailedToLoad: (error) {
        debugPrint('Failed to load: $error');
        _appOpenAd = null;
        completer.completeError(error);
      },
    ),
  );
  
  return completer.future;
}
```

对于大多数情况，基于回调的实现已经足够。

## 测试广告加载

### 在应用中测试

你可以在应用的初始化代码中调用 `loadAd()` 来测试：

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 创建管理器并加载广告
    final manager = AppOpenAdManager();
    manager.loadAd();
    
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Test Ad Loading')),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ElevatedButton(
                onPressed: () {
                  manager.loadAd();
                },
                child: Text('Load Ad'),
              ),
              SizedBox(height: 20),
              Text('Ad Available: ${manager.isAdAvailable}'),
            ],
          ),
        ),
      ),
    );
  }
}
```

### 检查日志

运行应用后，查看控制台输出。如果广告加载成功，你应该看到：

```text
App Open Ad loaded successfully: Instance of 'AppOpenAd'
```

如果加载失败，你会看到错误信息：

```text
App Open Ad failed to load: LoadAdError(...)
Error code: 3
Error message: No ad available
```

## 常见问题

### 为什么广告加载失败？

可能的原因包括：

1. **网络问题**：设备没有网络连接或网络不稳定
2. **无广告填充**：当前没有可用的广告（错误代码 3）
3. **配置错误**：广告单元 ID 不正确或 AdMob App ID 未正确配置
4. **测试模式**：确保使用测试广告单元 ID 进行测试

### 如何确保广告在显示前已加载？

App Open Ads 的设计理念是预加载。你应该在应用启动时或广告关闭后立即加载下一个广告，这样当需要显示时，广告已经准备好了。

### 加载需要多长时间？

广告加载通常需要 1-3 秒，但可能因网络状况而异。这就是为什么需要预加载的原因。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现 loadAd() 方法**：在 `AppOpenAdManager` 中实现完整的 `loadAd()` 方法
2. **添加错误处理**：实现详细的错误处理和日志记录
3. **测试加载功能**：创建一个简单的测试应用来验证广告加载
4. **检查日志**：运行应用并检查控制台输出，确认广告加载成功或失败

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解广告加载的基本流程
- [ ] 实现 `loadAd()` 方法使用 `AppOpenAd.load()`
- [ ] 处理 `onAdLoaded` 成功回调
- [ ] 处理 `onAdFailedToLoad` 失败回调
- [ ] 理解 `LoadAdError` 对象和常见错误代码
- [ ] 实现基本的错误处理和日志记录
- [ ] 测试广告加载功能

## 下一步

现在我们已经实现了广告加载功能，在下一章中，我们将学习如何展示已加载的广告，并处理展示过程中的各种回调事件。

继续学习：[第 5 章：展示广告与回调处理](chapter-05-showing-ads.md)
