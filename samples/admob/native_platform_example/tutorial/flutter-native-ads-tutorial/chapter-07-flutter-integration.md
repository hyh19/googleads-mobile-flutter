# 第 7 章 Flutter 层集成

## 引言

在完成 Android 和 iOS 的原生工厂实现后，我们需要在 Flutter 层加载和显示原生广告。本章将详细讲解如何在 Flutter 中使用 `NativeAd` 类加载广告、使用 `factoryId` 匹配工厂、处理事件回调，以及使用 `AdWidget` 显示广告。

## NativeAd 类概述

### 基本概念

`NativeAd` 是 Flutter 中用于加载和显示原生广告的类。与其他广告格式不同，原生广告需要指定 `factoryId` 来匹配原生层的工厂。

### 关键属性

- `adUnitId`：广告单元 ID
- `factoryId`：工厂 ID，必须与原生层注册的工厂 ID 匹配
- `listener`：事件监听器
- `request`：广告请求对象

## 加载原生广告

### 基本加载实现

```dart
NativeAd? _nativeAd;
bool _nativeAdIsLoaded = false;

void _loadAd() async {
  // 检查是否可以请求广告
  var canRequestAds = await _consentManager.canRequestAds();
  if (!canRequestAds) {
    return;
  }

  setState(() {
    _nativeAdIsLoaded = false;
  });

  _nativeAd = NativeAd(
    adUnitId: _adUnitId,
    factoryId: 'adFactoryExample',  // 必须与原生层匹配
    listener: NativeAdListener(
      onAdLoaded: (ad) {
        print('$NativeAd loaded.');
        setState(() {
          _nativeAdIsLoaded = true;
        });
      },
      onAdFailedToLoad: (ad, error) {
        print('$NativeAd failedToLoad: $error');
        ad.dispose();
      },
    ),
    request: const AdRequest(),
  )..load();
}
```

### factoryId 的重要性

`factoryId` 是连接 Flutter 层和原生层的桥梁：

- **必须匹配**：Flutter 层的 `factoryId` 必须与原生层注册的工厂 ID 完全匹配
- **区分工厂**：如果注册了多个工厂，可以使用不同的 `factoryId` 来选择不同的工厂
- **错误处理**：如果 `factoryId` 不匹配，广告将无法加载

## NativeAdListener 回调

### 所有可用回调

`NativeAdListener` 提供了多个回调来处理广告事件：

```dart
listener: NativeAdListener(
  // 广告加载成功
  onAdLoaded: (ad) {
    print('NativeAd loaded.');
    setState(() {
      _nativeAdIsLoaded = true;
    });
  },
  
  // 广告加载失败
  onAdFailedToLoad: (ad, error) {
    print('NativeAd failedToLoad: $error');
    ad.dispose();
  },
  
  // 用户点击广告
  onAdClicked: (ad) {
    print('NativeAd clicked.');
  },
  
  // 广告产生展示
  onAdImpression: (ad) {
    print('NativeAd impression.');
  },
  
  // 广告关闭（移除覆盖层）
  onAdClosed: (ad) {
    print('NativeAd closed.');
  },
  
  // 广告打开（显示覆盖层）
  onAdOpened: (ad) {
    print('NativeAd opened.');
  },
  
  // iOS 专用：即将关闭全屏视图
  onAdWillDismissScreen: (ad) {
    print('NativeAd will dismiss screen.');
  },
  
  // 广告收益事件
  onPaidEvent: (ad, valueMicros, precision, currencyCode) {
    print('NativeAd paid event: $valueMicros $currencyCode');
  },
),
```

## 使用 AdWidget 显示广告

### 基本显示

使用 `AdWidget` 来显示原生广告：

```dart
if (_nativeAdIsLoaded && _nativeAd != null)
  SizedBox(
    height: _nativeAdHeight,
    width: MediaQuery.of(context).size.width,
    child: AdWidget(ad: _nativeAd!),
  ),
```

### 设置广告尺寸

原生广告的尺寸由原生层的布局决定，但可以在 Flutter 层设置容器尺寸：

```dart
final double _nativeAdHeight = Platform.isAndroid ? 320 : 300;

SizedBox(
  height: _nativeAdHeight,
  width: MediaQuery.of(context).size.width,
  child: AdWidget(ad: _nativeAd!),
),
```

### 使用 Stack 布局

可以使用 `Stack` 来更好地控制广告位置：

```dart
Stack(
  children: [
    SizedBox(
      height: _nativeAdHeight,
      width: MediaQuery.of(context).size.width,
    ),
    if (_nativeAdIsLoaded && _nativeAd != null)
      SizedBox(
        height: _nativeAdHeight,
        width: MediaQuery.of(context).size.width,
        child: AdWidget(ad: _nativeAd!),
      ),
  ],
),
```

## 广告状态管理

### 状态变量

管理广告的加载状态：

```dart
class NativeExampleState extends State<NativeExample> {
  NativeAd? _nativeAd;
  bool _nativeAdIsLoaded = false;
  
  // ...
}
```

### 更新状态

在广告加载成功或失败时更新状态：

```dart
onAdLoaded: (ad) {
  setState(() {
    _nativeAdIsLoaded = true;
  });
},

onAdFailedToLoad: (ad, error) {
  setState(() {
    _nativeAdIsLoaded = false;
  });
  ad.dispose();
},
```

## 完整的 Flutter 集成示例

以下是示例项目中的完整实现：

```dart
class NativeExampleState extends State<NativeExample> {
  final _consentManager = ConsentManager();
  final double _nativeAdHeight = Platform.isAndroid ? 320 : 300;
  var _isMobileAdsInitializeCalled = false;
  var _isPrivacyOptionsRequired = false;
  NativeAd? _nativeAd;
  bool _nativeAdIsLoaded = false;

  final String _adUnitId = Platform.isAndroid
      ? 'ca-app-pub-3940256099942544/2247696110'
      : 'ca-app-pub-3940256099942544/3986624511';

  @override
  void initState() {
    super.initState();

    _consentManager.gatherConsent((consentGatheringError) {
      if (consentGatheringError != null) {
        debugPrint(
          "${consentGatheringError.errorCode}: ${consentGatheringError.message}",
        );
      }

      _getIsPrivacyOptionsRequired();
      _initializeMobileAdsSDK();
    });

    _initializeMobileAdsSDK();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Native Example',
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Native Example'),
          actions: _appBarActions(),
        ),
        body: Center(
          child: Column(
            children: [
              Stack(
                children: [
                  SizedBox(
                    height: _nativeAdHeight,
                    width: MediaQuery.of(context).size.width,
                  ),
                  if (_nativeAdIsLoaded && _nativeAd != null)
                    SizedBox(
                      height: _nativeAdHeight,
                      width: MediaQuery.of(context).size.width,
                      child: AdWidget(ad: _nativeAd!),
                    ),
                ],
              ),
              TextButton(
                onPressed: _loadAd,
                child: const Text("Refresh Ad"),
              ),
            ],
          ),
        ),
      ),
    );
  }

  /// 加载原生广告
  void _loadAd() async {
    var canRequestAds = await _consentManager.canRequestAds();
    if (!canRequestAds) {
      return;
    }

    setState(() {
      _nativeAdIsLoaded = false;
    });

    _nativeAd = NativeAd(
      adUnitId: _adUnitId,
      factoryId: 'adFactoryExample',
      listener: NativeAdListener(
        onAdLoaded: (ad) {
          print('$NativeAd loaded.');
          setState(() {
            _nativeAdIsLoaded = true;
          });
        },
        onAdFailedToLoad: (ad, error) {
          print('$NativeAd failedToLoad: $error');
          ad.dispose();
        },
        onAdClicked: (ad) {},
        onAdImpression: (ad) {},
        onAdClosed: (ad) {},
        onAdOpened: (ad) {},
        onAdWillDismissScreen: (ad) {},
        onPaidEvent: (ad, valueMicros, precision, currencyCode) {},
      ),
      request: const AdRequest(),
    )..load();
  }

  void _initializeMobileAdsSDK() async {
    if (_isMobileAdsInitializeCalled) {
      return;
    }

    if (await _consentManager.canRequestAds()) {
      _isMobileAdsInitializeCalled = true;
      MobileAds.instance.initialize();
      _loadAd();
    }
  }

  @override
  void dispose() {
    _nativeAd?.dispose();
    super.dispose();
  }
}
```

## 资源清理

### 在 dispose() 中清理

在组件销毁时，必须清理广告资源：

```dart
@override
void dispose() {
  _nativeAd?.dispose();
  _nativeAd = null;
  super.dispose();
}
```

### 加载新广告前清理

在加载新广告之前，清理旧的广告：

```dart
void _loadAd() async {
  // 清理旧广告
  _nativeAd?.dispose();
  _nativeAd = null;
  
  // 加载新广告
  _nativeAd = NativeAd(/* ... */)..load();
}
```

## 错误处理

### 处理加载失败

```dart
onAdFailedToLoad: (ad, error) {
  print('NativeAd failedToLoad: $error');
  print('Error code: ${error.code}');
  print('Error domain: ${error.domain}');
  print('Error message: ${error.message}');
  
  // 清理资源
  ad.dispose();
  setState(() {
    _nativeAdIsLoaded = false;
    _nativeAd = null;
  });
  
  // 可以在这里实现重试逻辑
},
```

### 检查 factoryId 匹配

如果广告加载失败，检查 `factoryId` 是否匹配：

1. **检查 Android**：确认 `MainActivity` 中注册的 `factoryId` 与 Flutter 层一致
2. **检查 iOS**：确认 `AppDelegate` 中注册的 `factoryId` 与 Flutter 层一致
3. **检查大小写**：`factoryId` 区分大小写，必须完全匹配

## 实践练习

1. 在 Flutter 中实现 `_loadAd()` 方法
2. 设置正确的 `factoryId`
3. 实现 `NativeAdListener` 的所有回调
4. 使用 `AdWidget` 显示广告
5. 实现资源清理逻辑

## 常见问题

### Q: 如果 `factoryId` 不匹配会怎样？

A: 广告将无法加载，`onAdFailedToLoad` 回调会被触发。

### Q: 可以同时加载多个原生广告吗？

A: 可以，但每个广告需要使用不同的 `factoryId` 或不同的 `NativeAd` 实例。

### Q: 广告视图的尺寸是由什么决定的？

A: 广告视图的尺寸由原生层的布局文件（XML/XIB）决定。Flutter 层只能设置容器的尺寸。

### Q: 如何刷新广告？

A: 调用 `_loadAd()` 方法重新加载广告。记得先清理旧广告。

## 总结与检查清单

### 本章要点

- 使用 `NativeAd` 类加载原生广告
- `factoryId` 必须与原生层注册的工厂 ID 匹配
- 使用 `NativeAdListener` 处理广告事件
- 使用 `AdWidget` 显示原生广告
- 及时清理广告资源

### 检查清单

在继续下一章之前，确保你理解：

- [ ] `NativeAd` 类的使用
- [ ] `factoryId` 的作用和匹配要求
- [ ] `NativeAdListener` 的所有回调
- [ ] 如何使用 `AdWidget` 显示广告
- [ ] 如何清理广告资源

下一章，我们将学习用户同意管理（UMP）的集成。
