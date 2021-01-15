---
title: npm常用命令
date: 2020-10-10 10:01:52
tags: [npm]
---
[npm官方中文文档](https://www.npmjs.cn/getting-started/what-is-npm/)

#### 安装

```js
npm i `包名` -g // 全局安装

npm i `包名` -dev //

npm i `包名` --save-dev
```

#### 查看

```js
npm -v // 查看安装版本

npm ls `包名` // 查看该安装包版本

npm ls `包名` -g // 查看全局安装包版本

npm view `包名` dependencies // 查看安装包依赖关系

npm list // 查看已安装包列表

```
