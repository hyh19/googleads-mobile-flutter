# 第 6 章：应用生命周期管理

## 章节简介

在本章中，我们将学习如何监听应用的生命周期事件，以便在用户将应用切换到前台时自动显示 App Open 广告。我们将创建 `AppLifecycleReactor` 类，使用 `AppStateEventNotifier` 监听应用状态变化，并理解它与 `WidgetsBindingObserver` 的区别。

## 为什么需要生命周期管理

App Open Ads 的核心特性是在用户将应用切换到前台时显示。为了实现这个功能，我们需要：

1. **监听应用状态变化**：知道应用何时切换到前台
2. **自动触发显示**：在适当时机自动显示广告
3. **避免重复显示**：确保不会在短时间内重复显示广告

## AppStateEventNotifier 简介

`AppStateEventNotifier` 是 Google Mobile Ads SDK 提供的工具，用于监听应用状态变化。它专门设计用于 App Open Ads，比 Flutter 的 `WidgetsBindingObserver` 更适合这个场景。

### 为什么使用 AppStateEventNotifier 而不是 WidgetsBindingObserver？

| 特性 | AppStateEventNotifier | WidgetsBindingObserver |
|------|---------------------|----------------------|
| 区分场景 | 能区分 Flutter 视图失去焦点和应用失去焦点 | 无法区分 |
| 适用场景 | 专门为 App Open Ads 设计 | 通用应用生命周期监听 |
| 准确性 | 更准确地检测应用前台/后台状态 | 可能误判某些场景 |

**关键区别**：

- `WidgetsBindingObserver` 会在 Flutter 视图失去焦点时触发（例如，在 Android 上显示系统对话框时）
- `AppStateEventNotifier` 只在应用真正切换到前台或后台时触发

## 创建 AppLifecycleReactor 类

### 1. 创建文件

在 `lib` 目录下创建新文件 `app_lifecycle_reactor.dart`：

```dart
import 'package:flutter/material.dart';
import 'package:app_open_example/app_open_ad_manager.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

/// Listens for app foreground events and shows app open ads.
class AppLifecycleReactor {
  final AppOpenAdManager appOpenAdManager;

  AppLifecycleReactor({required this.appOpenAdManager});
}
```

### 2. 实现监听方法

添加 `listenToAppStateChanges()` 方法来启动监听：

```dart
void listenToAppStateChanges() {
  AppStateEventNotifier.startListening();
  AppStateEventNotifier.appStateStream.forEach(
    (state) => _onAppStateChanged(state),
  );
}
```

**代码说明**：

- `AppStateEventNotifier.startListening()`：启动应用状态监听
- `AppStateEventNotifier.appStateStream`：应用状态变化流
- `forEach`：对每个状态变化执行回调函数

### 3. 实现状态变化处理

添加 `_onAppStateChanged()` 方法来处理状态变化：

```dart
void _onAppStateChanged(AppState appState) {
  debugPrint('New AppState state: $appState');
  if (appState == AppState.foreground) {
    appOpenAdManager.showAdIfAvailable();
  }
}
```

**代码说明**：

- `appState` 参数表示当前应用状态
- `AppState.foreground` 表示应用切换到前台
- 当应用切换到前台时，调用 `showAdIfAvailable()` 显示广告

## AppState 枚举值

`AppState` 枚举包含以下值：

- `AppState.foreground`：应用在前台运行
- `AppState.background`：应用在后台运行

对于 App Open Ads，我们主要关注 `foreground` 状态。

## 完整的 AppLifecycleReactor 实现

以下是完整的 `AppLifecycleReactor` 类实现：

```dart
import 'package:flutter/material.dart';
import 'package:app_open_example/app_open_ad_manager.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';

/// Listens for app foreground events and shows app open ads.
class AppLifecycleReactor {
  final AppOpenAdManager appOpenAdManager;

  AppLifecycleReactor({required this.appOpenAdManager});

  void listenToAppStateChanges() {
    AppStateEventNotifier.startListening();
    AppStateEventNotifier.appStateStream.forEach(
      (state) => _onAppStateChanged(state),
    );
  }

  void _onAppStateChanged(AppState appState) {
    debugPrint('New AppState state: $appState');
    if (appState == AppState.foreground) {
      appOpenAdManager.showAdIfAvailable();
    }
  }
}
```

## 在应用中集成

### 在 main.dart 中初始化

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import '../../../../../flutter-app-open-ads-tutorial/app_open_ad_manager.dart';
import '../../../../../flutter-app-open-ads-tutorial/app_lifecycle_reactor.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  MobileAds.instance.initialize();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'App Open Example',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  late AppOpenAdManager _appOpenAdManager;
  late AppLifecycleReactor _appLifecycleReactor;

  @override
  void initState() {
    super.initState();
    
    // 创建广告管理器并加载广告
    _appOpenAdManager = AppOpenAdManager();
    _appOpenAdManager.loadAd();
    
    // 创建生命周期监听器并启动监听
    _appLifecycleReactor = AppLifecycleReactor(
      appOpenAdManager: _appOpenAdManager,
    );
    _appLifecycleReactor.listenToAppStateChanges();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('App Open Demo Home Page'),
      ),
      body: const Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Leave and switch back to the app to see the ad.'),
          ],
        ),
      ),
    );
  }
}
```

### 初始化顺序

正确的初始化顺序很重要：

1. **WidgetsFlutterBinding.ensureInitialized()**：确保 Flutter 绑定已初始化
2. **MobileAds.instance.initialize()**：初始化 Mobile Ads SDK
3. **创建 AppOpenAdManager**：创建广告管理器
4. **加载广告**：调用 `loadAd()` 预加载广告
5. **创建 AppLifecycleReactor**：创建生命周期监听器
6. **启动监听**：调用 `listenToAppStateChanges()` 开始监听

## 测试生命周期管理

### 测试步骤

1. **运行应用**：启动应用，广告应该开始加载
2. **切换到后台**：按 Home 键或切换到其他应用
3. **切换回前台**：切换回你的应用
4. **观察广告**：应该看到 App Open 广告自动显示

### 检查日志

在控制台中，你应该看到：

```text
New AppState state: AppState.background
New AppState state: AppState.foreground
App Open Ad loaded successfully: Instance of 'AppOpenAd'
Instance of 'AppOpenAd' onAdShowedFullScreenContent
```

## 避免重复显示

### 问题场景

如果用户通过点击广告中的链接离开应用，然后立即返回，可能会触发另一个 App Open 广告。这会导致糟糕的用户体验。

### 解决方案

在 `showAdIfAvailable()` 方法中，我们已经通过 `_isShowingAd` 标志防止了重复显示。但还可以添加额外的保护：

```dart
class AppLifecycleReactor {
  final AppOpenAdManager appOpenAdManager;
  DateTime? _lastAdShownTime;
  final Duration _minTimeBetweenAds = const Duration(seconds: 30);

  AppLifecycleReactor({required this.appOpenAdManager});

  void listenToAppStateChanges() {
    AppStateEventNotifier.startListening();
    AppStateEventNotifier.appStateStream.forEach(
      (state) => _onAppStateChanged(state),
    );
  }

  void _onAppStateChanged(AppState appState) {
    debugPrint('New AppState state: $appState');
    if (appState == AppState.foreground) {
      // 检查距离上次显示广告的时间
      if (_lastAdShownTime == null ||
          DateTime.now().difference(_lastAdShownTime!) > _minTimeBetweenAds) {
        appOpenAdManager.showAdIfAvailable();
        _lastAdShownTime = DateTime.now();
      }
    }
  }
}
```

**注意**：这个示例仅供参考。在实际应用中，你可能需要更复杂的逻辑来判断是否应该显示广告。

## 与 WidgetsBindingObserver 的对比

### 使用 WidgetsBindingObserver（不推荐）

```dart
class _HomePageState extends State<HomePage> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.resumed) {
      // 可能在不应该显示的时候触发
      appOpenAdManager.showAdIfAvailable();
    }
  }
}
```

**问题**：

- 在 Android 上，当显示系统对话框时也会触发 `resumed` 状态
- 无法区分是应用切换到前台还是视图恢复焦点

### 使用 AppStateEventNotifier（推荐）

```dart
AppLifecycleReactor(
  appOpenAdManager: appOpenAdManager,
).listenToAppStateChanges();
```

**优势**：

- 专门为 App Open Ads 设计
- 准确区分应用前台/后台状态
- 更简单易用

## 常见问题

### 什么时候调用 startListening()？

`startListening()` 应该在应用启动时调用一次，通常是在 `initState()` 中。不需要重复调用。

### 需要手动停止监听吗？

通常不需要。`AppStateEventNotifier` 会在应用生命周期内持续工作。如果你确实需要停止监听，可以取消订阅流（但这在大多数情况下是不必要的）。

### 可以同时使用 WidgetsBindingObserver 吗？

可以，但不推荐。对于 App Open Ads，应该使用 `AppStateEventNotifier`。如果你有其他生命周期需求，可以同时使用 `WidgetsBindingObserver`，但要确保它们不会冲突。

### 为什么广告没有自动显示？

可能的原因：

1. **广告尚未加载完成**：确保在调用 `listenToAppStateChanges()` 之前已经调用了 `loadAd()`
2. **SDK 未初始化**：确保在 `main()` 中调用了 `MobileAds.instance.initialize()`
3. **应用状态未正确检测**：检查日志确认 `AppState.foreground` 是否被触发

## 实践练习

完成以下练习以巩固本章内容：

1. **创建 AppLifecycleReactor**：创建 `app_lifecycle_reactor.dart` 文件并实现类
2. **集成到应用**：在 `main.dart` 中集成生命周期监听
3. **测试功能**：运行应用，切换到后台再切换回前台，验证广告自动显示
4. **观察日志**：检查控制台输出，确认状态变化被正确检测
5. **添加保护机制**：实现最小时间间隔保护，避免频繁显示广告

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解为什么需要生命周期管理
- [ ] 理解 `AppStateEventNotifier` 的作用和优势
- [ ] 区分 `AppStateEventNotifier` 和 `WidgetsBindingObserver`
- [ ] 创建 `AppLifecycleReactor` 类
- [ ] 实现 `listenToAppStateChanges()` 方法
- [ ] 实现 `_onAppStateChanged()` 方法处理状态变化
- [ ] 在应用中正确集成生命周期监听
- [ ] 理解正确的初始化顺序
- [ ] 测试生命周期管理功能

## 下一步

现在我们已经实现了应用生命周期管理，广告可以在用户切换回应用时自动显示。在下一章中，我们将学习如何处理广告过期问题，确保不会显示过期的广告。

继续学习：[第 7 章：广告过期处理](chapter-07-ad-expiration.md)
