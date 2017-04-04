title: webpack性能优化
date: 2017-03-30 10:50:20
banner: https://future-team.github.io/blog-resources/imgs/webpack_optimization/banner_webpack.jpg
thumbnail: https://future-team.github.io/blog-resources/imgs/webpack_optimization/banner_webpack.jpg
tags:
- webpack
- 性能优化
- 陈小饼
---

Webpack是前端模块加载及资源打包工具。

> webpack takes modules with dependencies and generates static assets representing those modules.

### 为什么要优化?
随着Webpack的普遍，越来越多的开发者反映构建慢、bundle包大等问题。虽然Webpack可以通过简单的配置完成琐碎的文件编译、打包，但是如果每次改动到看到效果、每次打包完成都需要消耗大量的时间，最终换来一个或多个体积巨大的js文件，开发者对Webpack可能会渐渐失去信心，所以了解Webpack性能优化在所难免。

举个例子，一个业务项目中，一般需要如babel的编译包，如react或其他的底层框架包，如less/sass的样式编译包，如css/style的样式加载包，如文件加载包，当然还有webpack本身，如果配合其他工具还需要gulp/grunt及其插件包，不算测试相关的模块和项目中用到的其它模块，林林总总有10+个模块。花费时间多并不是模块多这个事实，而是处理过程中不同的模块对不同类型且大量的文件搜寻匹配/编译/打包工作。

<!-- more -->

### 如何优化?
#### 减少安装时间
这部分跟Webpack无关，跟开发者的包管理工具和依赖模块有关。
##### 1. 包管理工具
npm火了一阵之后开始被各种诟病，npm容易导致在多依赖关系中某些依赖没有指定版本号，导致拉取到的版本不一致，16年10月出了另一个模块管理工具`Yarn`，yarn可以缓存装过的包实现离线安装，安装速度快，也解决了版本号不确定的问题。

具体关于yarn：[Yarn-模块管理工具](http://uedfamily.com/2017/01/11/cab/Yarn-%E6%A8%A1%E5%9D%97%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7/)

##### 2. 依赖模块
每次开发完成可以检查一下是不是所有save到package.json以及import/require到文件中的模块都用到了，尽量减少可避免的时间消耗和代码冗余。

除此，确定放在`dependencies`的依赖是否可以转移到`devDependencies`中放置，避免增加下载包的大小。

#### 减少build&rebuild时间
开发中一般都会开启`watch`的模式，每一次改动可以即时反映到浏览器上，这部分时间的消耗是至关重要的。

##### 1. babel-loader优化
1. 用`exclude`或`include`限制babel编译的文件范围。
2. babel-loader可以缓存处理过的模块，`cacheDirectory`对于没有修改过的文件不会再重新编译。

```code
{
    test: /\.jsx?$/,
    loaders: ['react-hot', 'babel-loader?cacheDirectory'],
    exclude: /node_modules/
}
```

##### 2. resolve.root VS resolve.modulesDirectories
`root`是包含你的模块的绝对路径，`modulesDirectories`是一个目录数组，将解析当前目录及祖先目录，因此，只有在如`node_modules`的复杂路径下考虑使用`modulesDirectories`，大部分情景下使用`root`就足够了。
```code
resolve:{
    root: path.resolve(__dirname, './src'),
    modulesDirectories: ['node_modules','bower_components'],
    extensions: ['', '.js', '.jsx']
}
```

##### 3. resolve.alias 
`alias`把用户的一个请求重定向到另一个路径，减少路径搜索的成本。
```code
resolve:{
    alias: {
    	xyz: "/absolute/path/to/file.js"
	}
}
```

##### 4. module.noParse
`noParse`忽略对已知文件的解析，确定一个模块中没有其它新的依赖 就可以配置这项，webpack 将不再扫描这个文件中的依赖。
```code
module:{
    noParse:[/jquery/]
}
```

##### 5. DLLPlugin & DllReferencePlugin
DLLPlugin通过前置依赖包的构建，来提高构建效率。打包dll的时候，Webpack会将所有包含的库做一个索引，写在一个manifest文件中，而引用dll的代码在打包的时候，只需要读取这个manifest文件就可以了。

Dll打包以后是独立存在的，只要其包含的库没有增减、升级，hash也不会变化，因此线上的dll代码不需要随着版本发布频繁更新。

首先新建一个dll的配置文件如`webpack.dll.config.js`，entry只包含第三方库：
```code
module.exports = {
	entry: { 　// 只包含需要单独打包的依赖包
		vendor: ['react', 'react-dom', 'classnames']
	},
	output: {
		path: path.join(process.cwd(),'dist'),
		filename: '[name].[chunkhash].js',
		library: '[name]_[chunkhash]',
	},
	plugins: [
		new webpack.DllPlugin({
			path: 'manifest.json',
			name: '[name]_[chunkhash]',
			context: __dirname,
		}),
	],
};
```

```code
// manifest.json 
{
  "name": "vendor_45a4e5523d37b29e930b",
  "content": {
    "../node_modules/.npminstall/react/0.14.8/react/react.js": 1,
    "../node_modules/.npminstall/react/0.14.8/react/lib/React.js": 2,
    ...
    "../node_modules/.npminstall/react-dom/0.14.8/react-dom/index.js": 158,
    "../node_modules/.npminstall/classnames/2.2.5/classnames/index.js": 159
  }
}
```

在普通配置文件如`webpack.config.js`中plugin增加`DllReferencePlugin`，这里的manifest对应刚刚的`manifest.json`：

```code
plugins: [
	...
    new webpack.DllReferencePlugin({
      context: __dirname,
      manifest: require('./manifest.json'),
    }),
  ]
```

##### 6. happypack
happyPack利用了loader多进程去处理文件，同时还利用缓存来使得重新构建更快。
工作流大概如下图：
![happypack_workflow](https://future-team.github.io/blog-resources/imgs/webpack_optimization/happypack_workflow.png)

详细可以参考：[happypack](https://github.com/amireh/happypack)

#### 减少bundle包体积

##### 1. 提取公共代码块
Webpack提供了`webpack.optimize.CommonsChunkPlugin`的方法，对于多个输出文件之间重复使用的代码，整体减少了bundle包的大小，同时缓存固定依赖。
```code
entry: {
    ...
    vendor: ['react', 'react-dom', ...]
},
plugins: [
    new webpack.optimize.CommonsChunkPlugin({
        names: ['vendor', 'manifest'],
    }),
]
```

##### 2. 文件压缩
Webpack提供了UglifyJS来压缩js文件，一般会减少`60%左右`的体积，有人推荐使用`webpack-parallel-uglify-plugin`来压缩代码能够得到更快速的体验。

##### 3. 灵活使用externals
最近的工作是开发提供给业务的React UI库，之前没有去仔细研究过`externals`，打包之后发现不压缩的文件有1兆＋，对业务中的模块下载和打包肯定产生一定影响了。
认真开始研究性能优化之后发现问题出在`react-addons-css-transition-group`上，源码只有简单的一句：

```code
module.exports = require('react/lib/ReactCSSTransitionGroup');
```

把整个react模块又打包进来了，在`externals`中加入`react-addons-css-transition-group`之后打包体积剩下300+，非常显著。`externals`并没有那么智能去判断这个主模块是否已经放在不打包的行列里了，因此如使用到`react/lib/ReactDOM`也是需要单独列出来。

```code
externals:[{
    'react': 'React',
    'react/lib/ReactDOM': 'ReactDOM',
    'react-addons-css-transition-group': 'ReactCSSTransitionGroup'
}]
```

另外，`externals`对应的值需要正确对应当前模块暴露的模块名，否则在通过script引入的情况下无法正常使用：
```code
<script src="https://npmcdn.com/react@0.14.2/dist/react.js"></script>
<script src="https://npmcdn.com/react-dom@0.14.3/dist/react-dom.js"></script>
<script src="./../dist/bundle.js"></script>
```

那么在业务项目中使用的时候，当前模块找不到react相关的依赖会向外在项目的`node_modules`里面寻找依赖模块。

### 总结
林林总总提了一些优化点，部分是前人总结的优化点。最终还是要针对项目的需求来做优化。

参考：[如何 10 倍提高你的 Webpack 构建效率](http://eternalsky.me/ru-he-10-bei-ti-gao-ni-de-webpack-gou-jian-xiao-lu/)

