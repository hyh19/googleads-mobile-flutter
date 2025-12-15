# 第 4 章 同意管理与 GDPR

本章基于 `consent_manager.dart` 解释如何获取用户同意、何时请求广告以及隐私入口的处理，满足 GDPR 要求。

## 简介

Google User Messaging Platform 提供同意收集能力。示例封装在 `ConsentManager`，负责更新同意信息、展示表单以及检查隐私入口需求。

## 核心步骤

1. 启动时调用 `gatherConsent`，在回调中检查错误并继续初始化。
2. 使用 `canRequestAds` 判断是否可发起广告请求。
3. `isPrivacyOptionsRequired` 返回是否需要在 UI 中提供隐私入口。
4. 通过 `_consentManager.showPrivacyOptionsForm` 打开隐私设置表单。

## 代码示例

```dart
_consentManager.gatherConsent((consentGatheringError) {
  if (consentGatheringError != null) {
    debugPrint(
      "${consentGatheringError.errorCode}: ${consentGatheringError.message}",
    );
  }
  _getIsPrivacyOptionsRequired();
  _initializeMobileAdsSDK();
});
```

`ConsentManager` 核心方法：

```dart
Future<bool> canRequestAds() async {
  return await ConsentInformation.instance.canRequestAds();
}

void gatherConsent(OnConsentGatheringCompleteListener onComplete) {
  ConsentRequestParameters params = ConsentRequestParameters(
    consentDebugSettings: ConsentDebugSettings(),
  );
  ConsentInformation.instance.requestConsentInfoUpdate(
    params,
    () async {
      ConsentForm.loadAndShowConsentFormIfRequired(onComplete);
    },
    (FormError formError) {
      onComplete(formError);
    },
  );
}
```

## 练习与检查

- 在真机上测试欧洲地区 IP，确认同意表单展示。
- 点击 AppBar 菜单的隐私项，验证隐私入口能打开表单。
- 调试输出同意错误码，确保错误被记录。

## 小结

你已理解同意收集与隐私入口的实现。下一章会讲解 UI 结构与导航。
