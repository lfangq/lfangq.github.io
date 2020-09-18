---
title: Hexo + Github actions自动部署
date: 2020-09-17 18:04:37
tags: [hexo, github]
---
### Hexo
#### 1. 安装Hexo
```js
npm install -g hexo-cli
```
#### 2. 初始化Hexo
```js
hexo init <folder>
cd <folder>
npm install
```
#### 3. 一键部署
修改根目录下_config.yml 例:
```yml
# github部署配置
deploy:
  type: git
  repo:  git@github.com:<username>/<username>.github.io.git
  branch: master
```
生成站点文件并推送至远程库
```js
hexo clean && hexo deploy
```
[Hexo文档](https://hexo.io/zh-cn/docs/one-command-deployment)

### Github
#### 1. 创建库名为<username>.github.io的公共存储库
#### 2. 配置github密钥
生成github密钥
```js
ssh-keygen -t rsa -C "<user.email>"
```
* `id_rsa.pub` 配置在Settings -> SSH and GPG keys -> New SSH key
* `id_rsa` 配置在`存储库Settings` -> Secrets -> New Secrets

#### 3. 创建自动部署脚本
分支安排
```
- master: 部署分支
- pages: 写作分支
```
在`pages分支`创建自动部署脚本,只需在`pages分支`创建`.github/workflows/<fileName>.yml`

`<fileName>.yml`文件内容 例：
```yml
name: HEXO CI
on: 
  push:
    branches: [pages]
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [10.x]
    steps:
      - uses: actions/checkout@v1
      - name: 配置node环境
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: 配置github密钥
        env:
          HEXO_DEPLOY_PRI: ${{ secrets.HEXO_DEPLOY_PRI }}
        run: |
          mkdir -p ~/.ssh/
          echo "$HEXO_DEPLOY_PRI" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name "<user.name>"
          git config --global user.email "<user.email>"
      - name: 安装依赖
        run: |
          npm i -g hexo-cli
          npm i -g hexo-deployer-git
          npm install
      - name: 发布页面
        run: |
          hexo clean
          hexo d --config _config.yml
```
#### 4.错误问题
1.问题：引用的样式没有自动部署<br>
原因：git clone <remoteUrl> themes/<主题名>下载的目录中有隐藏的.git文件，导致git push时该目录下的文件不能添加<br>
解决方法：删除.git文件

2.问题：git@github.com: Permission denied (publickey)
原因：github权限问题
解决方法：检查github密钥是否正确


