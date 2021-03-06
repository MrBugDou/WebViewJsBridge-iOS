# WebViewJsBridge-iOS

[![](https://img.shields.io/badge/build-pass-green)](https://github.com/al-liu/WebViewJsBridge-iOS) [![](https://img.shields.io/badge/language-Objective--C-brightgreen)](https://github.com/al-liu/WebViewJsBridge-iOS) [![](https://img.shields.io/cocoapods/p/HCWebViewJsBridge)](https://github.com/al-liu/WebViewJsBridge-iOS) [![](https://img.shields.io/github/license/al-liu/WebViewJsBridge-iOS)](./LICENSE)

WebViewJsBridge-iOS 是 JavaScript 与 iOS WKWebView 通信的桥接库。

配套的 Android 版本：[https://github.com/al-liu/WebViewJsBridge-Android](https://github.com/al-liu/WebViewJsBridge-Android)

如果还需要兼容 UIWebView 请参考文档底部的说明。

本桥接库的特点是跨平台，支持 iOS，Android。Js 对 iOS，Android 的接口调用统一，简单易用。支持将客户端接口划分到多个实现类中进行管理，用命名空间加以区分，如 ui.alert，ui 对应一个实现类的命名空间，alert 是该实现类的一个实现方法。

## 安装

### CocoaPods
[CocoaPods](https://cocoapods.org/) 是 Cocoa 项目的依赖管理器，使用方法和安装步骤请参考 CocoaPods 的官网。使用 CocoaPods 整合 WebViewJsBridge-iOS 到你的项目中，需要指定如下内容到你的 Podfile 文件：

```oc
platform :ios, '8.0'

target 'TargetName' do
  pod 'HCWebViewJsBridge', '~> 1.2.0'
end
```

然后，运行如下命令：

```oc
$ pod install
```

### 手动安装
下载 HCWebViewJsBridge 的源代码，并添加到自己的项目中即可使用。

### 引入 hcJsBridge.min.js 文件
在 html 文件中引用 js： `<script>./hcJsBridge.min.js</script>` 。

#### 在 vue 项目中引用 hcJsBridge.min.js
引入方法：`import './hcJsBridge.min.js'`
import js 文件后，即可用 `window.hcJsBridge` 调用客户端接口。
如果不想用 `window.hcJsBridge` 的方式调用，可以使用 vue 插件。

```js
var jsBridge = {}
jsBridge.install = function (Vue, options) {
  Vue.prototype.$hcJsBridge = window.hcJsBridge
}
export default jsBridge
```

然后在 main.js 中：

```js
import JsBridge from './plugins/jsBridge'
Vue.use(JsBridge)
```

在 vue 实例中使用：

```js
this.$hcJsBridge.callHandler('ui.alert', data, function (responseData) {
    if (responseData === 'ok') {
      affirm()
    } else {
      cancel()
    }
})
```

## Example 的说明
`/Example/iOS Example` 文件夹下提供完整使用示例，包括基础的调用演示和进阶用法，如，使用 `UIImagePickerController` 调用相机拍摄一张图片，使用 `NSURLSession` 发起一个 GET 请求。

## 使用方法

### 初始化原生的 WebViewJsBridge 环境

#### WKWebView

**如果 H5 引入 hcJsBridge.js，则使用下面这个初始化方法。**

```oc
_bridge = [HCWKWebViewJsBridge bridgeWithWebView:self.wkWebView];
```

**如果 H5 不引入 hcJsBridge.js，则使用下面这个初始化方法。**

```oc
_bridge = [HCWKWebViewJsBridge bridgeWithWebView:self.wkWebView injectJS:YES];
```

### 原生注册接口供 HTML5 调用

```c
UIJsApi *uiApi = [UIJsApi new];
[_bridge addJsBridgeApiObject:uiApi namespace:@"ui"];

RequestJsApi *requestJsApi = [RequestJsApi new];
[_bridge addJsBridgeApiObject:requestJsApi namespace:@"request"];
```

UIJsApi 实现类：

```c
- (void)alert:(NSDictionary *)data callback:(HCJBResponseCallback)callback {
    callback(@"native api alert’callback.");
}
// 接口实现类支持四种方法签名（Js 在调用时要遵循对应的方法签名）：
// 1. 有参数，有回调
- (void)test1:(NSString *)data callback:(HCJBResponseCallback)callback {
    NSLog(@"Js call native api test1, data is:%@", data);
    callback(@"native api test1’callback.(备注：汉字测试)");
}
// 2. 有参数，无回调
- (void)test2:(NSDictionary *)data {
    NSLog(@"Js native api:test2, data is:%@", data);
}
// 3. 无参数，无回调
- (void)test3 {
    NSLog(@"Js native api:test3");
}
// 4. 无参数，有回调
- (void)test4:(HCJBResponseCallback)callback {
    NSLog(@"Js native api:test4");
    callback(@"native api test4'callback.(备注：汉字测试)");
}
```

### 原生调用 HTML5 接口

```c
[_bridge callHandler:@"testCallJs" data:@{@"foo": @"bar"} responseCallback:^(id  _Nonnull responseData) {
    NSLog(@"testCallJs callback data is:%@", responseData);
}];
```

### 初始化 HTML5 的 WebViewJsBridge 环境

初始化环境需要引入  `hcJsBridge.min.js` 文件，引用方式前面有介绍。

**如果 H5 不引入 hcJsBridge.min.js，则需要使用下面的方式注册接口。**

```js
// 在这个 window._hcJsBridgeInitFinished 全局函数中等待 bridge 初始化完成，然后注册接口，初始调用。
window._hcJsBridgeInitFinished = function(bridge) {
    // 注册接口给原生调用
    bridge.registerHandler("test1", function(data, callback) {
        callback('callback native,handlename is test1');
    })
    
    // 调用原生接口
    bridge.callHandler('ui.test3');
}
```

### HTML5 注册接口供原生调用

```js
hcJsBridge.registerHandler("testCallJs", function(data, callback) {
    log('Native call js ,handlename is testCallJs, data is:', data);
    callback('callback native, handlename is testCallJs');
})
```

### HTML5 调用原生接口

```js
var data = {foo: "bar"};
hcJsBridge.callHandler('ui.alert', data, function (responseData) {
    log('Js receives the response data returned by native, response data is', responseData);
})
```

### 开启 debug 日志

开启 debug 日志，将打印一些调用信息，辅助排查问题。debug 日志默认不开启，release 模式下屏蔽 debug 日志，但不屏蔽 error 日志。

```oc
[_bridge enableDebugLogging:YES];
```

## 兼容 UIWebView

您如果还需要兼容 UIWebView，请使用 1.1.0 版本。

```oc
platform :ios, '8.0'

target 'TargetName' do
  pod 'HCWebViewJsBridge', '~> 1.1.0'
end
```

使用过程中如果不起作用，请查看项目中除本工具库外，有没有使用该方法 `- (void)webView:(id)unuse didCreateJavaScriptContext:(JSContext *)ctx forFrame:(id)frame`，如果使用多个该方法只会有一个生效，所以只能保留一个该分类方法，并统一通知的 NotificationName。


```
NSString * const kHCDidCreateJSContextNotification = @"com.lhc.HCWebViewJsBridge.DidCreateJSContext";
@implementation NSObject (HCJSContext)

- (void)webView:(id)unuse didCreateJavaScriptContext:(JSContext *)ctx forFrame:(id)frame {
    [[NSNotificationCenter defaultCenter] postNotificationName:kHCDidCreateJSContextNotification object:ctx];
}
```

## License
WebViewJsBridge-iOS 使用 MIT license 发布，查看 [LICENSE](./LICENSE) 详情。

