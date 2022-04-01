---
title: "微信小程序开发简介"
date: 2019-05-06T15:44:07+08:00
draft: false
tags:
    - MiniProgram
authors:
    - zhaolei
# weight: 1
---

<!--more-->
{{< toc >}}

[演讲视频链接](https://cdn.devops.shinetechsoftware.org/training/190506-zhaolei-wx-miniprogram.mp4)

## 小程序介绍

### 微信小程序是什么

- 移动端应用，运行依赖于微信客户端提供的宿主环境
- 同时包含 Native 和微信自身的组件及 API
- 使用特有的语言、框架进行开发
- 提供了 IDE 及 CLI，用来开发、调试、测试、发布等

### 微信小程序的特点

#### 轻量易得

小程序无需打包，无需下载安装。可以通过微信（或企业微信）的小程序入口唤起、公众号自定义菜单或模板消息、 App 分享消息卡片唤起等多个场景（[场景值][scene]）唤起。

#### 功能丰富

同时兼备了微信和原生的多种能力。微信如：拍摄、录音、语音识别、二维码、地图、支付、分享、卡券、发票等。原生如：Wi-Fi、蓝牙、联系人、电话、剪切板、网络、加速计、陀螺仪等。

#### 微信生态

- 小程序与其他移动应用、网站应用及公众号统一并共享用户信息

  小程序绑定微信开放平台帐号后，可与帐号下的其他移动应用、网站应用及公众号打通，通过 UnionID 机制满足在多个应用和公众号之间统一用户帐号的需求。

  场景：可以用在支持微信注册及登录的应用中

  > [**UnionID 机制说明：**][unionid] 如果开发者拥有多个移动应用、网站应用、和公众帐号（包括小程序），可通过 UnionID 来区分用户的唯一性，因为只要是同一个微信开放平台帐号下的移动应用、网站应用和公众帐号（包括小程序），用户的 UnionID 是唯一的。换句话说，同一用户，对同一个微信开放平台下的不同应用，UnionID 是相同的。用户的 UnionID 可通过调用“获取用户信息”接口获取。

  > **Tips**  
  > 如何获取 UnionID？

* 小程序与其他移动应用、网站应用及公众号相互跳转

  ![小程序与其他应用相互跳转][mp-jump]

  - [公众号 - 小程序][official account]

    需要在公众号的微信公众平台`小程序管理`中关联小程序。公众号关联小程序后，可在图文消息、自定义菜单、模板消息等功能中跳转小程序。

  - 小程序 - 公众号

    需要在小程序的微信公众平台中设置相关公众号。公众号会在小程序的关于页显示。从而实现引流的目的。

    > **Tips**  
    > 使用场景：由于小程序仅能在某些用户触发如支付、提交表单等交互行为时才能推送模板消息。可将重要消息以模板消息的形式推送到关联了小程序的服务号（前提是用户订阅了此服务号）。再从消息跳转到小程序某页面。

  - App - 小程序

    有两种方法：

    需要发起分享的 App 与小程序属于同一微信开放平台帐号。可以分享小程序类型消息至微信会话，通过点击分享卡片跳转小程序。

    需要在同一微信开放平台帐号，或者发起关联小程序操作。移动应用可以直接拉起小程序。

  - 小程序 - App

    小程序不能打开任意 APP，只能 `跳回`到由上述两种方法唤起小程序的 APP。且需要用户主动触发才能打开 APP，所以不由 API 来调用，需要用 `open-type` 的值设置为 `launchApp` 的 `<button>` 组件的点击来触发。

    > **Tips**  
    > 使用场景：分享到微信的消息类型，可以是比链接功能更强大的小程序页面。而且，如果分享到微信的是链接类型。链接默认会在微信内置浏览器中打开。而微信内置浏览器并不支持 App 跳转。但如果分享的是小程序类型，可以实现这一功能。

  - 小程序 - Web 页面

    需要先将要访问的 web 页面域名添加到小程序微信公众平台的业务域名中。然后在小程序组件<web-view>中访问页面，并且可以通过 `JSSDK` 与小程序通信。

    > **Tips**  
    > 如何保持登录状态？可以将 token 通过 url 参数传入；也可以搭建一个中间服务，传入要跳转的 url 和 code，中间服务通过 code 得到 session，再返回 302 重定向地址

  - Web 页面 - 小程序

    Web 页面可使用微信 JSSDK 提供的接口返回小程序页面。并可通过 postMessage 向小程序通信。

    场景：Web 页面能够嵌入小程序，提供了极大的灵活性。我们可以复用已有的移动端 H5 页面。可以将一些变更频繁的页面通过 web-view 实现，达到快速部署的目的。

  - 小程序间跳转

    需要先在配置文件中声明需要跳转的小程序 appId 列表。使用 `wx.navigateToMiniProgram` 接口跳转到其他小程序，再通过`wx.navigateBackMiniProgram`跳回

    场景：可以是同一公司的多个功能独立、彼此关联的小程序

  > **Tips**  
  >  - 小程序不能分享朋友圈
  >  - 不支持消息推送

#### 新的框架

前端怎么又岀新框架了

## 框架浅析

### 项目结构

小程序由配置代码 JSON 文件、模板代码 WXML 文件、样式代码 WXSS 文件以及逻辑代码 JavaScript 文件组成。

小程序包含一个描述整体程序的 app 和多个描述各自页面的 page。

![mp-structure]

一个小程序主体部分由三个文件组成，必须放在项目的根目录，如下：

| 文件                | 必填 | 作用                                                                        |
| ------------------- | ---- | --------------------------------------------------------------------------- |
| app.js              | 是   | 小程序逻辑                                                                  |
| app.json            | 是   | 全局配置（包括了小程序的所有页面路径、界面表现、网络超时时间、底部 tab 等） |
| project.config.json | 是   | 开发工具配置（例如界面颜色、编译配置等）                                    |
| app.wxss            | 否   | 小程序公共样式表                                                            |

一个小程序页面由四个文件组成，分别是：

| 文件 | 必填 | 作用       |
| ---- | ---- | ---------- |
| js   | 是   | 页面逻辑   |
| wxml | 是   | 页面结构   |
| wxss | 否   | 页面样式表 |
| json | 否   | 页面配置   |

### 架构解析

小程序的运行环境分成渲染层（以下用 AppView 代指）和逻辑层（以下用 AppService 代指）。分别由 2 个线程管理：AppView 的界面使用了 WebView 进行渲染；AppService 采用 JsCore 线程运行 JS 脚本。多个界面，对应多个 WebView 线程。WXML 模板和 WXSS 样式工作在 AppView，JS 脚本工作在 AppService。AppService 和 AppView 通过 WeixinJSBridge 经由微信客户端进行通讯。AppService 把数据变化通知到视图层，触发视图层页面更新，视图层把触发的事件通知到 AppService 进行业务处理。

WeixinJSBridge 主要接口：

```js
var t = {
  // 调用Native API
  invoke: function(e, t, n) {
    return WeixinJSBridge.invoke(e, t, n);
  },
  // 订阅Native消息
  on: function(e, t) {
    return WeixinJSBridge.on(e, t);
  },
  // 向AppView/AppService发布事件
  publish: function(e, t, n) {
    return WeixinJSBridge.publish(e, t, n);
  },
  // 订阅AppView/AppService消息
  subscribe: function(e, t) {
    return WeixinJSBridge.subscribe(e, t);
  }
};
```

![架构图][mp-core]

> **Tips**  
> 为了解决管控与安全问题,提供一个沙箱环境,阻止在 webview 上执行诸如跳转页面、操作 DOM、动态执行脚本的开放性接口。DOM 操作都是基于数据驱动

### 编译过程

#### WXML

在本地开发工具或者上传服务端后，工具 wcc 会把所有 wxml 转换成一个 JS 函数 `$gwx` ，形如：

```js
$gwx = function(path) {
  return function(data) {
    return {};
  };
};
```

传入相应的页面路径，执行函数 `$gwx('/page/index/index')` ，会返回一个新的函数，这个函数相当于这个页面 Virtual Dom 的骨架。此函数接受一个参数 `data` ,而这个 `data` 就是这个页面 Virtual Dom 的血肉。`$gwx('/page/index/index')(data)`再次调用，就会生成这个页面的 Virtual Dom，形如：

```javascript
var v_tree = {
  tag: 'wx-page',
  children: [
    {
      tag: 'wx-view',
      attr: { class: 'page-body' },
      children: [
        {
          tag: 'wx-textarea',
          attr: { autoHeight: true, bindblur: 'bindTextAreaBlur' },
          children: [],
          raw: {},
          generics: {}
        }
      ],
      raw: {},
      generics: {}
    }
  ]
};
```

`$gwx`会随其他辅助脚本一起嵌入到`page-frame.html`页面中。`page-frame.html`是每次小程序需要新建页面时，WebView 要加载的页面，然后再根据 path 和 data ，使用 Exparser 渲染出页面。

在使用 setData 更新数据时，会将 setData 合并到 data，并重新生成 Virtual Dom，比较新旧节点树，更新页面。

![v-dom][mp-v-dom]

> **Tips**
>
> - 不要频繁的去 setData
> - 不要每次 setData 都传递大量新数据
> - 使用 myData 存储与页面渲染无关的数据

**WXSS**

同样是使用一个转换工具 wcsc，将 WXSS 转换成 CSS。而其中主要的工作是把长度单位`rpx(responsive pixel)`根据不同机型转换成`px`。

![mp-wcsc][mp-wcsc]

**JS**

将所有 js 文件合并到 appservice.js 中，在小程序启动时依次执行，直接加载所有页面逻辑代码进内存。但加载时会使用 define 函数封装，调用时将包括 window,document 之后的参数赋空值。

![mp-js][mp-js]

```js
define("pages/logs/logs.js", function(require, module, exports, window,document,frames,self,location,navigator,localStorage,history,Caches,screen,alert,confirm,prompt,fetch,XMLHttpRequest,WebSocket,webkit,WeixinJSCore,Reporter,print,URL,DOMParser,upload,preview,build,showDecryptedInfo,syncMessage,checkProxy,showSystemInfo,openVendor,openToolsLog,showRequestInfo,help,showDebugInfoTable,closeDebug,showDebugInfo,__global,WeixinJSBridge){

'use strict';

Page({
  data: {
    logs: []
  },
  onLoad: function onLoad() {
    const env = {
      require,
      module,
      exports,
      window,
      document,
      frames,
      self,
      location,
      navigator,
      localStorage,
      history,
      Caches,
      screen,
      alert,
      confirm,
      prompt,
      fetch,
      XMLHttpRequest,
      WebSocket,
      webkit,
      WeixinJSCore,
      Reporter,
      print,
      URL,
      DOMParser,
      upload,
      preview,
      build,
      showDecryptedInfo,
      syncMessage,
      checkProxy,
      showSystemInfo,
      openVendor,
      openToolsLog,
      showRequestInfo,
      help,
      showDebugInfoTable,
      closeDebug,
      showDebugInfo,
      __global,
      WeixinJSBridge
    };

    console.log(env);
  }
});

```

`console.log:`

![mp-console-log][mp-console-log]

> **Tips**
>
> - window,document 等都是 undefined。因为 AppService 只能执行业务逻辑代码，并没有 DOM 和 BOM 可以操作。
> - 同时，我们要特别留意一点，所有页面的脚本逻辑都跑在同一个 JsCore 线程，页面使用 setTimeout 或者 setInterval 的定时器，然后跳转到其他页面时，这些定时器并没有被清除，需要开发者自己在页面离开的时候进行清理。

### 生命周期

**应用生命周期**

初次进入小程序的时候，微信客户端初始化好宿主环境，同时从网络下载或者从本地缓存中拿到小程序的代码包，把它注入到宿主环境，初始化完毕后，微信客户端就会给 App 实例派发 onLaunch 事件，App 构造器参数所定义的 onLaunch 方法会被调用。

| 属性           | 类型     | 描述                           | 触发时机                                                                 |
| -------------- | -------- | ------------------------------ | ------------------------------------------------------------------------ |
| onLaunch       | Function | 生命周期函数--监听小程序初始化 | 当小程序初始化完成时，会触发 onLaunch（全局只触发一次）                  |
| onShow         | Function | 生命周期函数--监听小程序显示   | 当小程序启动，或从后台进入前台显示，会触发 onShow                        |
| onHide         | Function | 生命周期函数--监听小程序隐藏   | 当小程序从前台进入后台，会触发 onHide                                    |
| onError        | Function | 错误监听函数                   | 当小程序发生脚本错误，或者 api 调用失败时，会触发 onError 并带上错误信息 |
| onPageNotFound | Function | 页面不存在监听函数             | 当小程序出现要打开的页面不存在的情况，会带上页面信息回调该函数，详见下文 |
| 其他           | Any      |                                | 开发者可以添加任意的函数或数据到 Object 参数中，用 `this` 可以访问       |

![mp-app-lifecycle][mp-app-lifecycle]

**页面生命周期**

| 属性              | 类型     | 描述                                                                               |
| ----------------- | -------- | ---------------------------------------------------------------------------------- |
| data              | Object   | 页面的初始数据                                                                     |
| onLoad            | Function | 生命周期回调—监听页面加载。可获取打开当前页面路径中的参数                          |
| onShow            | Function | 生命周期回调—监听页面显示                                                          |
| onReady           | Function | 生命周期回调—监听页面初次渲染完成                                                  |
| onHide            | Function | 生命周期回调—监听页面隐藏                                                          |
| onUnload          | Function | 生命周期回调—监听页面卸载                                                          |
| onPullDownRefresh | Function | 监听用户下拉动作                                                                   |
| onReachBottom     | Function | 页面上拉触底事件的处理函数                                                         |
| onShareAppMessage | Function | 用户点击右上角转发                                                                 |
| onPageScroll      | Function | 页面滚动触发事件的处理函数                                                         |
| onTabItemTap      | Function | 当前是 tab 页时，点击 tab 时触发                                                   |
| 其他              | Any      | 开发者可以添加任意的函数或数据到 `Object` 参数中，在页面的函数中用 `this` 可以访问 |

![mp-page-lifecycle][mp-page-lifecycle]

1. **AppView** 初始化。初始化完成后向 AppService 发送`初始化完成信号`，然后进入`等待状态`，等待 AppService 提供`初始化数据`。
1. **AppService** 初始化。初始化完成后调用自定义的 onLoad 和 onShow，然后等待 AppView 的`初始化完成信号`。onLoad 是只会首次渲染的时候执行一次，onShow 每次界面切换都会执行。
1. **AppService** 接收到 AppView 的`初始化完成信号`后，将`初始化数据`发送给 AppView，等待 AppView 完成初次渲染。
1. **AppView** 收到 AppService 提供的`初始化数据`后，渲染小程序界面，渲染完毕后，发送`首次渲染完成信号`给 AppService，并将页面展示给用户。
1. **AppService** 收到 AppView 发送来的`首次渲染完成信号`后，就进入激活状态即程序的正常运行状态，并调用自定义的 onReady。此状态下可以通过 this.setData 函数发送数据给 AppView 进行局部渲染，更新页面。
1. **AppView** 一直等待 AppService 通过 this.setdata 函数发送来的数据。只要收到，就进行局部渲染，更新页面。

![生命周期][page-lifecycle]

## 微信小程序开发工具

#### 框架选择

| 框架   | contributor | npm    | 语法        | TypeScript | star | 多端复用                    |
| ------ | ----------- | ------ | ----------- | ---------- | ---- | --------------------------- |
| native | tencent     | 不支持 |             | 支持       |      | 微信小程序                  |
| wepy   | tencent     | 支持   | Vuejs-style | 支持       | 17k  | 微信小程序                  |
| mpvue  | 美团点评    | 支持   | Vuejs       | 支持       | 17k  | 微信小程序/H5               |
| taro   | 京东        | 支持   | Reactjs     | 支持       | 18k  | 微信小程序/H5/RN/其他小程序 |
| omi    | tencent     | 支持   | React-style | 支持       | 8.6k | 微信小程序/H5/桌面 Web      |

## 坑

- package & subPackages  
  单包最大 2M;所有包最大 8M

- wx.request  
  statusCode 无论是多少，都会进入 success  
  默认是 json 格式，特殊字符会导致解析失败

- HTTPS  
  证书验证 https://myssl.com/ssl.html

- UnionID 的获取

  - 存在同主体的公众号，且该用户已关注
  - 存在同主体的公众号或移动应用，且该用户已授权登录
  - const { encryptedData } = wx.getUserInfo({ withCredentials : true})

- cicd
  小程序 CLI 工具是和 IDE 一起打包安装的。而 IDE 只有 windows 和 MacOS 两个版本。所以如果想在 linux 服务器上架设 cicd，只能通过黑科技安装第三方开发工具。而且在执行上传部署命令之前，还需要通过扫码登录。

- 原生组件
  脱离在 WebView 渲染流程外，一些 CSS 样式无法应用于原生组件，例如，不能在父级节点使用 overflow:hidden 来裁剪原生组件的显示区域；不能使用 transformrotate 让原生组件产生旋转等。  
  原生组件会浮于页面其他组件之上（相当于拥有正无穷大的 z-index 值）使其它组件不能覆盖在原生组件上展示。想要解决这个问题，可以考虑使用 cover-view 和 cover-image 组件。这两个组件也是原生组件，同样是脱离 WebView 的渲染流程外，而原生组件之间的层级就可以按照一定的规则控制。

[scene]: https://developers.weixin.qq.com/miniprogram/dev/reference/scene-list.html
[unionid]: https://developers.weixin.qq.com/miniprogram/introduction/#%E5%B0%8F%E7%A8%8B%E5%BA%8F%E7%BB%91%E5%AE%9A%E5%BE%AE%E4%BF%A1%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0%E5%B8%90%E5%8F%B7
[official account]: https://developers.weixin.qq.com/miniprogram/introduction/#%E5%85%AC%E4%BC%97%E5%8F%B7%E5%85%B3%E8%81%94%E5%B0%8F%E7%A8%8B%E5%BA%8F
[mp-jump]: mp-jump.png
[mp-core]: mp-core-2.png
[mp-structure]: mp-structure.png
[mp-v-dom]: mp-v-dom.png
[mp-js]: mp-js.png
[mp-wcsc]: mp-wcsc.png
[mp-app-lifecycle]: mp-app-lifecycle.png
[mp-page-lifecycle]: page-lifecycle.png
[page-lifecycle]: mp-page-lifecycle.png
[mp-console-log]: mp-console-log.png