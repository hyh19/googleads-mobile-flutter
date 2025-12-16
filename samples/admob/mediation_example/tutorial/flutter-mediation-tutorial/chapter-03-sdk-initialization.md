# 第 3 章 SDK 初始化与适配器状态

## 引言

在使用广告中介时，正确初始化 Mobile Ads SDK 并检查适配器状态非常重要。本章将详细讲解如何初始化 SDK、理解初始化状态、检查适配器状态，以及为什么等待初始化完成很重要。

## SDK 初始化

### initialize() 方法

`MobileAds.instance.initialize()` 方法用于初始化 Mobile Ads SDK。在使用广告中介时，这个方法不仅初始化 Google Mobile Ads SDK，还会初始化所有已添加的中介适配器。

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  
  // 初始化 SDK
  MobileAds.instance.initialize().then((initializationStatus) {
    // 处理初始化结果
  });
  
  runApp(MyApp());
}
```

### 初始化回调

`initialize()` 方法返回一个 `Future<InitializationStatus>`，你可以通过 `.then()` 或 `await` 来处理初始化结果：

```dart
MobileAds.instance.initialize().then((initializationStatus) {
  // 检查每个适配器的状态
  initializationStatus.adapterStatuses.forEach((key, value) {
    debugPrint('Adapter status for $key: ${value.description}');
  });
});
```

## InitializationStatus 对象

`InitializationStatus` 对象包含以下信息：

- `adapterStatuses`：一个 `Map<String, AdapterStatus>`，包含所有适配器的状态
- `description`：初始化状态的描述

### 获取适配器状态

```dart
MobileAds.instance.initialize().then((initializationStatus) {
  initializationStatus.adapterStatuses.forEach((key, value) {
    debugPrint('Adapter: $key');
    debugPrint('State: ${value.description}');
    debugPrint('Latency: ${value.latency}');
  });
});
```

## AdapterStatus 对象

`AdapterStatus` 对象表示单个适配器的状态，包含以下信息：

- `description`：状态的文本描述
- `latency`：初始化延迟（毫秒）
- `initializationState`：初始化状态枚举

### 初始化状态枚举

`InitializationState` 有以下值：

- `READY`：适配器已准备好，可以请求广告
- `NOT_READY`：适配器未准备好，可能正在初始化或初始化失败
- `MISSING`：适配器库未添加或未找到

## 检查适配器状态

### 基本检查

```dart
MobileAds.instance.initialize().then((initializationStatus) {
  initializationStatus.adapterStatuses.forEach((key, value) {
    if (value.initializationState == AdapterInitializationState.READY) {
      debugPrint('$key is ready');
    } else if (value.initializationState == AdapterInitializationState.NOT_READY) {
      debugPrint('$key is not ready: ${value.description}');
    } else if (value.initializationState == AdapterInitializationState.MISSING) {
      debugPrint('$key is missing');
    }
  });
});
```

### 完整示例

以下是示例项目中的完整初始化代码：

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  
  // 在发起广告请求之前初始化 SDK
  // 你可以在回调中检查每个适配器的初始化状态
  MobileAds.instance.initialize().then((initializationStatus) {
    initializationStatus.adapterStatuses.forEach((key, value) {
      debugPrint('Adapter status for $key: ${value.description}');
    });
  });
  
  runApp(MyApp());
}
```

## 等待初始化完成的重要性

### 为什么需要等待？

在使用广告中介时，等待所有适配器初始化完成非常重要，原因包括：

1. **确保所有网络参与**：只有在适配器初始化完成后，对应的广告网络才能参与广告请求
2. **避免丢失广告机会**：如果适配器未初始化就请求广告，该网络将无法提供广告
3. **提高填充率**：所有网络都准备好后，广告填充率会显著提高

### 等待初始化完成

```dart
Future<void> _initializeAds() async {
  final initializationStatus = await MobileAds.instance.initialize();
  
  // 检查所有适配器是否已准备好
  bool allReady = true;
  initializationStatus.adapterStatuses.forEach((key, value) {
    if (value.initializationState != AdapterInitializationState.READY) {
      allReady = false;
      debugPrint('$key is not ready: ${value.description}');
    }
  });
  
  if (allReady) {
    debugPrint('All adapters are ready. You can now load ads.');
    // 开始加载广告
    _loadAd();
  } else {
    debugPrint('Some adapters are not ready. Waiting...');
    // 可以选择等待或继续
  }
}
```

### 处理初始化失败

如果某些适配器初始化失败，你应该：

1. **记录错误信息**：使用 `description` 字段了解失败原因
2. **继续使用其他网络**：即使某些适配器失败，其他已初始化的网络仍然可以使用
3. **重试机制**：可以考虑实现重试机制

```dart
Future<void> _initializeAds() async {
  final initializationStatus = await MobileAds.instance.initialize();
  
  initializationStatus.adapterStatuses.forEach((key, value) {
    if (value.initializationState == AdapterInitializationState.NOT_READY) {
      debugPrint('Warning: $key failed to initialize: ${value.description}');
      // 可以在这里实现重试逻辑或通知用户
    } else if (value.initializationState == AdapterInitializationState.MISSING) {
      debugPrint('Warning: $key adapter is missing. Did you add the dependency?');
    }
  });
  
  // 即使某些适配器失败，也可以继续使用其他网络
  _loadAd();
}
```

## 适配器状态监控

### 实时监控

你可以在应用运行时监控适配器状态：

```dart
void _monitorAdapterStatus() {
  MobileAds.instance.initialize().then((initializationStatus) {
    initializationStatus.adapterStatuses.forEach((key, value) {
      debugPrint('=== Adapter Status ===');
      debugPrint('Name: $key');
      debugPrint('State: ${value.initializationState}');
      debugPrint('Description: ${value.description}');
      debugPrint('Latency: ${value.latency}ms');
      debugPrint('====================');
    });
  });
}
```

### 状态变化处理

虽然适配器状态通常在初始化后不会改变，但在某些情况下（如网络变化、SDK 更新），你可能需要重新检查状态。

## 完整初始化流程

以下是完整的 SDK 初始化流程示例：

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Mediation Example',
      home: MyHomePage(),
    );
  }
}

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  
  // 初始化 SDK
  _initializeMobileAds();
  
  runApp(MyApp());
}

Future<void> _initializeMobileAds() async {
  try {
    final initializationStatus = await MobileAds.instance.initialize();
    
    // 打印所有适配器状态
    debugPrint('=== Mobile Ads SDK Initialization ===');
    initializationStatus.adapterStatuses.forEach((key, value) {
      debugPrint('Adapter: $key');
      debugPrint('  State: ${value.initializationState}');
      debugPrint('  Description: ${value.description}');
      debugPrint('  Latency: ${value.latency}ms');
    });
    debugPrint('=====================================');
    
    // 检查是否有适配器未准备好
    final notReadyAdapters = initializationStatus.adapterStatuses.entries
        .where((entry) => 
            entry.value.initializationState != AdapterInitializationState.READY)
        .map((entry) => entry.key)
        .toList();
    
    if (notReadyAdapters.isNotEmpty) {
      debugPrint('Warning: The following adapters are not ready:');
      notReadyAdapters.forEach((adapter) {
        debugPrint('  - $adapter');
      });
    }
  } catch (e) {
    debugPrint('Error initializing Mobile Ads SDK: $e');
  }
}
```

## 实践练习

1. 实现基本的 SDK 初始化
2. 检查所有适配器的状态
3. 实现等待初始化完成的逻辑
4. 处理初始化失败的情况

## 常见问题

### Q: 如果我不等待初始化完成会怎样？

A: 某些适配器可能无法参与广告请求，导致广告填充率降低。

### Q: 初始化需要多长时间？

A: 初始化时间取决于网络状况和适配器数量，通常在几秒内完成。

### Q: 如果某个适配器初始化失败，我还能使用其他网络吗？

A: 可以。即使某些适配器失败，其他已初始化的网络仍然可以正常工作。

### Q: 我需要在每次应用启动时都初始化吗？

A: 是的，建议在应用启动时初始化 SDK，确保所有适配器都准备好。

## 总结与检查清单

### 本章要点

- 使用 `MobileAds.instance.initialize()` 初始化 SDK
- 检查 `InitializationStatus` 和 `AdapterStatus` 来了解适配器状态
- 等待初始化完成以确保所有网络都能参与广告请求
- 处理初始化失败的情况

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `initialize()` 方法的使用
- [ ] `InitializationStatus` 和 `AdapterStatus` 对象
- [ ] 如何检查适配器状态
- [ ] 为什么需要等待初始化完成
- [ ] 如何处理初始化失败

下一章，我们将学习如何在 AdMob 控制台中配置中介。
