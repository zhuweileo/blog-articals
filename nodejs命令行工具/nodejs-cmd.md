# 开发一个nodejs命令行工具
最近在学习nodejs，在掌握了一些基础知识之后，就想着如何运用它解决一些简单的问题。

## 代码地址

https://github.com/zhuweileo/gitignore-cmd

## 目标
平时初始化项目工程的时候，总是需要添加`.gitignore`文件，我之前总是手动复制粘贴一个写好的模板，虽然也不算麻烦，但是作为懒癌晚期，我连复制粘贴也不愿意，于是我就想用nodejs写一个命令行工具，直接输入一个命令，就在当前文件夹内生成`.gitignore`文件。其实就是让nodejs帮你复制粘贴。

## 项目目录

![1548755955537](G:\自己的项目\blog-articals\nodejs命令行工具\assets\1548755955537.png)

##  主要功能代码

**getIgnore.js**

```js
var https = require('https');
var fs = require('fs');

var url = 'https://raw.githubusercontent.com/zhuweileo/my-git-ignore/master/.gitignore';

module.exports = function getIgnore(){
  https.get(url, function (res) {
    res.setEncoding('utf8');

    let rawData = '';

    res.on('data', (chunk) => {
      rawData += chunk;
    });
    res.on('end', () => {
      fs.writeFileSync('.gitignore',rawData)
    });
  }).on('error',function (e) {
    console.log(e)
  })
}
```

实现逻辑很简单：

- 通过https请求，从我的git仓库中获得.gitignore文件
- 将获得文件内容写入到当前目录

为什么从远程请求？

这样我每次都可以取到最新的.gitignore文件。一旦.gitignore文件更新，不需要更新命令行工具。

##package.json

```
{
  "name": "zw-gi",
  "version": "0.0.1",
  "description": "快速生成gitignore的命令行工具",
  "main": "getIgnore.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "朱维",
  "license": "ISC",
  "bin": {
    "zw-gi": "bin/zw-gi"
  }
}
```

这里的关键点是**bin**字段：

```
 "bin": {
    "zw-gi": "bin/zw-gi"
  }
```

意思是：当你在命令行中执行`zw-gi`命令时，nodejs会执行`bin/zw-gi`文件中的代码

当通过`npm install zw-gi  -g`命令安装你的nodejs命令行工具时，npm会根据这个 **bin**字段，生成可执行程序。不同的系统生成的可执行程序不同：

- windows系统：zw-gi.cmd （位于C:\Users\{用户名}\AppData\Roaming\npm）
- linux系统：zw-gi

## bin/zw-gi

意思很简单，就是执行我们写的getIgnore函数

```
#!/usr/bin/env node
var getIgnore = require('../getIgnore')
getIgnore();
```
`#!/usr/bin/env node`的意思
我们告诉 *nix 系统，JavaScript 文件的解释器应该是 /usr/bin/env node，它查找本地安装的 node。

## 发布

这样这个模块就写好了，可以发布到npm上了。

也可以不发布，在`package.json`文件的同级目录执行`npm install . -g`安装到本地，自娱自乐。



## 参考

[七天学会NodeJS](http://nqdeng.github.io/7-days-nodejs/)

[Nodejs 开发命令行工具](https://blog.csdn.net/haokur/article/details/81460973)

