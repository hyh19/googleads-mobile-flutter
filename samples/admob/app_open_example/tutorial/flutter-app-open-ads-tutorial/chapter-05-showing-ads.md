# 第 5 章：展示广告与回调处理

## 章节简介

在本章中，我们将实现 `showAdIfAvailable()` 方法，学习如何展示已加载的 App Open 广告，并处理展示过程中的各种回调事件。我们将详细讲解 `FullScreenContentCallback` 的三个关键回调函数，以及如何正确清理广告资源。

## 展示广告的流程

展示 App Open 广告的基本流程如下：

1. **检查广告可用性**：确保广告已加载且可用
2. **检查展示状态**：确保当前没有正在展示的广告
3. **设置回调**：注册 `FullScreenContentCallback` 处理广告事件
4. **展示广告**：调用 `show()` 方法显示广告
5. **处理回调**：根据广告状态执行相应操作

## 实现 showAdIfAvailable() 方法

### 基础检查

在展示广告之前，我们需要进行几项检查：

```dart
void showAdIfAvailable() {
  // 1. 检查广告是否可用
  if (!isAdAvailable) {
    debugPrint('Tried to show ad before available.');
    loadAd();
    return;
  }
  
  // 2. 检查是否正在展示广告
  if (_isShowingAd) {
    debugPrint('Tried to show ad while already showing an ad.');
    return;
  }
  
  // 3. 展示广告的逻辑将在下面实现
}
```

**代码说明**：

- 如果广告不可用，记录日志并尝试加载新广告
- 如果正在展示广告，防止重复展示
- 这些检查确保广告展示的安全性和用户体验

### 设置 FullScreenContentCallback

在展示广告之前，我们需要设置 `FullScreenContentCallback` 来处理广告的生命周期事件：

```dart
_appOpenAd!.fullScreenContentCallback = FullScreenContentCallback(
  onAdShowedFullScreenContent: (ad) {
    // 广告成功展示时调用
  },
  onAdFailedToShowFullScreenContent: (ad, error) {
    // 广告展示失败时调用
  },
  onAdDismissedFullScreenContent: (ad) {
    // 广告被关闭时调用
  },
);
```

### 展示广告

设置回调后，调用 `show()` 方法展示广告：

```dart
_appOpenAd!.show();
```

## FullScreenContentCallback 详解

`FullScreenContentCallback` 包含三个关键回调函数，让我们逐一详细讲解：

### 1. onAdShowedFullScreenContent

当广告成功展示到全屏时，此回调会被调用：

```dart
onAdShowedFullScreenContent: (ad) {
  _isShowingAd = true;
  debugPrint('$ad onAdShowedFullScreenContent');
}
```

**代码说明**：

- `ad` 参数是正在展示的 `AppOpenAd` 对象
- 设置 `_isShowingAd = true` 标记广告正在展示
- 记录日志以便调试

**使用场景**：

- 暂停应用中的某些操作（如游戏暂停）
- 记录分析事件（广告展示统计）
- 更新 UI 状态

### 2. onAdFailedToShowFullScreenContent

当广告展示失败时，此回调会被调用：

```dart
onAdFailedToShowFullScreenContent: (ad, error) {
  debugPrint('$ad onAdFailedToShowFullScreenContent: $error');
  _isShowingAd = false;
  ad.dispose();
  _appOpenAd = null;
}
```

**代码说明**：

- `ad` 参数是失败的 `AppOpenAd` 对象
- `error` 参数是 `AdError` 对象，包含错误信息
- 重置 `_isShowingAd` 标志
- 调用 `dispose()` 释放广告资源
- 清空 `_appOpenAd` 引用

**错误处理**：

```dart
onAdFailedToShowFullScreenContent: (ad, error) {
  debugPrint('$ad onAdFailedToShowFullScreenContent: $error');
  debugPrint('Error code: ${error.code}');
  debugPrint('Error message: ${error.message}');
  
  _isShowingAd = false;
  ad.dispose();
  _appOpenAd = null;
  
  // 可以选择重新加载广告
  loadAd();
}
```

### 3. onAdDismissedFullScreenContent

当用户关闭广告（点击关闭按钮或返回）时，此回调会被调用：

```dart
onAdDismissedFullScreenContent: (ad) {
  debugPrint('$ad onAdDismissedFullScreenContent');
  _isShowingAd = false;
  ad.dispose();
  _appOpenAd = null;
  loadAd();
}
```

**代码说明**：

- `ad` 参数是被关闭的 `AppOpenAd` 对象
- 重置 `_isShowingAd` 标志
- 调用 `dispose()` 释放广告资源
- 清空 `_appOpenAd` 引用
- **重要**：立即加载下一个广告，为下次显示做准备

**为什么立即加载下一个广告？**

App Open Ads 需要预加载才能在需要时立即显示。如果等到需要显示时再加载，用户会看到明显的延迟，影响用户体验。

## 完整的 showAdIfAvailable() 实现

以下是完整的 `showAdIfAvailable()` 方法实现：

```dart
/// Shows the ad, if one exists and is not already being shown.
///
/// If the previously cached ad has expired, this just loads and caches a
/// new ad.
void showAdIfAvailable() {
  // 检查广告是否可用
  if (!isAdAvailable) {
    debugPrint('Tried to show ad before available.');
    loadAd();
    return;
  }
  
  // 检查是否正在展示广告
  if (_isShowingAd) {
    debugPrint('Tried to show ad while already showing an ad.');
    return;
  }
  
  // 设置全屏内容回调并展示广告
  _appOpenAd!.fullScreenContentCallback = FullScreenContentCallback(
    onAdShowedFullScreenContent: (ad) {
      _isShowingAd = true;
      debugPrint('$ad onAdShowedFullScreenContent');
    },
    onAdFailedToShowFullScreenContent: (ad, error) {
      debugPrint('$ad onAdFailedToShowFullScreenContent: $error');
      _isShowingAd = false;
      ad.dispose();
      _appOpenAd = null;
    },
    onAdDismissedFullScreenContent: (ad) {
      debugPrint('$ad onAdDismissedFullScreenContent');
      _isShowingAd = false;
      ad.dispose();
      _appOpenAd = null;
      loadAd(); // 立即加载下一个广告
    },
  );
  
  // 展示广告
  _appOpenAd!.show();
}
```

## 广告资源清理

### 为什么需要清理？

广告对象占用系统资源，如果不及时清理，可能导致内存泄漏。每次广告展示完成后，都应该调用 `dispose()` 方法释放资源。

### 清理时机

广告资源应该在以下情况下清理：

1. **广告展示失败**：在 `onAdFailedToShowFullScreenContent` 中
2. **广告被关闭**：在 `onAdDismissedFullScreenContent` 中
3. **广告过期**：在检测到广告过期时（下一章会详细讲解）

### dispose() 方法

`dispose()` 方法会：

- 释放广告占用的内存
- 取消所有待处理的请求
- 清理内部状态

**重要**：调用 `dispose()` 后，广告对象不能再使用。如果需要再次展示广告，必须加载新的广告。

## 错误处理最佳实践

### 1. 记录详细错误信息

```dart
onAdFailedToShowFullScreenContent: (ad, error) {
  debugPrint('Failed to show ad: ${ad.adUnitId}');
  debugPrint('Error code: ${error.code}');
  debugPrint('Error domain: ${error.domain}');
  debugPrint('Error message: ${error.message}');
  
  // 处理错误...
}
```

### 2. 根据错误类型采取不同策略

```dart
onAdFailedToShowFullScreenContent: (ad, error) {
  switch (error.code) {
    case 0: // 内部错误
      // 可能需要重新初始化 SDK
      break;
    case 1: // 无效请求
      // 检查配置
      break;
    case 2: // 网络错误
      // 可以稍后重试
      break;
    default:
      // 其他错误
  }
  
  // 清理资源
  _isShowingAd = false;
  ad.dispose();
  _appOpenAd = null;
}
```

### 3. 避免重复展示

通过 `_isShowingAd` 标志防止同时展示多个广告，这是非常重要的用户体验保护措施。

## 测试广告展示

### 创建测试应用

```dart
import 'package:flutter/material.dart';
import '../../../../../flutter-app-open-ads-tutorial/app_open_ad_manager.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: AdTestPage(),
    );
  }
}

class AdTestPage extends StatefulWidget {
  @override
  _AdTestPageState createState() => _AdTestPageState();
}

class _AdTestPageState extends State<AdTestPage> {
  final _manager = AppOpenAdManager();

  @override
  void initState() {
    super.initState();
    // 加载广告
    _manager.loadAd();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Ad Show Test')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                _manager.showAdIfAvailable();
              },
              child: Text('Show Ad'),
            ),
            SizedBox(height: 20),
            Text('Ad Available: ${_manager.isAdAvailable}'),
          ],
        ),
      ),
    );
  }
}
```

### 观察回调执行

运行应用并点击「Show Ad」按钮，观察控制台输出：

1. 如果广告成功展示，你会看到：`onAdShowedFullScreenContent`
2. 如果用户关闭广告，你会看到：`onAdDismissedFullScreenContent`
3. 如果展示失败，你会看到：`onAdFailedToShowFullScreenContent`

## 常见问题

### 为什么需要 _isShowingAd 标志？

`_isShowingAd` 标志防止同时展示多个广告。如果没有这个检查，用户可能在关闭一个广告后立即看到另一个广告，或者多个广告同时展示，导致糟糕的用户体验。

### 什么时候调用 dispose()？

应该在以下情况调用 `dispose()`：

- 广告展示失败后
- 广告被用户关闭后
- 广告过期后（下一章会讲解）

### 为什么在 onAdDismissedFullScreenContent 中立即加载下一个广告？

App Open Ads 需要预加载才能在需要时立即显示。如果等到需要显示时再加载，用户会看到明显的延迟。立即加载下一个广告可以确保在用户下次切换回应用时，广告已经准备好了。

### 可以重复使用同一个广告对象吗？

不可以。每个 `AppOpenAd` 对象只能展示一次。展示完成后必须调用 `dispose()` 释放资源，然后加载新的广告。

## 实践练习

完成以下练习以巩固本章内容：

1. **实现 showAdIfAvailable()**：在 `AppOpenAdManager` 中实现完整的 `showAdIfAvailable()` 方法
2. **设置回调**：实现 `FullScreenContentCallback` 的三个回调函数
3. **测试展示**：创建一个测试应用来验证广告展示功能
4. **观察回调**：运行应用并观察不同场景下的回调执行
5. **资源清理**：确保在所有适当的地方调用 `dispose()`

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解广告展示的基本流程
- [ ] 实现 `showAdIfAvailable()` 方法的基础检查
- [ ] 设置 `FullScreenContentCallback` 处理广告事件
- [ ] 理解并实现 `onAdShowedFullScreenContent` 回调
- [ ] 理解并实现 `onAdFailedToShowFullScreenContent` 回调
- [ ] 理解并实现 `onAdDismissedFullScreenContent` 回调
- [ ] 正确清理广告资源
- [ ] 理解为什么需要立即加载下一个广告

## 下一步

现在我们已经实现了广告的加载和展示功能。在下一章中，我们将学习如何监听应用的生命周期事件，以便在用户将应用切换到前台时自动显示广告。

继续学习：[第 6 章：应用生命周期管理](chapter-06-lifecycle-management.md)
