---
title: Chrome浏览器安装Vue插件方法
date: 2021-02-19 18:04:37
tags: [工具]
---

1. 首先去github下载vue.zip文件插件下载地址：https://github.com/vuejs/vue-devtools
```js
git clone https://github.com/vuejs/vue-devtools.git vue-devtools
```

2. 查看当前分支是不是master分支, 如果不是切换到master分支
```js
git checkout master
```

3. 下载cnpm,因为vue插件要通过cnpm下载
```js
npm install -g cnpm -registry=https://registry.npm.taobao.org
```

4. 定位到刚才下载的vue插件目录里，之后输入`cnpm install`安装依赖

5. 通过`npm run build`编译

6. 接着修改manifest.json 里persistent 字段为true

7. 在Chrome中打开地址`chrome://extensions/`  将vue-devtools-dev\shells\chrome文件夹拖入Chrome中

至此插件才算安装完成

