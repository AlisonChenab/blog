title: Yarn-模块管理工具
date: 2017-01-11 19:39:50
banner: https://future-team.github.io/blog-resources/imgs/yarn/banner_yarn.jpg
thumbnail: https://future-team.github.io/blog-resources/imgs/yarn/banner_yarn.jpg
tags: 
- Yarn
- NPM
- 工具
- 陈小饼
---

### 简介
2016年10月11日，facebook发布新的模块管理工具Yarn。   
随着NPM的广泛使用中，开始出现安全性和性能不足的问题，如：

-   安装速度慢（虽然淘宝NPM镜像解决了国内下载慢的现象，但是不可否认使用NPM安装慢的问题）。
-   允许安装packages时执行代码，埋下安全隐患。
-   安装packages的版本不够稳定等。

为弥补存在部分缺陷的NPM，facebook和Exponent、Google和Tilde一起打造NPM的替代版本Yarn。
<!-- more -->

### Yarn vs NPM
#### yarn.lock
NPM和Yarn都是通过package.json拉取依赖的模块，以下是拉取模块的版本号规则（以1.2.2为例）：

-   指定版本号（1.2.2）：只安装指定版本
-   波浪号＋版本号（~1.2,2）：安装1.2.x的最新版本（不低于1.2.2）
-   插入号＋版本号（^1.2.2）：安装1.x.x的最新版本（不低于1.2.2）

当执行`npm install [package] --save`时，默认save的是插入号的方式，表示当前的模块至多是向下兼容的功能新增或问题修正。虽然能够手动改成安装指定版本的方式，但也不能确定当前模块依赖的模块是否是固定的，也并不是所有的模块都能按照这样的方式来更新发布，因此每次安装都有可能会拉取到一份版本不固定的模块，这个不稳定性是一个极大的隐患。

yarn.lock锁定文件解决了以上的问题，在执行yarn安装命令时自动生成，确切的记录安装模块的版本号。同样NPM中存在相似的`npm-shrinkwrap.json`，需要通过执行 `npm shrinkwrap` 手动生成。对比之下在数据格式上yarn.lock更为精简清晰。

#### 更快的模块下载
yarn下载时会进行更好的排序，最大限度的提高网络利用率，且安装过程中如果网络请求失败会进行自动重试。

以下载grunt为例，yarn耗时13.52s，NPM第一次尝试超过一分钟，再次尝试在40s左右。（以上数据跟当前网络状况有相对密切关系，为增加实验可信度，在多次尝试不同的模块的之后，yarn都有相对优异的下载速度）

除此，yarn还支持离线下载，前提是之前下过这个包。

对于安装一整个项目yarn的速度还是需要提升，但是同样也可以给yarn设置安装源：
```code
$ yarn config set registry 'https://registry.npm.taobao.org'
```

#### 更简洁的输出
yarn的输出信息添加了一些emoji表情😄，信息简单明了，NPM的输出相对冗长。

更多信息：[migrating-from-npm](https://yarnpkg.com/en/docs/migrating-from-npm)


### 使用
#### 安装
```code
// 用Homebrew安装
$ brew update
$ brew install yarn

// 用NPM安装
$ npm install yarn -g
```

#### CLI
##### 安装依赖
安装package.json的依赖
```code
$ yarn
or
$ yarn install
```

本地安装某个模块依赖
```code
$ yarn add [package] 
->$ npm install [package] --save

$ yarn add [package] [--dev/-D]
->$ npm install --save-dev [package]
```

全局安装
```code
$ yarn global add [package]
->$ npm install [--global/-g] [package]
```

##### 删除依赖
```code
$ yarn remove [package]
->$ npm uninstall --save [package]
```

##### 升级依赖
根据package.json文件重新安装到当前配置的最新版本
```code
$ yarn upgrade
->$ rm -rf node_modules && npm install
```

更多与NPM的CLI对比：[cli-commands-comparison](https://yarnpkg.com/en/docs/migrating-from-npm#toc-cli-commands-comparison)
更多CLI介绍：[cli-introduction](https://yarnpkg.com/en/docs/cli/)

### 其他
提醒一下，当开始使用yarn要记得将yarn.lock文件一起放进代码仓库中分享给小伙伴，这是准确安装的保证。

### 总结
好像一整篇文章都在赞扬yarn贬低NPM，但是一个东西做出来要让大众接受势必要比前者更加优秀。NPM的强大都见识过了，但问题也是接踵而至，yarn对此做了很多的改善，但是目前看到yarn的issue也是不断，毕竟是新东西，还需要时间的沉淀😄。

