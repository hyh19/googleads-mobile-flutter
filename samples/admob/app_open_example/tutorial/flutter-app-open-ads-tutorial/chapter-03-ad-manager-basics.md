# 第 3 章：创建广告管理器基础类

## 章节简介

在本章中，我们将创建 `AppOpenAdManager` 类，这是管理 App Open Ads 的核心组件。我们将学习如何定义类结构、管理广告状态、设置广告单元 ID，以及创建基础方法框架。这是实现 App Open Ads 功能的第一步。

## 为什么需要广告管理器

在实现 App Open Ads 时，我们需要一个专门的类来管理广告的加载、展示和状态。`AppOpenAdManager` 类提供了以下功能：

- **集中管理**：将所有广告相关逻辑集中在一个类中
- **状态跟踪**：跟踪广告的加载状态和展示状态
- **资源管理**：管理广告对象的生命周期
- **错误处理**：统一处理广告加载和展示的错误

## 创建 AppOpenAdManager 类

### 1. 创建文件

在 `lib` 目录下创建新文件 `app_open_ad_manager.dart`：

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'dart:io' show Platform;

/// Utility class that manages loading and showing app open ads.
class AppOpenAdManager {
  // 我们将在下面实现具体内容
}
```

### 2. 定义广告单元 ID

App Open Ads 需要为 Android 和 iOS 平台分别设置不同的广告单元 ID。我们可以使用 `Platform` 类来检测当前平台：

```dart
class AppOpenAdManager {
  /// 广告单元 ID，根据平台自动选择
  String adUnitId = Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/9257395921'  // Android 测试 ID
      : 'ca-app-pub-3940256099942544/5575463023'; // iOS 测试 ID
}
```

**代码说明**：

- `Platform.isAndroid`：检测当前是否为 Android 平台
- 如果为 Android，使用 Android 测试广告单元 ID
- 如果为 iOS，使用 iOS 测试广告单元 ID

### 3. 定义状态管理变量

我们需要跟踪广告的当前状态，包括已加载的广告对象和是否正在展示广告：

```dart
class AppOpenAdManager {
  String adUnitId = Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/9257395921'
      : 'ca-app-pub-3940256099942544/5575463023';

  /// 当前加载的 App Open 广告对象
  AppOpenAd? _appOpenAd;

  /// 标记是否正在展示广告
  bool _isShowingAd = false;
}
```

**变量说明**：

- `_appOpenAd`：存储已加载的 `AppOpenAd` 对象。使用 `?` 表示可能为 `null`（广告尚未加载或已被释放）
- `_isShowingAd`：布尔标志，用于防止同时展示多个广告

### 4. 创建基础方法框架

现在让我们创建基础方法框架，这些方法将在后续章节中实现：

```dart
class AppOpenAdManager {
  String adUnitId = Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/9257395921'
      : 'ca-app-pub-3940256099942544/5575463023';

  AppOpenAd? _appOpenAd;
  bool _isShowingAd = false;

  /// 加载一个 App Open 广告
  void loadAd() {
    // 我们将在下一章实现此方法
  }

  /// 检查是否有可用的广告
  bool get isAdAvailable {
    return _appOpenAd != null;
  }

  /// 如果有可用的广告，则展示它
  void showAdIfAvailable() {
    // 我们将在后续章节实现此方法
  }
}
```

**方法说明**：

- `loadAd()`：负责从 Google 广告服务器加载广告
- `isAdAvailable`：只读属性（getter），检查是否有已加载的广告可用
- `showAdIfAvailable()`：检查广告是否可用，如果可用则展示

## 完整的类结构

以下是 `AppOpenAdManager` 类的完整基础结构：

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'dart:io' show Platform;

/// Utility class that manages loading and showing app open ads.
class AppOpenAdManager {
  /// 广告单元 ID，根据平台自动选择
  String adUnitId = Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/9257395921'
      : 'ca-app-pub-3940256099942544/5575463023';

  /// 当前加载的 App Open 广告对象
  AppOpenAd? _appOpenAd;

  /// 标记是否正在展示广告
  bool _isShowingAd = false;

  /// 加载一个 App Open 广告
  void loadAd() {
    // TODO: 实现广告加载逻辑
  }

  /// 检查是否有可用的广告
  bool get isAdAvailable {
    return _appOpenAd != null;
  }

  /// 如果有可用的广告，则展示它
  void showAdIfAvailable() {
    // TODO: 实现广告展示逻辑
  }
}
```

## 代码详解

### 导入语句

```dart
import 'package:flutter/material.dart';
import 'package:google_mobile_ads/google_mobile_ads.dart';
import 'dart:io' show Platform;
```

- `flutter/material.dart`：提供 `debugPrint` 等调试工具
- `google_mobile_ads/google_mobile_ads.dart`：Google Mobile Ads SDK
- `dart:io`：提供 `Platform` 类用于平台检测

### 私有变量命名

注意我们使用下划线前缀（`_appOpenAd`、`_isShowingAd`）来标记私有变量。这是 Dart 的约定，下划线前缀的变量和方法只能在类内部访问。

### Getter 方法

`isAdAvailable` 是一个 getter 方法，它提供了一种简洁的方式来检查广告是否可用，而不需要直接访问私有变量。

## 测试基础类

虽然我们还没有实现具体功能，但我们可以创建一个简单的测试来验证类结构：

```dart
// 在 main.dart 中测试
import 'package:flutter/material.dart';
import '../../../../../flutter-app-open-ads-tutorial/app_open_ad_manager.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('App Open Ad Test')),
        body: Center(
          child: ElevatedButton(
            onPressed: () {
              final manager = AppOpenAdManager();
              print('Ad Unit ID: ${manager.adUnitId}');
              print('Is Ad Available: ${manager.isAdAvailable}');
            },
            child: Text('Test Manager'),
          ),
        ),
      ),
    );
  }
}
```

## 常见问题

### 为什么使用私有变量？

使用私有变量（`_appOpenAd`、`_isShowingAd`）可以：

- **封装性**：防止外部代码直接修改内部状态
- **安全性**：确保状态变更通过受控的方法进行
- **可维护性**：集中管理状态变更逻辑

### 为什么需要 `_isShowingAd` 标志？

`_isShowingAd` 标志用于防止同时展示多个广告。如果没有这个标志，用户可能在关闭一个广告后立即看到另一个广告，这会导致糟糕的用户体验。

### 如何切换生产环境的广告单元 ID？

当准备发布应用时，只需将测试广告单元 ID 替换为从 AdMob 控制台获取的真实 ID：

```dart
String adUnitId = Platform.isAndroid
    ? 'ca-app-pub-XXXXXXXXXXXXXXXX/XXXXXXXXXX'  // 你的 Android 广告单元 ID
    : 'ca-app-pub-XXXXXXXXXXXXXXXX/XXXXXXXXXX'; // 你的 iOS 广告单元 ID
```

## 实践练习

完成以下练习以巩固本章内容：

1. **创建类文件**：在 `lib` 目录下创建 `app_open_ad_manager.dart` 文件
2. **实现基础结构**：按照本章示例创建类的基础结构
3. **测试平台检测**：验证 `adUnitId` 在不同平台上是否正确设置
4. **测试状态检查**：验证 `isAdAvailable` 在广告未加载时返回 `false`

## 总结检查清单

完成本章学习后，你应该能够：

- [ ] 理解为什么需要广告管理器类
- [ ] 创建 `AppOpenAdManager` 类的基础结构
- [ ] 使用 `Platform` 类检测平台并设置相应的广告单元 ID
- [ ] 定义私有变量来管理广告状态
- [ ] 创建基础方法框架（`loadAd`、`isAdAvailable`、`showAdIfAvailable`）
- [ ] 理解私有变量和 getter 方法的作用

## 下一步

现在我们已经创建了基础类结构，在下一章中，我们将实现 `loadAd()` 方法，学习如何从 Google 广告服务器加载 App Open 广告。

继续学习：[第 4 章：加载 App Open 广告](chapter-04-loading-ads.md)
