# `lib/main.dart` 代码讲解

本文概览 `lib/main.dart` 的核心结构，解释同意管理、Mobile Ads SDK 初始化、横幅广告加载与屏幕方向处理，并提供使用与排查提示。

## 应用流程概览

- 应用入口在 `main()`，启动前确保 Flutter 绑定初始化，再以 `BannerExample` 作为根页面。
- `BannerExample` 是一个 `StatefulWidget`，`BannerExampleState` 负责同意流程、SDK 初始化、横幅广告加载与重载。
- 同意获取完成后，根据结果决定是否可请求广告；若允许则初始化 Mobile Ads SDK 并加载自适应横幅。
- `OrientationBuilder` 监控横竖屏变化：方向变更后释放旧广告并重新加载，以匹配当前宽度。

## 关键成员与角色

- `_consentManager`：封装同意管理逻辑，提供 `gatherConsent`、`canRequestAds`、`isPrivacyOptionsRequired`、`showPrivacyOptionsForm`。
- `_isMobileAdsInitializeCalled`：防止重复初始化 Mobile Ads SDK。
- `_isPrivacyOptionsRequired`：指示是否需要在 AppBar 菜单中展示隐私设置入口。
- `_bannerAd`：当前加载的 `BannerAd` 实例，随方向变化或失败时更新与释放。
- `_currentOrientation`：记录上一次的方向，帮助检测变化。
- `_adUnitId`：平台区分的测试广告单元 ID（Android 与 iOS 各一条）。

## 生命周期与同意流程

- `initState`：
  - 调用 `_consentManager.gatherConsent` 获取当前会话的同意状态，记录错误（如有）并拉取隐私入口需求。
  - 同意获取后尝试 `_initializeMobileAdsSDK()`；同时也尝试用上一次会话同意状态直接初始化一次。
- `dispose`：释放 `_bannerAd` 避免资源泄漏。

## SDK 初始化与广告加载

- `_initializeMobileAdsSDK()`：
  - 若已初始化则直接返回。
  - 调用 `canRequestAds()`，只有在同意允许请求广告时才标记初始化已调用并执行 `MobileAds.instance.initialize()`，随后触发 `_loadAd()`。
- `_loadAd()` 主要步骤：
  - 再次校验 `canRequestAds()` 与组件 `mounted` 状态，确保安全。
  - 通过 `AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize` 获取与当前屏幕宽度匹配的锚定自适应尺寸；返回空时跳出。
  - 创建 `BannerAd`，设置测试广告单元、请求对象与尺寸，并注册监听器处理加载成功、失败、曝光、点击等事件；最后调用 `.load()`。
  - 加载成功回调中使用 `setState` 存储 `BannerAd`，供 UI 展示；失败时打印错误并释放广告。

## UI 结构与方向处理

- 页面由 `MaterialApp` 包裹 `Scaffold`，AppBar 标题固定为 “Banner Example”。
- `AppBar` 右侧 actions 由 `_appBarActions()` 提供：
  - 默认包含 “Ad Inspector” 入口，点击后调用 `MobileAds.instance.openAdInspector`。
  - 当 `_isPrivacyOptionsRequired` 为真时额外加入 “Privacy Settings”，点击后显示隐私选项表单。
- `body` 使用 `OrientationBuilder`：
  - 检测方向变化，若与 `_currentOrientation` 不同且已有历史方向，则先 `dispose` 旧广告、清空引用，再调用 `_loadAd()` 重载。
  - 更新 `_currentOrientation` 后，通过 `Stack` + `Align` + `SafeArea` 在底部居中展示广告，尺寸取决于 `_bannerAd!.size`。

## 使用与注意事项

- 该示例使用官方测试广告单元 ID，勿用于生产。发布前应替换为自己的广告单元并按照政策配置同意消息。
- 在请求广告前必须完成同意检查，示例通过 `ConsentManager` 统一处理。
- 自适应横幅尺寸依赖当前屏幕宽度，方向变化会触发重新计算与加载。
- iOS 端的 `onAdWillDismissScreen` 回调用于全屏视图即将关闭的时机；Android 不触发该回调。

## 常见排查提示

- **始终返回 size 为 null**：确认 `MediaQuery` 可用（需要在 widget 树中）、屏幕宽度获取成功。
- **广告不展示**：检查 `canRequestAds()` 是否为真，同意状态是否允许；查看加载失败回调中的错误信息。
- **方向切换后广告不刷新**：确保 `OrientationBuilder` 未被移除，且 `_bannerAd` 在方向变化时被正确释放与重建。
- **隐私入口未显示**：确认 `_isPrivacyOptionsRequired` 是否已通过 `_consentManager.isPrivacyOptionsRequired()` 置为 true。
