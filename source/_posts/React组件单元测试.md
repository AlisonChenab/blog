---
title: React组件单元测试
date: 2017-01-18 11:25:02
banner: https://future-team.github.io/blog-resources/imgs/test-utilities/react_component_unit_test.jpg
thumbnail: https://future-team.github.io/blog-resources/imgs/test-utilities/react_component_unit_test.jpg
tags:
- react
- test
- 陈小饼
categories:
- react
- test
---

针对react组件的单元测试，facebook提供了Test Utilities测试工具库，Airbnb基于此封装了使用更方便的Enzyme。另外普及一下React的测试框架Jest。

### Test Utilities
#### Shallow Rendering - 浅渲染
Shallow rendering是指将组件渲染成虚拟DOM对象，本质上并没有加载进真实DOM，所以并不要求DOM环境，浅渲染只渲染组件的第一层，不涉及所有的子组件，因此在处理速度上非常快。
<!-- more -->
函数shallowRender返回浅渲染的虚拟DOM对象：
```code
import TestUtils from 'react/lib/ReactTestUtils.js';

function shallowRender(Component, props){
    const renderer = TestUtils.createRenderer();
    renderer.render(<Component {...props}/>);
    return renderer.getRenderOutput();
}
```
浅渲染适用于不需要DOM交互，不涉及子组件的情况，如简单验证属性设置是否正确，传值是否正常等。举个例子，验证Button组件phStyle设置为primary是否成功添加到className中：
```code
import React from 'react';
import assert from 'assert';
import Button from '../src/Button';

describe("<Button/>", function(){
    it('phStyle设置为primary', ()=>{
        const button = shallowRender(<Button phStyle='primary'></Button>);
        assert(button.props.className.match('primary'));
    });
});
```

#### renderIntoDocument - 将element渲染成真实的DOM节点
renderIntoDocument将组件实例化，渲染成真实的DOM对象，因此操作环境也是要求必须是真实的DOM环境。渲染成真实DOM之后，可以进行交互，也可以查看子组件的状态。举个例子，验证Switch组件改变时是否触发了onChange：
```code
// ...
import {findDOMNode} from 'react-dom';
import Switch from '../src/Switch';

describe("<Switch/>", function(){
    it('改变时触发onChange', ()=>{
        let checked = true;
        const switchs = TestUtils.renderIntoDocument(<Switch checked={checked} onChange={()=>{checked=false}} />);
        const checkbox = TestUtils.scryRenderedDOMComponentsWithTag(switchs, 'input')[0];

        TestUtils.Simulate.change(checkbox);
        assert.equal(checked, false);
    });
});
```
初始设置checked为true，由于当前是真实DOM，用到了findDOMNode函数，查找DOM结构中的当前switch节点，手动改变，根据checked数值的对比判断是否执行了onChange。

#### Simulate - 模拟用户操作
`Simulate.{eventName}(element,[eventData])`将组件渲染进真实DOM中，模拟人为操作，如点击、输入等。
```code
TestUtils.Simulate.click(node);
TestUtils.Simulate.change(node);
TestUtils.Simulate.keyDown(node, {key: "Enter", keyCode: 13, which: 13});
```

#### 节点查找
Test Utilities提供的节点查找的方法仅适用于渲染成真实DOM节点的情况，如果是虚拟DOM是通过对`props.children`进行查找。
##### 虚拟DOM的节点查找
```code
// ...
it('checkbox: label传值显示正常', ()=>{
    let label = '测试';
    const checkbox = shallowRender(<Input type='checkbox' label={label} />);

    assert.equal(checkbox.props.children[1].props.children, label);
});
// ...
```
`checkbox.props.children[1].props.children`查找的是第二个子元素的内容。

##### 真实DOM的节点查找
对于真实DOM节点，可以是JavaScript原生的一些方法，如`findDOMNode(node).querySelector(...)`。以下是Test Utilities提供的方法：
-   scryRenderedDOMComponentsWithClass：找出所有匹配指定className的节点
-   findRenderedDOMComponentWithClass：与scryRenderedDOMComponentsWithClass用法相同，但只返回一个节点，如有零个或多个匹配的节点就报错
-   scryRenderedDOMComponentsWithTag：找出所有匹配指定标签的节点
-   findRenderedDOMComponentWithTag：与scryRenderedDOMComponentsWithTag用法相同，但只返回一个节点，如有零个或多个匹配的节点就报错
-   scryRenderedComponentsWithType：找出所有符合指定子组件的节点
-   findRenderedComponentWithType：与scryRenderedComponentsWithType用法相同，但只返回一个节点，如有零个或多个匹配的节点就报错
-   findAllInRenderedTree：遍历当前组件所有的节点，只返回那些符合条件的节点

更多信息：[Test Utilities](https://facebook.github.io/react/docs/test-utils.html)

### Enzyme
Enzyme模拟了JQuery的API封装了一套快速便捷的测试工具库，熟悉JQ的开发者能够更容易上手。
#### 使用
正式使用前有几点需要注意，否则会报错。
-   首先，对于webpack的使用者，对于不同的React版本，配置中需要加上：
	```code
	/* webpack.config.js for React 0.14 */
	// ...
	externals: {
	  'cheerio': 'window',
	  'react/addons': true,
	  'react/lib/ExecutionEnvironment': true,
	  'react/lib/ReactContext': true
	}
	// ...
	```
	```code
	/* webpack.config.js for React 15 */
	// ...
	externals: {
	  'react/addons': true,
	  'react/lib/ExecutionEnvironment': true,
	  'react/lib/ReactContext': true
	}
	// ...
	```
更多信息：[enzyme-guides](http://airbnb.io/enzyme/docs/guides/webpack.html)

-   针对不同版本React的安装
	```code
	/* React 0.13 */
	npm i react@0.13 --save
	npm i --save-dev enzyme
	```
	```code
	/* React 0.14 */
	npm i --save react@0.14 react-dom@0.14
	npm i --save-dev react-addons-test-utils@0.14
	npm i --save-dev enzyme
	```
	```code
	/* React 15 */
	npm i --save react@15.0.0-rc.2 react-dom@15.0.0-rc.2
	npm i --save-dev react-addons-test-utils@15.0.0-rc.2
	npm i --save-dev enzyme
	```
更多信息：[enzyme-installation](http://airbnb.io/enzyme/docs/installation/index.html)

#### API
##### shallow
shallow方法是对Shallow Rendering的封装，同一个例子，获取`className`的方法通过`props()`方法返回。
```code
// ...
it('phStyle设置为primary', ()=>{
    const button = shallow(<Button phStyle='primary'>btn</Button>);
    assert(button.props().className.match('primary'));
});
// ...
```
同一个例子，使用`find()`方法查找子节点，`text()`方法获取文本内容。
```code
// ...
it('checkbox: label传值显示正常', ()=>{
    let label = '测试';
    const checkbox = shallow(<Input type='checkbox' label={label} />);

    assert.equal(checkbox.find('span').text(), label);
});
// ...
```

##### mount
mount将React组件加载为真实的DOM节点。和shallow的api相同，用`find()`方法查找子节点，模拟用户操作的方法封装为`simulate(enentType)`。
```code
// ...
it('改变时触发onChange', ()=>{
    let checked = true;
    const switchs = mount(<Switch checked={checked} onChange={()=>{checked=false}} />);
    
    switchs.find('input').simulate('change');
    assert.equal(checked, false);
});
// ...
```

##### render
render方法采用第三方库Cheerio，将React组件渲染成静态的HTML片段并分析其HTML代码结构，返回一个Cheerio实例对象。
```code
// ...
it('checkbox: label传值显示正常', ()=>{
    let label = '测试';
    const checkbox = render(<Input type='checkbox' label={label} />);

    assert.equal(checkbox.find('span').text(), label);
});
// ...
```
使用render方法时碰到一个错误：`_cheerio2.default.load is not a function`，还是比较普遍的，跟webpack配置有关系，修改如下：
```code
externals: {
        'jsdom': 'window',
        // 'cheerio': 'window',
        // ...
    },
    module:{
        loaders:[
            // ...
            {
                test: /\.json$/,
                loader: 'json',
            }
        ]
    }
```

更多信息：[enzyme-api](http://airbnb.io/enzyme/docs/api/index.html)

### 其他
#### Jest
Jest是facebook基于Jasmine的开发的针对React的单元测试框架。查资料的时候有些文章会对比Enzyme和Jest，让人误以为是同一种东西的不同表达形式，但其实前者是测试工具库，后者是测试框架，更多的时候，两者是配合起来使用的。
> Jest is a JavaScript unit testing framework, used by Facebook to test services and React applications.

普及到此为止，有需要之后会研究一下Jest，然后好好写一篇。

更多信息：[jest-react-tutorial](https://facebook.github.io/jest/docs/tutorial-react.html)

### 总结
其实这一篇介绍的东西很少，却写了很多的例子，还记录了一些使用过程中让人困扰的报错。目前测试算是有一套比较完善的机制，但是总的来说东西比较旧比较少，查到的大部分资料都是年份比较久远的。😄

