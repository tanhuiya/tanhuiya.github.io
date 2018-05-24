---
layout: post
title: CocoaPods 多语言实现
date: 2018-05-24 15:59:28.000000000 +09:00
---


最近公司有需求，所有的服务要走上国际化。
之前做的服务（`cocoapods` 私有库）都是只用了中文，借此机会记录一下。

### Step 1

在`Podfile `中使用 `use_frameworks!`. 这样每个私有库的内容会被单独打包进一个framework 内部，而不是全部分散的放在`main bundle 内部`。

### Step 2

私有库配置文件定义.
在podspec 文件中，指定资源打包的方式 `resource_bundles`
`Sample` 如下

```
  s.resource_bundles = {
    'PictureBook' => ['SourceCode/Assets/*']
  }
```
如上，`SourceCode/Assets/*` 目录下的文件都会被打进`PictureBook.bundle` 里。


### Step 3

在你私有库开发目录的资源文件目录下，新建 `Strings File`, 命名为`Localizable`, 这个是默认的名字. 
然后选择`Pods` 工程文件，切换到`Info` tab 栏，添加 简体中文，默认选中刚才创建的 `Localizable.strings` 文件。

再看`Localizable.strings` 文件夹可以展开了，在`Localizable.strings(English)` 和 `Localizable.strings(Chinese(xxx))` 中编写对应的`key-value` 值。

`Sample`分别如下

_`Localizable.strings(English)`_

> ```
> "test"="this is test";
> ```

`Localizable.strings(Chinese(xxx))`

> ```
> "test"="这是测试";
> ```

注意最后的 **_分号_** ，如果不加分号会加载不到文字哦 ！

### Step 4

代码中调用，不能按通常的方式。
一般的调用方式，程序会在`main Bundle` 的根目录下查找语言文件。而我们的语言文件被打包到私有库中（`xxxx.framewok`）,该`framework `中有二进制程序和`xxxx.bundle` 文件，所有的资源文件都会被打包到(`xxxx.bundle`)中。

所以我们要加载资源的时候要指定加载的`bundle`.

**_OC 实现:_**

```
//  NSString+Localable.h
#import <Foundation/Foundation.h>
@interface NSString (Localable)
@property (copy, nonatomic, readonly) NSString *CI_localizable;
@end
// 该类用于定位bundle 位置
@interface NoUse : NSObject
@end


//  NSString+Localable.m
#import "NSString+Localable.h"

@implementation NoUse
@end

@implementation NSString (Localable)

-(NSString *)CI_localizable{
    NSArray *bundlePaths = [[NSBundle bundleForClass:[NoUse class]] pathsForResourcesOfType:@"bundle" inDirectory:nil];
    if (bundlePaths.count < 1) {
        return self;
    }
    NSString *resourcePath = bundlePaths.firstObject;
    NSBundle *resourceBundle = [NSBundle bundleWithPath:resourcePath];
    NSString *ret = [resourceBundle localizedStringForKey:self value:@"" table:nil];
    return ret;
}

@end
```
**_调用：_**
`NSString *retStr = @"test".CI_localizable`

**_Swift 实现:_**

```
import Foundation
extension String{
    class NoUse {}
    var CI_locale: String {
        let bundlePaths = Bundle(for: NoUse.self).paths(forResourcesOfType: "bundle", inDirectory: nil)
        guard bundlePaths.count > 0 , let resourcePath = bundlePaths.first else {
            return self
        }
        let resourceBundle = Bundle(path: resourcePath)
        let msg = NSLocalizedString(self, tableName: nil, bundle: resourceBundle!, value: "", comment: "")
        return msg
    }
}
```
**_调用：_**
`let retStr = "test".CI_locale `

这样就基本`OK`了。