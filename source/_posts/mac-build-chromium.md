---
title: mac下载编译chromium源码
date: 2018-11-21 10:11:58
tags: chromium
---

> 此文章来自网络。由于按照官方的教程没有成功编译iOS版，而按此文章成功编译，特此记录，[原文链接](http://www.alonemonkey.com/2016/06/15/chromium-source-compile/)

## 安装depot_tools工具
#### 获取depot_tools工具
```sh
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```
#### 添加环境变量
把depot_tools工具路径添加到系统的环境变量，~/.bashrc or ~/.zshrc。
```sh
export PATH=$PATH:/path/to/depot_tools
```
## 获取chromium源码
```sh
mkdir chromium
cd chromium
fetch --no-history chromium
```

`--no-history`参数可以不下载以往的历史信息。
如果网络中断了，输入gclient sync继续下载。

获取依赖：
```sh
gclient runhooks
```

更新代码：
```sh
git rebase-update
gclient sync
```
编译chromium源码
```sh
GYP_DEFINES=mac_sdk=10.11（可选）
export GYP_GENERATORS=ninja,xcode-ninja
./build/gyp_chromium
ninja -C out/Release/ chrome
```
最新使用gn用于生成ninja文件，gn gen out/Default。

问题：
```sh
Traceback (most recent call last):
  File "./gyp-mac-tool", line 712, in <module>
    sys.exit(main(sys.argv[1:]))
  File "./gyp-mac-tool", line 29, in main
    exit_code = executor.Dispatch(args)
  File "./gyp-mac-tool", line 44, in Dispatch
    return getattr(self, method)(*args[1:])
  File "./gyp-mac-tool", line 67, in ExecCopyBundleResource
    self._CopyStringsFile(source, dest)
  File "./gyp-mac-tool", line 133, in _CopyStringsFile
    import CoreFoundation
ImportError: No module named CoreFoundation
[6208/20844] MACTOOL copy-bundle-resource ../../chrome/app/nibs/AppMenu.xib
```
解决：
```sh
pip install pyobjc
```
#### 运行chromiun
编译完成后会在out/Release下生成Chromium.app，直接运行：
<div style="text-align: center;">
  <img src="1.png"/>
</div>

## 编译iOS平台
为了编译成iOS平台的版本，可以直接把本地存在的Mac checkout转成iOS checkout，只要在chromium/.gclient文件的最后加一行target_os = ["ios"]

这里使用gn生成ninja文件(按照官方的教程，正是缺少此处)：
```sh
gn args out/Debug-iphonesimulator
```

写入内容：
```sh
# Set to true if you have a valid code signing key.
ios_enable_code_signing = false
target_os = "ios"
# Set to "x86", "x64", "arm", "armv7", "arm64". "x86" and "x64" will create a
# build to run on the iOS simulator (and set use_ios_simulator = true), all
# others are for an iOS device.
target_cpu = "x64"
# Release vs debug build.
is_debug = true
```
最后执行：
```sh
gclient sync
```
编译：
这里有点不一样，可以先尝试：
```sh
autoninja -C out/Debug-iphonesimulator gn_all
```
or

```sh
ninja -C out/Debug-iphonesimulator All
```
运行测试app:
```sh
./iossim -d "iPhone 6" -s 9.3 ./base_unittests.app
```
#### 单个模块
```sh
ninja -C out/Debug-iphonesimulator net
```
```sh
export GYP_GENERATORS=xcode-ninja 
export CHROMIUM_GYP_FILE='net/net.gyp'
./build/gyp_chromium
```
参考文档
[Get the Code: Checkout, Build, & Run Chromium](https://www.chromium.org/developers/how-tos/get-the-code)