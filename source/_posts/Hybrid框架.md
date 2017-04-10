title: Hybrid框架
date: 2017-04-05 10:34:12
banner: https://future-team.github.io/blog-resources/imgs/hybrid-framework/banner_hybrid.jpg
thumbnail: https://future-team.github.io/blog-resources/imgs/hybrid-framework/banner_hybrid.jpg
tags:
- hybrid
- 陈小饼
---

### Cordova & PhoneGap 入门

`Apacha Cordova`是开源的移动开发框架，允许使用HTML/CSS/JavaScript进行跨平台开发。
`Apacha Cordova`是从`PhoneGap`中抽出的核心代码，是驱动`PhoneGap`的核心引擎。目前跨平台开发离不开PhoneGap/Cordova，所以有必要先了解一下。

> You can think of Apache Cordova as the engine that powers PhoneGap, similar to how WebKit is the engine that powers Chrome or Safari. 

推荐阅读：[PhoneGap, Cordova, and what’s in a name?](http://phonegap.com/blog/2012/03/19/phonegap-cordova-and-whate28099s-in-a-name/)

<!-- more -->

#### Cordova
##### 使用
- 全局安装cordova。
```code
$ sudo npm install -g cordova
``` 

- 用cordova创建项目。
```code
$ cordova create hello com.example.hello HelloWorld
```

- 增加平台platforms。
```code
$ cordova platform add ios --save
$ cordova platform add android --save

// 查看平台
$ cordova platform ls
```

- 安装构建项目的环境，查看环境要求，之后按照步骤搭建起来。
```code
$ cordova requirements

>>
Requirements check results for android:
Java JDK: installed 1.8.0
Android SDK: installed true
Android target: installed android-25
Gradle: installed 

Requirements check results for ios:
Apple OS X: installed darwin
Xcode: installed 8.1
ios-deploy: installed 1.9.1
CocoaPods: installed 
```

- 构建项目。
```code
$ cordova build 
// 或单独某个平台如IOS
$ cordova build ios
```

- 安装cordova插件。
```code
$ cordova plugin add cordova-plugin-camera
// 查看项目插件
$ cordova plugin ls
```

重要插件地址: [core-plugin-apis](https://cordova.apache.org/docs/en/6.x/guide/support/index.html#core-plugin-apis)
更多其他插件: [plugreg](http://www.plugreg.com/)

##### 创建插件
插件必须配置`plugin.xml`文件，`id`表示插件名，`platform`对应当前配置的平台，`js-module`对应Javascript文件路径，`header-file`和`source-file`分别对应`.h`和`.m`文件。具体参数可以参考: [plugin.xml](https://cordova.apache.org/docs/en/6.x/plugin_ref/spec.html)
```code
$ cordova plugin add https://git-wip-us.apache.org/repos/asf/cordova-plugin-device.git
```

可以通过`plugman`检验当前插件是否能正确安装到各个平台。
```code
$ npm install -g plugman
$ plugman install --platform ios --project /path/to/my/project/www --plugin /path/to/my/plugin
```

如果要写一个兼容Android/IOS的插件，需要会不同平台的语言，还是有点挑战的。具体的开发方式可以参照官网。发布直接通过npm的方式`npm publish`的方式即可，记得在发布之前新建`package.json`文件记录插件的各类信息。

#### PhoneGap
`PhoneGap`提供了桌面应用和通过`phonegap cli`来创建/操作项目，同时提供了移动端的调试工具，通过wifi连接即可。

##### 使用
- 全局安装phonegap。
```code
$ npm install -g phonegap@latest
```

- 创建项目，通过客户端创建相对会慢很多，不指定模板默认`phonegap-template-hello-world`。
```code
$ phonegap create myApp

>>
Creating a new cordova project.

Using cordova-fetch for phonegap-template-hello-world
```

- 增加平台platforms和插件plugin的命令和cordova类似，直接用cordova命令也是可以的。但在phonegap项目中在执行添加平台的命令会添加很多的常用的插件，需要等待一段时间，但是很大程度省去了安装插件的操作。
```code
$ phonegap platform add ios

$ phonegap plugin add cordova-plugin-device
```

- 将项目拖拽到phonegap客户端界面，可自定义服务的端口，然后手机端连接项目查看效果。或者通过cli开启服务，不需要安装平台(ios/android/.etc)的依赖就可以直接看到效果。
```code
$ phonegap serve

// 默认端口3000，可通过以下命令改变端口并开启服务
phonegap serve --port, -p <n> 
```

- 调用插件，以拍照为例。
```code
// html
<div id="camera">拍照<div>

// JavaScript
// 插件调用都以 `navigator.插件名.方法`
var camera = document.getElementById('camera');
camaraButton.addEventListener('click', function(){
    navigator.camera.getPicture(onLoadImageSuccess, onLoadImageFail);
}, false);

function onLoadImageSuccess(imageURI){ // 成功返回图片地址
    alert(imageURI);
}

function onLoadImageFail(message){ // 失败返回错误原因
    navigator.notification.alert("拍照失败，原因：" + message, null, "警告");
}

```

### 主流Hybrid框架对比
### OnsenUI
提供Angular 1, 2, React 和 Vue.js框架的移动端开发，还需要配合PhoneGap/Cordova实现跨平台开发。

项目地址：[https://github.com/OnsenUI/OnsenUI](https://github.com/OnsenUI/OnsenUI)

优势：
- 文档完善，提供丰富的示例。
- 提供较多的预定义组件，如Grid、Dialog、Form等常用组件。
- 提供2套样式，支持Material Design。
- 提供monaca脚手架，让项目搭建更方便。
- 多框架语言的支持：React、Angular1/2、Vue2、Meteor。
- 持续维护，更新及时。

劣势：
- 不包含但支持PhoneGap/Cordova构建。

推荐指数：🌟🌟🌟🌟🌟

#### Ionic
使用Angular构建跨平台app。

项目地址：[https://github.com/driftyco/ionic](https://github.com/driftyco/ionic)

优势：
- 提供Creator非代码环境拖拽组件构建App。
- 提供丰富的预定义组件，如Grid、Modals、Input、Menus等组件。
- 提供不同平台的样式选择和修改方式，支持Material Design。
- 完善的文档，庞大的社区。

劣势：
- 必须会使用AngularJS去完成复杂的开发。

推荐指数：🌟🌟🌟🌟🌟

#### Framework 7
全功能的HTML框架，用于构建iOS和Android应用程序。针对IOS的UI设计，如果要兼容Android需要另外使用PhoneGap/Cordova。

项目地址：[https://github.com/nolimits4web/Framework7/](https://github.com/nolimits4web/Framework7/)

优势：
- 丰富的示例、教程，容易上手。
- 丰富的组件和插件。
- 能够轻易和js框架配合使用，如React、Vue。

劣势：
- 不包含但支持PhoneGap/Cordova构建。
- 没有提供脚手架，只提供了最基础的服务。

推荐指数：🌟🌟🌟🌟

#### React Native
用React构建跨原生app，和PhoneGap/Cordov原理不一样，性能表现更流畅。

项目地址：[https://github.com/facebook/react-native](https://github.com/facebook/react-native)

优势：
- 仿原生的表现，相对流畅。
- 庞大的社区。

劣势：
- 学习曲线陡，需要适当学习客户端开发。
- 跨平台的兼容性有待考究。

推荐指数：🌟🌟🌟

### 总结
稍微普及了PhoneGap/Cordova，是目前比较成熟和稳定的实现用JavaScript开发的Hybrid框架，但是由于还是通过WebView去展现，所以在性能方面还是不如原生Native来的流畅。

对比当中混入了React Native，其实现原理和PhoneGap/Cordova没什么太大关系。目前很多人用React Native开发项目，从入门到放弃，目前还有太多兼容性和需要自定义的部分，开发成本比预期要来的高，要求开发者不仅会Web的技术，还需要IOS／Android的技术。







