---
title: 混合开发的JsBridge(-)
date: 2019-07-11 13:23:41
tags: [Mobile, JSBirdege]
categories: [Mobile]
description: 在混合开发中要和原生应用端产生交互，互相调用方法这里介绍JsBridge
---

## 简介

**JSBridge**
听其取名就是 js 和 Native 之前的桥梁，而实际上 JSBridge 确实是 JS 和 Native 之前的一种通信方式。混合开发，最重要的问题是：**H5 和 Native 的双向通信**。

**JSBridge 的实现原理**
`JavaScript` 是运行在一个单独的 `JS Context` 中（例如，`WebView` 的 `Webkit` 引擎、`JSCore`）。由于这些 `Context`与原生运行环境的天然隔离，我们可以将这种情况与 `RPC`（Remote Procedure Call，远程过程调用）通信进行类比，将 `Native` 与 `JavaScript` 的每次互相调用看做一次 `RPC` 调用。
在 `JSBridge`的设计中，可以把前端看做 `RPC` 的客户端，把 `Native` 端看做 `RPC` 的服务器端，从而 `JSBridge` 要实现的主要逻辑就出现了：通信调用（`Native`与 `JS` 通信） 和 句柄解析调用。（如果你是个前端，而且并不熟悉 `RPC`的话，你也可以把这个流程类比成 JSONP 的流程）

流程如下图所示：
![JSBridege](../../images/moblie/JSBirgde.png)

<!-- ![JSBridege](../../images/moblie/JSBirgde-1-2.png) -->

## H5 和 NA 的双向通信通用方法

H5 通信方式和兼容性如下表所示。指的是借助 Native 的 webview 加载 H5 页面，H5 和 NA 之间通过 API、URL 拦截、全局调用等形式，实现消息通信。

### H5 调用 NA 方法

|      平台      |                    方法                    |         备注         |
| :------------: | :----------------------------------------: | :------------------: |
|    Android     |          shouldOverrideUrlLoading          |   scheme 拦截方法    |
|    Android     |           addJavascriptInterface           |                      |
|    Android     | onJsAlert()、onJsConfirm()、onJsPrompt（） |                      |
|      IOS       |                  拦截 URL                  |                      |
| IOS(UIwebview) |               JavaScriptCore               | API 方法，IOS7+ 支持 |
| IOS(WKwebview) |       window.webkit.messageHandlers        | API 方法，IOS7+ 支持 |

### NA 调用 H5 方法

|      平台      |                  方法                  |     备注      |
| :------------: | :------------------------------------: | :-----------: |
|    Android     |               loadurl()                | Android 4.4 + |
|    Android     |          evaluateJavascript()          |               |
| IOS(UIwebview) | stringByEvaluatingJavaScriptFromString |               |
| IOS(UIwebview) |             JavaScriptCore             |    IOS7.0+    |
| IOS(UIwebview) |  evaluateJavaScript:javaScriptString   |    iOS8.0+    |

## 常用的 JSBridge 形式

- `H5 调 Android`-原生通过`addJavascriptInterface`注册，然后 H5 直接调用
- `Android 调 H5`-原生通过`loadUrl`来调用 H5，`4.4`及以上还可以通过`evaluateJavascript`调用
- `H5 调 iOS`-原生通过`JavaScriptCore`注册（需 ios7 以上），然后 H5 直接调用
- `iOS 调 H5`-通过`stringByEvaluatingJavaScriptFromString`
- 改写浏览器原有对象
- `url scheme`交互

### H5 调 Android

JSInterface 是安卓 4.2-官方推荐的解决方案，JSInterface 在 4.2 之前的版本都可以，但是存在严重的安全隐患，容易被利用提权。实现如下：
首先，原声 webview 需要先注册可供前端调用的 JS 函数：

```javascript
    WebSettings webSettings = mWebView.getSettings();
    //Android容器允许JS脚本
    webSettings.setJavaScriptEnabled(true);
    private Object getJSBridge(){
        Object insertObj = new Object(){
            @JavascriptInterface
            public String foo(){
                return "foo";
            }

            @JavascriptInterface
            public String foo2(final String param){
                return "foo2:" + param;
            }

        };
        return insertObj;
    }
    //Android容器设置侨连对象
    mWebView.addJavascriptInterface(getJSBridge(), "JSBridge");
```

Native 中通过 addJavascriptInterface 添加暴露出来的 JS 桥对象,然后再该对象内部声明对应的 API 方法。

**H5 调用 Native 的方法**

```javascript
//调用方法一
window.JSBridge.foo(); //返回:'foo'
//调用方法二
window.JSBridge.foo2('test'); //返回:'foo2:test'
```

> - 在 Android4.2 以上(api17 后),暴露的 api 要加上注解@JavascriptInterface，否则会找不到方法。

- 在 api17 以前,addJavascriptInterface 有风险,hacker 可以通过反编译获取 Native 注册的 Js 对象， 然后在页面通过反射 Java 的内置静态类，获取一些敏感的信息和破坏
- JS 调用 Native 暴露的 api,并且能得到相应返回值

### Android 调 H5

native 调用 js 比较简单，只要遵循："javascript: 方法名('参数,需要转为字符串')"的规则即可。

**在`4.4`版本之前**

```javascript
    // mWebView = new WebView(this);
    mWebView.loadUrl("javascript: 方法名('参数,需要转为字符串')");
    //ui线程中运行
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            mWebView.loadUrl("javascript: 方法名('参数,需要转为字符串')");
            Toast.makeText(Activity名.this, "调用方法...", Toast.LENGTH_SHORT).show();
        }
    });
```

**在 4.4 及以后（包括）**

```javascript
// 异步执行JS代码,并获取返回值
mWebView.evaluateJavascript("javascript: 方法名('参数,需要转为字符串')", new ValueCallback<String>() {
    @Override
    public void onReceiveValue(String value) {
        // 这里的value即为对应JS方法的返回值
    }
});

```

> - 4.4 之前 Native 通过 loadUrl 来调用 JS 方法,只能让某个 JS 方法执行,但是无法获取该方法的返回值

- 4.4 及之后,通过 evaluateJavascript 异步调用 JS 方法,并且能在 onReceiveValue 中拿到返回值
- mWebView.loadUrl(“javascript: 方法名(‘参数,需要转为字符串’)”); 函数需在 UI 线程运行，因为 mWebView 为 UI 控件(但是有一个坏处是会阻塞 UI 线程)

### H5 调 iOS

Native 中通过引入官方提供的 JavaScriptCore 库(iOS7 以上),然后可以将 api 绑定到 JSContext 上(然后 Html 中 JS 默认通过 window.top.\*可调用)。
以 OC 为例：

```javascript
    #import <JavaScriptCore/JavaScriptCore.h>
    -(void)webViewDidFinishLoad:(UIWebView *)webView{
        [self hideProgress];
        [self setJSInterface];
    }

    -(void)setJSInterface{
        JSContext *context =[_wv valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
        // 注册名为foo的api方法
        context[@"foo"] = ^() {
            //获取参数
            NSArray *args = [JSContext currentArguments];
            NSString *title = [NSString stringWithFormat:@"%@",[args objectAtIndex:0]];
            //做一些自己的逻辑
            //返回一个值  'foo:'+title
            return [NSString stringWithFormat:@"foo:%@", title];
        };
    }
```

H5 调用 IOS 方法：

```javascript
window.top.foo('test');
```

> - iOS7 之前，js 无法直接调用 Native,只能通过 urlscheme 方式间接调用

- JS 能调用到已经暴露的 api,并且能得到相应返回值
- iOS 原生本身是无法被 JS 调用的,但是通过引入官方提供的第三方"JavaScriptCore",即可开放 api 给 JS 调用

### iOS 调 H5

Native 调用 js 的方法比较简单，Native 通过 stringByEvaluatingJavaScriptFromString 调用 Html 绑定在 window 上的函数。不过应注意 Oc 和 Swift 的写法。

```javascript
    // 可以取得JS函数执行的返回值
    // 方法必须是Html页面绑定在最顶层的window上对象的
    // 如window.top.foo
    // Swift
    webview.stringByEvaluatingJavaScriptFromString("方法名(参数)")
    // OC
    [webView stringByEvaluatingJavaScriptFromString:@"方法名(参数);"];
```

> - Native 调用 JS 方法时,能拿到 JS 方法的返回值

- 不适合传输大量数据(大量数据建议用接口方式获取)
- 有 iframe 时，需要获取顶层窗口的引用

### 改写浏览器原有对象

改写 window 上的四种方法，然后拦截固定规则的参数分发给 Java 对应的方法处理：

- alert，可以被 webview 的 onJsAlert 监听
- confirm，可以被 webview 的 onJsConfirm 监听
- console.log，可以被 webview 的 onConsoleMessage 监听
- prompt，可以被 webview 的 onJsPrompt 监听
  prompt 简单举例说明，Web 页面通过调用`prompt()`方法，安卓客户端通过监听`onJsPrompt`事件，拦截传入的参数，如果参数符合一定协议规范，那么就解析参数，扔给后续的 Java 去处理。这种协议规范，最好是跟 iOS 的协议规范一样，这样跨端调起协议是一致的，但具体实现不一样而已。比如：`hybrid://action?arg1=1` 这样的协议，而其他格式的`prompt`参数，是不会监听的，即除了`hybrid://action?arg1=1` 这样的规范协议，`prompt`还是原来的`prompt`。

这四个方法也是各有利弊，比如:

- `alert`/`console.log`是调试最常用的，如果你要看看协议是不是写错了，但是传入协议却被拦截了。
- `confirm`和`prompt`都带返回值，`prompt`是四个里面唯一可以自定义返回值，可以做同步的交互，要比写各种回调更「顺」，但是一旦串行调用了，就会比较坑。

> prompt 是目前安卓用的比较多的 JSBridge 解决方案。

### url scheme

这个叫法不是特别贴切，scheme 是 URI 的一种格式，上文提到的 hybrid://action?arg1=1 就是一个 scheme 协议，这里说的 scheme（或者 schema）泛指安卓和 iOS 的 schema 协议，因为它通用。

拦截 url scheme 的主要流程是：Web 端通过某种方式（例如 iframe.src）发送 url scheme 请求，之后 Native 拦截到请求并根据 url scheme（包括所带的参数）进行相关操作。
缺点：

- 使用 iframe.src 发送 URL SCHEME 会有 url 长度的隐患。
- 创建请求，需要一定的耗时，比注入 API 的方式调用同样的功能，耗时会较长。

> - 有些方案为了规避 url 长度隐患的缺陷，在 iOS 上采用了使用 Ajax 发送同域请求的方式，并将参数放到 head 或 body 里。这样，虽然规避了 url 长度的隐患，但是 WKWebView 并不支持这样的方式。

- 为什么选择 iframe.src 不选择 locaiton.href ？因为如果通过 location.href 连续调用 Native，很容易丢失一些调用。

## 唤起 APP 技术

APP 外（浏览器、微信等）调起 APP 自己，给 APP 进行导流。这时候就要用到 APP 的唤起技术。这里有一下几种方法：

- intent：安卓
- localserver: 安卓
- Universal links: IOS 9+
- Deep link/Applink: 安卓
- smart app banner: IOS

### 安卓 intent

intent 格式示例如下：

```javascript
intent:
   HOST/URI-path // Optional host
   #Intent;
      package=[string];
      action=[string];
      category=[string];
      component=[string];
      scheme=[string];
      S.xxx=xxx
   end;
```

- 第一部分：host 和 path 是跟 url 无异
- 第二部分：#intent 到 end 是完整的 intent，包含了调起的 app 包名，action 等是常用的配置项

因为 Intent 不仅仅是调起 APP，而是安卓客户端内部模块通信也会用，所以权限很大，一般浏览器都给封掉了。

### 安卓 localserver

启动一个本地 server，端口号是：8888，那么在手机上，网页就可以通过：http://127.0.0.1:8888 访问这个 server，server 接收到请求就可以进行一些 native 的操作，对于需要回调数据的，就通过返回请求内容来执行，比如：

- 获取个定位信息，js 执行\$.get('http://127.0.0.1:8888/getGeoLocation?callback=cbname')
- server 收到请求之后，调用 native 方法，获取 GPS 的定位信息，然后将数据通过 response：window.cbname&&cbname({xxx})给页面返回定位数据

> - 如果控制不好权限，因为 localserver 是一直后台守候的，容易被利用，比如提权获取通讯录、甚至给通讯录发短信、容易造成蠕虫攻击

- 另外安卓各种安全软件，都会清理内存和后台程序，很容易被干掉进程。浏览器也会封杀本地 server 调起，碰见 127.0.0.1 的请求就直接拦截。

### Universal links / Deep link / Applink

这三个是官方推荐的调起方法，调起协议格式也是可以统一的，比如前文提到的 hybrid://action?arg1=xxx 这类 scheme 协议就是。这样可以统一安卓和 iOS 调起和 JSBridge 通信。

#### Universal Links

iOS 9 新出的一个功能，需要在 App 内声明一个 https 域名（ul.test.com），然后在该网站根目录放置 apple-app-site-association 文件，文件指明了转发规则，例如：

```javascript
    {
        "applinks": {
            "apps": [],
            "details": [
            {
                "appID": “xxx.com.baidu.SomeApp”,
                "paths": ["*"]
            }
            ]
        }
    }
```

当 APP 安装成功之后，会下载这个文件，明确知道遇见 ul.test.com 的域名的 URL 时候，会把这个 URL 扔给你的 APP，让你去解析，APP 拿到这个 URL 就可以解析出来需要做什么事情。

Universal Link 是 iOS 9+的底层实现，所以在任何地方都可以直接调起 APP，不受微信这类封闭 APP 的限制。

#### Deep link / Applink

Deep link 是安卓一开始推出的，主要用于搜索调起 APP，后来推出 Applink，实际是 Deep link 的升级版。

这里需要提到微信的 APPlink，毕竟微信作为 SuperApp，是很大的分发资源，微信有自己的分发方法，安卓内可以申请微信的 APPlink，跟 Universal link 一样，也是一个域名下面的 URL，符合一定规则就由微信（ios 是底层系统）扔个对应的域名 APP 进行解析。

### smart app banner

在页面的 head 中添加下面 meta，在 Safari 浏览器中就会出现下面的 banner

```javascript
    <meta name="apple-itunes-app" content="app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL">
```

## 总结

Hybrid 是一种连接 H5 跟 NA 的思路，即可以快速迭代 H5 功能，又可以有 NA 的体验，是混合开发的典型开发模式。
**JSBridge 的最佳实践**

- 官方推荐的方法
- 跨平台通用
- 安全可靠
- 约定大于配置的原则
- 协议规范都使用：hybrid://action/method?arg1=xxx&arg2=xxx
- iOS 使用 Universal Link 和 UIWebview 的 delegate
- 安卓使用 shouldOverrideUrlLoading 和 Applink

以上就是 Native 和 H5 间的通信原理，在不同端的代码实现也有示例，下一篇是讲解怎么封装一个自己通用的 JSBridge。

## 参考

[JSBridge 深度剖析](https://yq.aliyun.com/articles/72774)
[H5 和 Native 交互原理](https://dailc.github.io/2017/12/24/quickhybrid_native2h5interaction.html)
[JSBridge 的实现](https://dailc.github.io/2017/12/24/quickhybrid_jsbridge.html)
[JSBridge 实战](https://juejin.im/post/5bda6f276fb9a0226d18931f#heading-11)
[JSBridge 的原理](https://juejin.im/post/5abca877f265da238155b6bc#heading-11)
