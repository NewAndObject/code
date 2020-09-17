---
title: 博客搭建
date: 2020-08-28 19:00:00
tags: 
- vuepress博客搭建-1
categories: 
- 学习笔记
# keys: #密码
# - '20729edcf2c71f4714309b8b64a3980c'
# publish: true #是否公布
---
## 下载Node.js 和vs.code，并安装
1. Node.js[win64位下载连接](https://vscode.cdn.azure.cn/stable/a0479759d6e9ea56afa657e454193f72aef85bd0/VSCodeUserSetup-x64-1.48.2.exe)
2. vs.code[win64位下载连接](https://nodejs.org/dist/v12.18.3/node-v12.18.3-x64.msi)
## vuepress博客生成步骤
### 静态文件生成
1. 打开cmd控制台
2. 输入npx @vuepress-reco/theme-cli init my-blog(自定义文件名)
3. 控制台访问生成的文件夹路径位置，在输入npm install
4. 打开vs.code访问生成的文件
5. 右键文件，选择open in Intergrated Tarminal
6. 在下面的TERMINAL里输入npm run build 生成pubil静态界面
### GitHub静态网站部署方法
1. 在[GitHub网站](https://github.com/)注册一个账户
2. 在账户里创建一个public的仓储，并创建一个CNAME用于网站访问（如果有域名，就需要将域名放在Custom domain里,并且CNAME里的值改成域名）
3. 使用svn将静态网站文件上传到这个仓储里
4. 最后访问CNAME里的值即可
