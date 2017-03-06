title: Promise-JavaScript异步编程
date: 2017-03-05 14:57:24
banner: https://future-team.github.io/blog-resources/imgs/promise/banner_promise.jpg
thumbnail: https://future-team.github.io/blog-resources/imgs/promise/banner_promise.jpg
tags:
- promise
- 异步编程
- 陈小饼
categories:
- promise
- 异步编程
---

Promise是异步编程的一种解决方案。

### Why Promise?
Javascript是单线程执行，因而很多的操作都是异步执行的，其中Ajax就是典型的异步操作。对于异步执行传统的解决方案是回调函数，优点是简单、容易理解和使用，缺点是不利于代码的阅读和维护，代码的高度耦合使得程序结构混乱，多层回调嵌套致使流程难以追踪。

ES2015提供了Promise原生对象，能够为异步代码执行结果的成功和失败分别绑定对应的处理方法，可以采用链式的写法串联多个Promise对象，语法上更明晰。

基本用法如下：
```code
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

关于链式写法，借用[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)的图参考如下：
![Promise Chain](https://future-team.github.io/blog-resources/imgs/promise/promise_chain.png)

Promise有三种状态，分别是`pending`表示初始状态；`fulfilled`(或`resolved`)表示操作成功完成；`rejected`表示操作失败。状态的改变只有2种可能，从`pending`变成`fulfilled`以及从`pending`变成`rejected`，一旦状态改变，就不会再变。

<!-- more -->

以Ajax为例：
```code
var ajax = function(url){
  return new Promise(function(resolve, reject){
    var xhr = new XMLHttpRequest();
    xhr.open("get", url, true);
    xhr.responseType = "json";
    xhr.onreadystatechange = readyStateHandle;
    xhr.send();

    function readyStateHandle(){
      if(this.readyState !== 4) return;
      if(this.status === 200){
        resolve(this.response);
      }else{
        reject(new Error(this.statusText));
      }
    }
  });
}
```

目前Chrome、Firefox、Safari的高版本已经支持Promise。

### Promise API
#### .then()
为Promise实例添加状态改变时的回调函数。第一个参数是`fulfilled`状态的回调函数，即promise被成功完成执行的回调；第二个参数(可选)是`rejected`状态的回调函数，即promise被拒绝或者出现任何错误、异常的过程中被捕捉到时执行的回调，可以不传对抛出的错误不做处理。

继续上面的Ajax的例子，用法如下：
```code
ajax("./posts.json").then((json) => {
  console.log('Contents:', json);
}, (error) => {
  console.error('出错了', error);
});
```

`then`方法返回的是一个新的Promise实例，因此可以使用链式写法。

第一个`fulfilled`回调函数的返回有2种情况：第一种是返回某个固定的值，将会作为参数传入第二个`then`方法的`fulfilled`回调：
```code
ajax("./posts.json").then(json => {
  // 返回一个string
  return json.nextJsonUrl;
}).then(
  url => console.log(url),  // 得到json.nextJsonUrl对应的值，一个string
  error => console.error('出错了', error)
)
```

第二种是返回一个Promise对象，则第二个`then`将会等待当前状态发生变化之后再按照结果决定调用哪个回调：
```code
ajax("./posts.json").then(json => {
  // 返回一个Promise对象
  return ajax(json.nextJsonUrl);
}).then(
  nextJson => console.log(nextJson),  // 假设成功，得到第二个json的内容
  error => console.error('出错了', error)
)
```

#### .catch()
`catch`方法用于指定发生错误时的回调函数，等同于`then(null, rejection)`，如下：
```code
ajax("./posts.json").then(
  json => {...}, 
  error => console.error('出错了', error)
)
// 等同于
ajax("./posts.json").then(
  json => {...}
).catch(
  error => console.error('出错了', error)
)
```
用`catch`能够捕获ajax()抛出的错误和`then`方法执行过程中发生的异常或错误，相对`then`用`catch`处理`rejected`状态的写法更佳。

`catch`方法返回的还是一个Promise对象，因此接下去还可以继续使用`then`方法。
```code
ajax("./posts.json").then(json => {
  return ajax(json.nextJsonUrl);
}).catch(
  error => console.error('出错了', error)
).then(
  nextJson => console.log(nextJson)
)
```
如果`catch`之前没有报错，则直接跳过`catch`执行下一个`then`方法，这时候这个`then`是否报错与前面的`catch`就无关了。若当前`catch`方法中抛出错误，在当前`catch`中是无法捕获的，需要在后面的`catch`方法中捕获/处理。

#### .all() & .race()
`all`方法和`race`方法同样是将多个Promise实力包装成一个新的Promise实例，如果参数非Promise实例，会先通过`Promise.resolve`方法转化，返回的每个成员都是Promise实例。

对于`all`方法，只有所有成员都变成`fulfilled`，`promises`的状态才会变成`fulfilled`，这时的返回值组成一个数组传递给`promises`的回调函数；只要任何一个成员的状态变为`rejected`，`promises`的状态就会变成`rejected`，第一个被`rejected`的实例的返回值传递给`promises`的回调函数。
```code
var promises = Promise.all([p1,p2,p3]).then((posts)=>{
  console.log(posts); // 假设成功，返回p1,p2,p3返回值的数组
}).catch((error)=>{
  console.error(error); // 假设失败，遍历之后返回第一个失败成员的错误信息
});
```

对于`race`方法，只要其中一个成员率先改变状态，就会作为返回值传递给`promises`的回调函数。
```code
var promises = Promise.race([p1,p2,p3]).then((posts)=>{
  console.log(posts); // 假设成功，返回成员中中最开始状态变为fulfilled的返回值
}).catch((error)=>{
  console.error(error); 
  // 假设失败，返回第一个失败成员的错误信息，区别在于只要发现有一个错误马上返回错误
});
```

#### .resolve()
`resolve`方法用于将传入的参数转化为Promise对象。
```code
Promise.resolve("Hello World");
// 等同于
new Promise(resolve => {
  resolve("Hello World");
})
```

传参可能有四种情况：
- 参数是一个Promise实例：不做任何改变返回。
- 参数是一个thenable对象：指的是具有then方法的对象，转为Promise对象，立即执行该对象的then方法。
- 参数根本不是对象：返回一个状态为Resolved的Promise对象。
- 不带任何参数：直接返回一个Resolved状态的Promise对象。

#### .reject()
`reject`方法会返回一个为`rejected`状态的Promise对象。
```code
Promise.reject("error!");
// 类似于
new Promise((resolve, reject) => {
  reject("error!");
}).catch(
  error => console.error(error) // 控制台的输出稍有不同
)
```

`reject`方法的参数会原封不动的作为错误的返回值，如：
```code
Promise.reject(ajax("./posts.json"));
```
即使已经成功获取到`posts.json`的内容，控制台仍然会输出`Uncaught (in promise) ...`。

### 总结
对Promise的研究还不是很透彻，从表层看，或许只是对传统回调函数写法的一种封装，提供一种更为简单干净的写法；从某些方面看，除此以外，Promise还具有一些相对严谨的特点，如只有异步操作的结果可以决定当前的状态，且状态一旦确定，即使往下再加其它的逻辑，也不会改变当前的结果。


参考：[ECMAScript 6入门－Promise对象](http://es6.ruanyifeng.com/#docs/promise)