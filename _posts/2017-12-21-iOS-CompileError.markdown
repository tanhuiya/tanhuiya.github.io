---
layout: post
title: 编译错误 error: clang importer creation failed
date: 2017-12-21 15:59:28.000000000 +09:00
---

项目里有四个` target `，其他三个都能编译通过， 还有一个死活过不了。
四个项目代码完全一样，就是配置不一样。
编译时报以下错误

```
<unknown>:0: error: PCH file '/Users/xxx/Library/Developer/Xcode/DerivedData/yg-hiamcrahplfrcgffnenbmeyjsoep/Build/Intermediates.noindex/PrecompiledHeaders/yg-Bridging-Header-swift_26VLR1JDUWQF2-clang_1N593KDGBQ1NH.pch' not found: module file not found
<unknown>:0: error: clang importer creation failed
```
好像是 `PCH` 的找不到或者`import` 失败。 但是其他三个 `target` 都一样的。
仔细排查后才发现，多了一个 空格 ！！！

![多一个空格图](http://upload-images.jianshu.io/upload_images/1453111-4e21a8802f54ed8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)