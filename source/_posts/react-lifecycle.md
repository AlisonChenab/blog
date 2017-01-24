---
title: React Lifecycle
date: 2016-12-29 15:25:08
tags:
- react
categories:
- react
---
## 生命周期

More Info: [Component Specs and Lifecycle](http://reactjs.cn/react/docs/component-specs.html)

## 组件渲染经历的生命周期

### 第一次渲染：
- getDefaultProps
- getInitialState
- componentWillMount
- render
- componentDidMount

### 第二次渲染：
- getInitialState
- componentWillMount
- render
- componentDidMount

### Props改变时：
- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- render
- componentDidUpdate

### State改变时：
- shouldComponentUpdate
- componentWillUpdate
- render
- componentDidUpdate

### 组件卸载时：
- componentWillUnmont