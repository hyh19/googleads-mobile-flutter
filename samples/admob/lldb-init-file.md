# Flutter iOS 调试 LLDB Init File 配置教程

本文记录在 iOS 设备（如 iPhone 12 mini）使用 Flutter 调试模式时出现的 LLDB Init File 缺失问题，以及在 Xcode 中的解决步骤。

## 问题现象

运行 `flutter run` 进入调试模式时，Xcode 提示 Runner scheme 未设置 LLDB Init File，典型输出如下：

```text
An error occurred when adding LLDB Init File:
Running Flutter in debug mode on new iOS versions requires a LLDB Init File, but the Runner scheme does not have it set. To ensure debug mode works, please complete the following:
  * Open Xcode > Product > Scheme > Edit Scheme and for the Run and Test actions, set LLDB Init File to:

  $(SRCROOT)/Flutter/ephemeral/flutter_lldbinit
```

原因：较新的 iOS 版本在调试时需要预置 LLDB 初始化脚本，但当前 Runner scheme 未指定该文件路径。

## 解决步骤

1. 在终端执行 `open ios/Runner.xcworkspace` 打开 Xcode。
2. 进入菜单 `Product > Scheme > Edit Scheme…`。
3. 在左侧选择 `Run`，右侧 `Info` 页的 `LLDB Init File` 填入 `$(SRCROOT)/Flutter/ephemeral/flutter_lldbinit`。
4. 同样在左侧选择 `Test`，将 `LLDB Init File` 填入相同路径。
5. 关闭对话框后，重新运行 `flutter run` 验证调试模式是否正常。

## 补充说明

- `$(SRCROOT)/Flutter/ephemeral/flutter_lldbinit` 由 Flutter 自动生成，作用是为调试器注入必要脚本，确保 LLDB 能正确附加 Flutter 进程。
- 若后续清理了构建产物或升级 Flutter，路径会自动保持，无需额外手动更新；若再次出现同类提示，重复上述步骤即可。
