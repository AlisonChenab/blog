---
title: Web Components
date: 2018-01-22 15:33:30
tags:
- web components
- 陈小饼
---

### 什么是Web Components
> Web components are a set of web platform APIs that allow you to create new custom, reusable, encapsulated HTML tags to use in web pages and web apps.

简单理解，Web Components提供了一种方法自定义HTML标签或重写HTML标签属性，使满足基本的外观和交互。

举个例子，自定义`ph-button`标签，默认灰色边框，可通过phType修改不同的样式，点击出现弹框或Tooltip，都可以定义在组件内部，且使用方式和普通标签一致`<ph-button>按钮</ph-button>`。

<!-- more -->

### 为什么需要Web Components
流行的语言框架/库很多，组件库如果单纯满足其中一种语言环境，假设换了一种框架需要重写一套，而为了用户使用的便利，需要确保API和交互一致，这其中需要不断的沟通和跟进，损耗了大量人力。

为此，Web Components很好的解决了以上的问题，不管是使用React、Angular或者其他，抽离出其中的核心逻辑，在此基础上用语言框架/库包装一层即可。此后维护组件库大部分的开发量集中在核心库上，只需修改一次，也不用再考虑一致性的问题。


### 使用
Web Components由4个主要技术组成：
- 自定义元素（Custom Elements）
- 影子DOM（shadow DOM）
- HTML模板（HTML Template）
- HTML引用（HTML Import）

以下详细介绍

#### Custom Elements
Custom Elements即自定义元素，能够自定义/重定义元素的参数、行为和外观。简要的提一句，Custom Elements存在`v0`和`v1`版本，虽然某些概念是一样的，但API存在重大差异。其中`v0`版本是chrome浏览器独有的语法，已经不作为标准存在，而`v1`版本是现行标准并获得各大浏览器厂商的支持。

（以下提到的都是`v1`标准规范的版本）

##### 1. Autonomous Custom Elements
Autonomous Custom Elements指完全自定义的新的元素，在原生HTML标签中不存在的元素。

一个简单的例子，继承类HTMLElement，通过传递attributes设置class。最后通过`customElements.define()`定义元素。
```code
class PhButton extends HTMLElement {
    constructor() {
        super()

        this.defaultClass = 'ph-button'
        this.classMap = {
            primary: 'primary',
            info: 'info',
            success: 'success'
        }
    }

    connectedCallback(){
        this._compile()
    }
  
    _compile(){
        this.classList.add(this.defaultClass)
        this.addClassName()
    }

    addClassName(){
        var attrs = this.formatAttributes(this.attributes)

        for(var i in attrs){
            var attr = this.classMap[attrs[i]]
            if(attr){
                this.classList.add(`${this.defaultClass}-${attr}`)
                this.removeAttribute(i)
            }            
        }
    }

    formatAttributes(attrs){
        let elemAttrs = {}
        attrs = attrs ? attrs : this.attributes

        for(let key in attrs){
            if(key && attrs[key].value){
                elemAttrs[attrs[key].nodeName] = attrs[key].value
            }
        }

        return elemAttrs
    }
}

customElements.define('ph-button', PhButton)
```
调用：
```code
<ph-button>按钮</ph-button>
<ph-button phstyle='primary'>按钮</ph-button>
```
效果图：
![web-components-01](https://alisonchenab.github.io/images/web-components-01.png)

##### 2. Customized Built-in Element
Customized Built-in Element指拓展已有元素，在已有属性的基础上定义特殊交互形成新的元素。

```code
class PhButton extends HTMLButtonElement {
    constructor() {
        super()

        this.addEventListener('click', ()=>{
            console.log('Hello World!')
        }, false)
    }
}

customElements.define('ph-button', PhButton, {extends: 'button'})
```

目前Chrome浏览器对Built-in Element的支持度低，需要借助类似`WebReflection/document-register-element`的polyfill来辅助完成，但在`constructor`方法还需要做特别的处理，感兴趣具体可以看 [document-register-element#v1-caveat](https://github.com/WebReflection/document-register-element#v1-caveat)。

调用和Autonomous Custom Elements不同，如果使用相同的调用方式报错：
```code
<button is='ph-button'>Click Me!</button>
```
效果图：
![web-components-02](https://alisonchenab.github.io/images/web-components-02.png)

##### 3. 生命周期
- constructor
创建元素的一个实例。用于初始化状态、设置事件监听或创建`Shadow DOM`等。

- connectedCallback 
元素每次插入到DOM时都会调用。可在此阶段例如获取资源、渲染。

- disconnectedCallback
元素每次从DOM中移除时都会调用。可在此阶段做一些清理工作，如移除事件监听等。

- attributeChangedCallback
属性添加、移除、更新或替换，创建元素时也会调用它来获取初始值。但仅`observedAttributes`属性中列出的特性才会调用此回调。

- adoptedCallback
自定义元素被移入新的document。例如，调用了`document.adoptNode(el)`从iframe中的文档流中移到当前文档流中。

源码：
```code
class PhButton extends HTMLElement {
    constructor() {
        super()

        this.clickHandle = this.clickHandle.bind(this)
        document.addEventListener('click', this.clickHandle, false)
    }

    clickHandle(){
        console.log('Hello World!')
    }

    static get observedAttributes(){
        return ['value']
    }

    attributeChangedCallback(name, oldValue, newValue){ // 将获取到的value新值放入元素内部
        this.innerHTML = newValue
    }

    connectedCallback(){ // 设置属性value等于10
        this.setAttribute('value', '10')
    }

    disconnectedCallback(){ // 元素移除时一出绑定的事件
        document.removeEventListener('click', this.clickHandle, false)
    }

    adoptedCallback(){
        console.log('adoptedCallback!')
    }
}

customElements.define('ph-button', PhButton)
```

HTML：
```code
<iframe src="//172.24.232.211:8000/ph-button.html" frameborder="0"></iframe>
<ph-button></ph-button>

<button onclick='remove()'>Remove</button>
<button onClick='adopt()'>Adopt</button>
```

Javascript：
```code
// 触发disconnectedCallback
var remove = function(){
    var phButton = document.getElementsByTagName('ph-button')[0]
    document.body.removeChild(phButton)
}

// 触发adoptedCallback
var adopt = function(){
    var ele = getEle()
    if(ele){
        document.body.appendChild(document.adoptNode(ele))
    }        
}

function getEle(){
    var iframe = document.getElementsByTagName("iframe")[0],
        ele = iframe.contentWindow.document.body.firstElementChild

    if(ele){
        document.body.appendChild(document.adoptNode(ele))
        return ele
    }
    return false
}
```

##### 4. 命名规则
- 和已有的HTML标签名不重复，如`div`，和某些固有属性名不重复，如`font-face`
- 防止命名冲突，名称可采用固定的规范，如带上组件特有的前缀等，`ph-*`
- 语义性和简洁，为用户更好的辨识和使用
- 至少包含一个连字符

更多关于[`[Custom Elements]`](https://w3c.github.io/webcomponents/spec/custom).


#### Shadow DOM
> Shadow DOM is a functionality that allows the web browser to render DOM elements without putting them into the main document DOM tree. 

Shadow DOM是一种能够渲染DOM元素但不放入主文档流的功能。借助Shadow DOM，可以创建作用域DOM树，附加到DOM元素上，但与本身的子项分离开，被附着的元素称为宿主。在shadow DOM中操作都局限在宿主元素内部，外部设置的样式或其他操作无法控制同规则下的shadow DOM。

就常用的一些元素如input、video等都含有shadow DOM，如写一个平时常用的输入框：
```code
<input type='text' value='test' placeholder='please input...'／>
```
在浏览器中打开显示shadow DOM，可以看到input元素中还有一些平时不存在的内容：
![web-components-03](https://alisonchenab.github.io/images/web-components-03.png)

> **Tip:** 关于如何使用Chrome查看shadow DOM，在控制台右上角的`开发工具栏`下拉选择`Settings`出现设置面板，勾选`Elements`下的`Show user agent shadow DOM`即可。

##### 1. 创建shadow DOM

用`attachShadow()`方法创建shadow DOM，通过innerHTML填充shadowRoot，如：
```code
<div id='box'>盒子</div>
...

var box = document.getElementById('box')
var shadowRoot = box.attachShadow({mode: 'open'})

shadowRoot.innerHTML = `
    <span id='shadow'>shadow DOM</span>
`
```

给自定义元素创建shadow DOM：
```code
class PhButton extends HTMLElement {
    constructor() {
        super()
        
        const shadowRoot = this.attachShadow({mode: 'open'})
        shadowRoot.innerHTML = `
            <span id='shadow'>shadow DOM</span>
        `
    }
}

customElements.define('ph-button', PhButton)
```

这时候shadow DOM已经可以显示，但是存在一个问题，原本在元素内部的子项没有渲染，需要`slot`元素来指定子项的位置，如放置于shadow DOM后面，这样就可以正常显示子项。还可以通过设置slot的`name`来放置指定的元素，如果不设置name默认显示所有子项。

```code
<div id='box'>
    盒子
    <span slot='test'>test</span>
</div>

...
var box = document.getElementById('box')
var shadowRoot = box.attachShadow({mode: 'open'})

shadowRoot.innerHTML = `
    <span id='shadow'>shadow DOM</span>
    <slot name='test'></slot>
`
```

##### 2. 设定样式

可在shadow DOM中直接设置style，在该区域内设置的样式和页面其他位置的元素没有冲突。

- 可通过`:host`指定当前宿主元素的默认样式，从外部给宿主定义的样式可覆盖`:host`设置的样式，如：
```code
...
shadowRoot.innerHTML = `
    <style>
        :host{
            font-weight: 800;
        }
        #shadow{
            color: red;
        }
    </style>
    <span id='shadow'>shadow DOM</span>
    <slot name='test'></slot>
`
```

- 可通过`:host-context(宿主选择器)`对不同状态的宿主进行样式定制化，如：
```code
<div id='box' class='black-theme'>盒子</div>

...
<style>
    :host-context(.black-theme){
        background-color: #333;
        color: #fff;
    }
</style>
```

- 可通过`::slotted(子项选择器)`对子项元素设置样式，但是只对顶层子元素生效，如：
```code
<div id='box'>盒子<span id='test'>test</span></div>

...
<style>
    ::slotted(#test){
        color: red;
    }
</style>
```

##### 3. 事件

给shadow DOM添加事件，通过`宿主元素.shadowRoot`可以定位到当前的影子节点，直接使用寻常查找DOM节点的方式不可行。
```code
<div id='box'>盒子</div>

...
var box = document.getElementById('box')
var shadowRoot = box.attachShadow({mode: 'open'})

shadowRoot.innerHTML = `
    <span id='shadow'>shadow</span>
    <slot></slot>
`

var shadow = box.shadowRoot.querySelector('#shadow')
shadow.addEventListener('click', function(e){
    e.stopPropagation() // 在shadow DOM中同样存在冒泡的问题
    console.log('shadow DOM!')
}, false)
```

提供了`slotchange`事件，监听子项添加或移除，如：

```code
...
var slot = box.shadowRoot.querySelector('slot')
slot.addEventListener('slotchange', function(e){
    console.log(e)
})
```

#### HTML Template

以上示例的代码稍显得有些凌乱，可以借助`<template>`元素创建模板，声明DOM片段，如：
```code
<template id="ph-button-template">
    <style>
        #shadow{ 
            color: red; 
        }
    </style>
    <span id='shadow'>shadow</span>
</template>

...
class PhButton extends HTMLElement {
    constructor() {
        super()

        let shadowRoot = this.attachShadow({mode: 'open'})
        const t = document.querySelector('#ph-button-template')
        const instance = t.content.cloneNode(true)
        shadowRoot.appendChild(instance)
    }
}

customElements.define('ph-button', PhButton)
```

template元素会产生一个`#document-fragment`节点，在页面中并不渲染。但这仅是作为插入一段DOM的推荐写法，并不是Web Component必须使用的。
目前在各浏览器都得到了支持。

#### HTML Import

可通过`link`标签在一个html文件中引入另一个html文件，设置ref为`import`，如：
```code
<link rel='import' href='./ph-button.html' id='temp'>

... 
var temp = document.getElementById('temp')
console.log(temp.import)
```

输出的`temp.import`是引入文件完整的document，从而达到`iframe`的效果。但目前兼容性还不够乐观。

### 其他
目前有一些封装了Web Components的JavaScript库，如Polymer、X-Tag等，其中Polymer是大厂Google的产物。

### 参考
[自定义元素 v1：可重用网络组件](https://developers.google.com/web/fundamentals/web-components/customelements?hl=zh-cn)
[Custom Elements](https://w3c.github.io/webcomponents/spec/custom/)
[Shadow DOM v1：独立的网络组件](https://developers.google.com/web/fundamentals/web-components/shadowdom?hl=zh-cn)