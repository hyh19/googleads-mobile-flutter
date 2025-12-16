# 第 6 章 使用 Platform Channel 调用第三方 SDK API

## 引言

某些第三方广告网络提供了额外的 API，用于设置隐私选项、用户同意等。由于这些 API 只在原生平台可用，我们需要使用 Platform Channel 从 Flutter 代码中调用它们。本章将详细讲解如何实现 Platform Channel 来调用第三方 SDK API。

## Platform Channel 概述

### 什么是 Platform Channel？

Platform Channel 是 Flutter 提供的机制，用于在 Dart 代码和原生平台代码（Android/iOS）之间进行通信。它允许你：

- 从 Dart 调用原生方法
- 从原生代码调用 Dart 方法
- 传递数据（基本类型、Map、List 等）

### 为什么需要 Platform Channel？

在使用广告中介时，某些第三方 SDK 提供了需要在原生代码中调用的 API，例如：

- **隐私设置**：设置用户同意、年龄限制等
- **SDK 配置**：配置 SDK 特定参数
- **分析事件**：记录自定义事件

## Dart 端实现

### 创建 MethodChannel

首先，在 Dart 代码中创建 `MethodChannel`：

```dart
import 'package:flutter/services.dart';

class MyMethodChannel {
  final MethodChannel _methodChannel = MethodChannel(
    'com.example.mediationexample/mediation-channel',
  );
}
```

### 定义方法

为每个需要调用的原生方法创建一个 Dart 方法：

```dart
class MyMethodChannel {
  final MethodChannel _methodChannel = MethodChannel(
    'com.example.mediationexample/mediation-channel',
  );

  /// 设置 AppLovin 用户年龄限制
  Future<void> setAppLovinIsAgeRestrictedUser(bool isAgeRestricted) async {
    return _methodChannel.invokeMethod('setIsAgeRestrictedUser', {
      'isAgeRestricted': isAgeRestricted,
    });
  }

  /// 设置 AppLovin 用户同意状态
  Future<void> setHasUserConsent(bool hasUserConsent) async {
    return _methodChannel.invokeMethod('setHasUserConsent', {
      'hasUserConsent': hasUserConsent,
    });
  }
}
```

### 使用 MethodChannel

在应用中使用 `MyMethodChannel`：

```dart
class _MyHomePageState extends State<MyHomePage> {
  static MyMethodChannel platform = MyMethodChannel();

  @override
  void initState() {
    super.initState();
    
    // 设置 AppLovin 隐私选项
    platform.setAppLovinIsAgeRestrictedUser(true);
    platform.setHasUserConsent(false);
    
    _loadBannerAd();
  }
}
```

## Android 端实现

### 设置 MethodChannel

在 `MainActivity` 中设置 MethodChannel 处理器：

```java
import android.os.Bundle;
import io.flutter.embedding.android.FlutterActivity;
import io.flutter.embedding.engine.FlutterEngine;
import io.flutter.plugin.common.MethodChannel;
import com.applovin.sdk.AppLovinPrivacySettings;

public class MainActivity extends FlutterActivity {
  private static final String CHANNEL_NAME =
      "com.example.mediationexample/mediation-channel";

  @Override
  public void configureFlutterEngine(@NonNull FlutterEngine flutterEngine) {
    super.configureFlutterEngine(flutterEngine);

    // 设置 MethodChannel 处理器
    new MethodChannel(flutterEngine.getDartExecutor().getBinaryMessenger(), CHANNEL_NAME)
        .setMethodCallHandler(
            (call, result) -> {
              switch (call.method) {
                case "setIsAgeRestrictedUser":
                  boolean isAgeRestricted = call.argument("isAgeRestricted");
                  AppLovinPrivacySettings.setIsAgeRestrictedUser(isAgeRestricted, this);
                  result.success(null);
                  break;
                  
                case "setHasUserConsent":
                  boolean hasUserConsent = call.argument("hasUserConsent");
                  AppLovinPrivacySettings.setHasUserConsent(hasUserConsent, this);
                  result.success(null);
                  break;
                  
                default:
                  result.notImplemented();
                  break;
              }
            });
  }
}
```

### 处理错误

在 MethodChannel 处理器中处理错误：

```java
.setMethodCallHandler(
    (call, result) -> {
      try {
        switch (call.method) {
          case "setIsAgeRestrictedUser":
            boolean isAgeRestricted = call.argument("isAgeRestricted");
            AppLovinPrivacySettings.setIsAgeRestrictedUser(isAgeRestricted, this);
            result.success(null);
            break;
            
          case "setHasUserConsent":
            boolean hasUserConsent = call.argument("hasUserConsent");
            AppLovinPrivacySettings.setHasUserConsent(hasUserConsent, this);
            result.success(null);
            break;
            
          default:
            result.notImplemented();
            break;
        }
      } catch (Exception e) {
        result.error("ERROR", e.getMessage(), null);
      }
    });
```

## iOS 端实现

### 设置 MethodChannel

在 `AppDelegate` 中设置 MethodChannel 处理器：

```objective-c
#import "AppDelegate.h"
#import "GeneratedPluginRegistrant.h"
#import <AppLovinSDK/AppLovinSDK.h>

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [GeneratedPluginRegistrant registerWithRegistry:self];

  // 设置 MethodChannel
  FlutterViewController* controller = (FlutterViewController*)self.window.rootViewController;
  
  FlutterMethodChannel* methodChannel = [FlutterMethodChannel
      methodChannelWithName:@"com.example.mediationexample/mediation-channel"
            binaryMessenger:controller.binaryMessenger];
  
  [methodChannel setMethodCallHandler:^(FlutterMethodCall* call, FlutterResult result) {
    if ([call.method isEqualToString:@"setIsAgeRestrictedUser"]) {
      BOOL isAgeRestricted = [call.arguments[@"isAgeRestricted"] boolValue];
      [ALPrivacySettings setIsAgeRestrictedUser:isAgeRestricted];
      result(nil);
    } else if ([call.method isEqualToString:@"setHasUserConsent"]) {
      BOOL hasUserConsent = [call.arguments[@"hasUserConsent"] boolValue];
      [ALPrivacySettings setHasUserConsent:hasUserConsent];
      result(nil);
    } else {
      result(FlutterMethodNotImplemented);
    }
  }];
  
  return [super application:application didFinishLaunchingWithOptions:launchOptions];
}

@end
```

### 处理错误

在 MethodChannel 处理器中处理错误：

```objective-c
[methodChannel setMethodCallHandler:^(FlutterMethodCall* call, FlutterResult result) {
  @try {
    if ([call.method isEqualToString:@"setIsAgeRestrictedUser"]) {
      BOOL isAgeRestricted = [call.arguments[@"isAgeRestricted"] boolValue];
      [ALPrivacySettings setIsAgeRestrictedUser:isAgeRestricted];
      result(nil);
    } else if ([call.method isEqualToString:@"setHasUserConsent"]) {
      BOOL hasUserConsent = [call.arguments[@"hasUserConsent"] boolValue];
      [ALPrivacySettings setHasUserConsent:hasUserConsent];
      result(nil);
    } else {
      result(FlutterMethodNotImplemented);
    }
  } @catch (NSException *exception) {
    result([FlutterError errorWithCode:@"ERROR"
                                 message:exception.reason
                                 details:nil]);
  }
}];
```

## 完整示例

以下是示例项目中的完整实现：

### Dart 端（my_method_channel.dart）

```dart
import 'package:flutter/services.dart';

/// 封装用于调用第三方中介 SDK 的 MethodChannel
class MyMethodChannel {
  final MethodChannel _methodChannel = MethodChannel(
    'com.example.mediationexample/mediation-channel',
  );

  /// 设置 AppLovin 用户年龄限制
  Future<void> setAppLovinIsAgeRestrictedUser(bool isAgeRestricted) async {
    return _methodChannel.invokeMethod('setIsAgeRestrictedUser', {
      'isAgeRestricted': isAgeRestricted,
    });
  }

  /// 设置 AppLovin 用户同意状态
  Future<void> setHasUserConsent(bool hasUserConsent) async {
    return _methodChannel.invokeMethod('setHasUserConsent', {
      'hasUserConsent': hasUserConsent,
    });
  }
}
```

### 使用示例

```dart
class _MyHomePageState extends State<MyHomePage> {
  static MyMethodChannel platform = MyMethodChannel();

  @override
  void initState() {
    super.initState();
    
    // 设置 AppLovin 隐私选项
    platform.setAppLovinIsAgeRestrictedUser(true);
    platform.setHasUserConsent(false);
    
    _loadBannerAd();
  }
}
```

## 其他第三方 SDK API

### Unity Ads

Unity Ads 也提供了类似的隐私 API：

```dart
// Dart 端
Future<void> setUnityAdsPrivacyMode(bool privacyMode) async {
  return _methodChannel.invokeMethod('setUnityAdsPrivacyMode', {
    'privacyMode': privacyMode,
  });
}
```

```java
// Android 端
case "setUnityAdsPrivacyMode":
  boolean privacyMode = call.argument("privacyMode");
  // Unity Ads 隐私设置
  result.success(null);
  break;
```

### IronSource

IronSource 提供了用户同意和年龄限制 API：

```dart
// Dart 端
Future<void> setIronSourceConsent(bool hasConsent) async {
  return _methodChannel.invokeMethod('setIronSourceConsent', {
    'hasConsent': hasConsent,
  });
}
```

## 最佳实践

### 1. 错误处理

始终在 Dart 和原生代码中处理错误：

```dart
try {
  await platform.setAppLovinIsAgeRestrictedUser(true);
} catch (e) {
  debugPrint('Error setting age restriction: $e');
}
```

### 2. 类型安全

确保传递的参数类型正确：

```dart
// ✅ 正确
await platform.setAppLovinIsAgeRestrictedUser(true);

// ❌ 错误
await platform.setAppLovinIsAgeRestrictedUser('true');
```

### 3. 通道名称

使用唯一的通道名称，避免与其他插件冲突：

```dart
// ✅ 使用包名作为前缀
'com.example.mediationexample/mediation-channel'

// ❌ 避免使用通用名称
'mediation-channel'
```

## 实践练习

1. 创建 `MyMethodChannel` 类
2. 在 Android 的 `MainActivity` 中设置 MethodChannel 处理器
3. 在 iOS 的 `AppDelegate` 中设置 MethodChannel 处理器
4. 测试调用 AppLovin 隐私 API

## 常见问题

### Q: 如果原生方法调用失败怎么办？

A: 在 Dart 端使用 `try-catch` 处理错误，在原生端使用 `result.error()` 返回错误。

### Q: 可以传递复杂对象吗？

A: 可以，但需要是可序列化的类型（基本类型、Map、List）。自定义对象需要转换为 Map。

### Q: MethodChannel 是异步的吗？

A: 是的，`invokeMethod` 返回 `Future`，应该使用 `await` 或 `.then()` 处理。

## 总结与检查清单

### 本章要点

- Platform Channel 用于在 Flutter 和原生代码之间通信
- 在 Dart 端创建 `MethodChannel` 并定义方法
- 在 Android 和 iOS 端设置 MethodChannel 处理器
- 处理错误和类型安全

### 检查清单

在继续下一章之前，确保你理解：

- [ ] Platform Channel 的基本概念
- [ ] 如何在 Dart 端创建和使用 MethodChannel
- [ ] 如何在 Android 端设置 MethodChannel 处理器
- [ ] 如何在 iOS 端设置 MethodChannel 处理器
- [ ] 如何调用第三方 SDK API

下一章，我们将学习如何配置网络特定参数（Network Extras）。
