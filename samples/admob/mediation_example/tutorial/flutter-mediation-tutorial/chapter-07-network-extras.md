# 第 7 章 网络特定参数（Network Extras）

## 引言

某些广告网络适配器支持额外的参数，可以在创建广告请求时传递。这些参数被称为 Network Extras，允许你为特定的广告网络配置特殊的行为。本章将详细讲解如何实现 Network Extras 提供者，为不同的广告网络传递特定参数。

## 什么是 Network Extras？

### 基本概念

Network Extras 是传递给特定广告网络适配器的额外参数，用于：

- **配置广告行为**：如静音、自动播放等
- **传递用户信息**：如用户 ID、年龄等
- **设置广告格式**：如视频质量、时长等

### 为什么需要 Network Extras？

不同的广告网络支持不同的配置选项。通过 Network Extras，你可以在创建广告请求时为每个网络传递特定的参数，而无需修改适配器代码。

## Android 实现

### 实现 MediationNetworkExtrasProvider

在 Android 中，你需要实现 `MediationNetworkExtrasProvider` 接口：

```java
import com.google.android.gms.ads.mediation.MediationExtrasReceiver;
import com.google.android.gms.ads.mediation.adapter.MediationAdapter;
import com.google.android.gms.ads.mediation.Adapter;
import com.google.ads.mediation.applovin.AppLovinExtras;
import com.google.ads.mediation.applovin.ApplovinAdapter;
import io.flutter.plugins.googlemobileads.GoogleMobileAdsPlugin;

import java.util.HashMap;
import java.util.Map;
import android.os.Bundle;

public class MainActivity extends FlutterActivity {

  @Override
  public void configureFlutterEngine(@NonNull FlutterEngine flutterEngine) {
    super.configureFlutterEngine(flutterEngine);

    // 注册 Network Extras 提供者
    GoogleMobileAdsPlugin.registerMediationNetworkExtrasProvider(
        flutterEngine, new MyMediationNetworkExtrasProvider());
  }
}

class MyMediationNetworkExtrasProvider implements MediationNetworkExtrasProvider {

  @Override
  public Map<Class<? extends MediationExtrasReceiver>, Bundle> getMediationExtras(
      String adUnitId, @Nullable String identifier) {
    
    // 为 AppLovin 创建 Extras
    Bundle appLovinBundle = new AppLovinExtras.Builder()
        .setMuteAudio(true)  // 静音广告
        .build();
    
    Map<Class<? extends MediationExtrasReceiver>, Bundle> extras = new HashMap<>();
    extras.put(ApplovinAdapter.class, appLovinBundle);
    
    return extras;
  }
}
```

### AppLovin Extras 示例

AppLovin 支持以下 Extras：

```java
Bundle appLovinBundle = new AppLovinExtras.Builder()
    .setMuteAudio(true)  // 静音音频
    .build();
```

### 根据广告单元 ID 传递不同参数

你可以根据广告单元 ID 传递不同的参数：

```java
@Override
public Map<Class<? extends MediationExtrasReceiver>, Bundle> getMediationExtras(
    String adUnitId, @Nullable String identifier) {
  
  Map<Class<? extends MediationExtrasReceiver>, Bundle> extras = new HashMap<>();
  
  // 根据广告单元 ID 传递不同参数
  if (adUnitId.contains("banner")) {
    // Banner 广告的 Extras
    Bundle appLovinBundle = new AppLovinExtras.Builder()
        .setMuteAudio(true)
        .build();
    extras.put(ApplovinAdapter.class, appLovinBundle);
  } else if (adUnitId.contains("interstitial")) {
    // Interstitial 广告的 Extras
    Bundle appLovinBundle = new AppLovinExtras.Builder()
        .setMuteAudio(false)
        .build();
    extras.put(ApplovinAdapter.class, appLovinBundle);
  }
  
  return extras;
}
```

## iOS 实现

### 实现 FLTMediationNetworkExtrasProvider

在 iOS 中，你需要实现 `FLTMediationNetworkExtrasProvider` 协议：

```objective-c
#import "AppDelegate.h"
#import "GeneratedPluginRegistrant.h"
#import <GoogleMobileAds/GoogleMobileAds.h>
#import <GoogleMobileAdsMediationAppLovin/GADMAdapterAppLovinExtras.h>

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [GeneratedPluginRegistrant registerWithRegistry:self];

  // 注册 Network Extras 提供者
  MyFLTMediationNetworkExtrasProvider *networkExtrasProvider =
      [[MyFLTMediationNetworkExtrasProvider alloc] init];
  [FLTGoogleMobileAdsPlugin registerMediationNetworkExtrasProvider:networkExtrasProvider
                                                           registry:self];
  
  return [super application:application didFinishLaunchingWithOptions:launchOptions];
}

@end

@implementation MyFLTMediationNetworkExtrasProvider

- (NSArray<id<GADAdNetworkExtras>> *_Nullable)getMediationExtras:(NSString *_Nonnull)adUnitId
                                        mediationExtrasIdentifier:
                                            (NSString *_Nullable)mediationExtrasIdentifier {
  // 为 AppLovin 创建 Extras
  GADMAdapterAppLovinExtras *appLovinExtras = [[GADMAdapterAppLovinExtras alloc] init];
  appLovinExtras.muteAudio = YES;  // 静音广告
  
  return @[ appLovinExtras ];
}

@end
```

### AppLovin Extras 示例

AppLovin 支持以下 Extras：

```objective-c
GADMAdapterAppLovinExtras *appLovinExtras = [[GADMAdapterAppLovinExtras alloc] init];
appLovinExtras.muteAudio = YES;  // 静音音频
```

### 根据广告单元 ID 传递不同参数

```objective-c
- (NSArray<id<GADAdNetworkExtras>> *_Nullable)getMediationExtras:(NSString *_Nonnull)adUnitId
                                        mediationExtrasIdentifier:
                                            (NSString *_Nullable)mediationExtrasIdentifier {
  NSMutableArray<id<GADAdNetworkExtras>> *extras = [[NSMutableArray alloc] init];
  
  // 根据广告单元 ID 传递不同参数
  if ([adUnitId containsString:@"banner"]) {
    // Banner 广告的 Extras
    GADMAdapterAppLovinExtras *appLovinExtras = [[GADMAdapterAppLovinExtras alloc] init];
    appLovinExtras.muteAudio = YES;
    [extras addObject:appLovinExtras];
  } else if ([adUnitId containsString:@"interstitial"]) {
    // Interstitial 广告的 Extras
    GADMAdapterAppLovinExtras *appLovinExtras = [[GADMAdapterAppLovinExtras alloc] init];
    appLovinExtras.muteAudio = NO;
    [extras addObject:appLovinExtras];
  }
  
  return extras;
}
```

## 使用 identifier 参数

`identifier` 参数允许你在 Dart 代码中传递额外的标识符：

### Dart 端

```dart
final adRequest = AdRequest(
  nonPersonalizedAds: true,
  // 传递 identifier
  mediationExtrasIdentifier: 'banner-ads',
);
```

### Android 端

```java
@Override
public Map<Class<? extends MediationExtrasReceiver>, Bundle> getMediationExtras(
    String adUnitId, @Nullable String identifier) {
  
  Map<Class<? extends MediationExtrasReceiver>, Bundle> extras = new HashMap<>();
  
  // 根据 identifier 传递不同参数
  if ("banner-ads".equals(identifier)) {
    Bundle appLovinBundle = new AppLovinExtras.Builder()
        .setMuteAudio(true)
        .build();
    extras.put(ApplovinAdapter.class, appLovinBundle);
  }
  
  return extras;
}
```

### iOS 端

```objective-c
- (NSArray<id<GADAdNetworkExtras>> *_Nullable)getMediationExtras:(NSString *_Nonnull)adUnitId
                                        mediationExtrasIdentifier:
                                            (NSString *_Nullable)mediationExtrasIdentifier {
  NSMutableArray<id<GADAdNetworkExtras>> *extras = [[NSMutableArray alloc] init];
  
  // 根据 identifier 传递不同参数
  if ([mediationExtrasIdentifier isEqualToString:@"banner-ads"]) {
    GADMAdapterAppLovinExtras *appLovinExtras = [[GADMAdapterAppLovinExtras alloc] init];
    appLovinExtras.muteAudio = YES;
    [extras addObject:appLovinExtras];
  }
  
  return extras;
}
```

## 其他网络的 Extras

### Unity Ads

Unity Ads 支持以下 Extras：

```java
// Android
import com.google.ads.mediation.unity.UnityExtras;

Bundle unityBundle = new UnityExtras.Builder()
    .setGdprConsent(true)
    .build();
extras.put(UnityAdapter.class, unityBundle);
```

```objective-c
// iOS
#import <GoogleMobileAdsMediationUnityAds/GADMAdapterUnityExtras.h>

GADMAdapterUnityExtras *unityExtras = [[GADMAdapterUnityExtras alloc] init];
unityExtras.gdprConsent = YES;
```

### IronSource

IronSource 支持以下 Extras：

```java
// Android
import com.google.ads.mediation.ironsource.IronSourceExtras;

Bundle ironSourceBundle = new IronSourceExtras.Builder()
    .setUserId("user123")
    .build();
extras.put(IronSourceAdapter.class, ironSourceBundle);
```

## 完整示例

以下是完整的 Network Extras 实现示例：

### Android

```java
public class MainActivity extends FlutterActivity {

  @Override
  public void configureFlutterEngine(@NonNull FlutterEngine flutterEngine) {
    super.configureFlutterEngine(flutterEngine);

    GoogleMobileAdsPlugin.registerMediationNetworkExtrasProvider(
        flutterEngine, new MyMediationNetworkExtrasProvider());
  }
}

class MyMediationNetworkExtrasProvider implements MediationNetworkExtrasProvider {

  @Override
  public Map<Class<? extends MediationExtrasReceiver>, Bundle> getMediationExtras(
      String adUnitId, @Nullable String identifier) {
    
    Map<Class<? extends MediationExtrasReceiver>, Bundle> extras = new HashMap<>();
    
    // AppLovin Extras
    Bundle appLovinBundle = new AppLovinExtras.Builder()
        .setMuteAudio(true)
        .build();
    extras.put(ApplovinAdapter.class, appLovinBundle);
    
    return extras;
  }
}
```

### iOS

```objective-c
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  [GeneratedPluginRegistrant registerWithRegistry:self];

  MyFLTMediationNetworkExtrasProvider *networkExtrasProvider =
      [[MyFLTMediationNetworkExtrasProvider alloc] init];
  [FLTGoogleMobileAdsPlugin registerMediationNetworkExtrasProvider:networkExtrasProvider
                                                           registry:self];
  
  return [super application:application didFinishLaunchingWithOptions:launchOptions];
}

@end

@implementation MyFLTMediationNetworkExtrasProvider

- (NSArray<id<GADAdNetworkExtras>> *_Nullable)getMediationExtras:(NSString *_Nonnull)adUnitId
                                        mediationExtrasIdentifier:
                                            (NSString *_Nullable)mediationExtrasIdentifier {
  GADMAdapterAppLovinExtras *appLovinExtras = [[GADMAdapterAppLovinExtras alloc] init];
  appLovinExtras.muteAudio = YES;
  
  return @[ appLovinExtras ];
}

@end
```

## 实践练习

1. 在 Android 中实现 `MediationNetworkExtrasProvider`
2. 在 iOS 中实现 `FLTMediationNetworkExtrasProvider`
3. 为 AppLovin 配置静音参数
4. 测试 Network Extras 是否生效

## 常见问题

### Q: 哪些网络支持 Network Extras？

A: 不是所有网络都支持。查看每个适配器的文档了解支持的 Extras。

### Q: Network Extras 是必需的吗？

A: 不是。只有在需要为特定网络传递额外参数时才需要实现。

### Q: 可以为一个请求传递多个网络的 Extras 吗？

A: 可以。返回的 Map（Android）或 Array（iOS）可以包含多个网络的 Extras。

## 总结与检查清单

### 本章要点

- Network Extras 用于为特定广告网络传递额外参数
- 在 Android 中实现 `MediationNetworkExtrasProvider`
- 在 iOS 中实现 `FLTMediationNetworkExtrasProvider`
- 可以根据广告单元 ID 或 identifier 传递不同参数

### 检查清单

在继续下一章之前，确保你理解：

- [ ] Network Extras 的概念和用途
- [ ] 如何在 Android 中实现 Network Extras 提供者
- [ ] 如何在 iOS 中实现 Network Extras 提供者
- [ ] 如何为不同网络配置 Extras

下一章，我们将学习如何检测广告来源网络。
