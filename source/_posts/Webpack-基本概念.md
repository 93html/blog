---
title: Webpack 基本概念
date: 2016-12-27 12:40:10
categories: 技术
tags: Webpack
---
Webpack 的核心概念可以分为：Entry point，Chunk，Module，这三者之间的关系可以用一张官方图片来解释：

![generated update chunks](https://webpack.github.io/assets/HMR.svg)

## [Entry point](http://webpack.github.io/docs/configuration.html#entry)
Entry point 是比较好理解的，指的是程序的入口文件，也是 Webpack 解析依赖的入口文件，Entry point 可以设置单入口或者多入口，以一个最简单的入口文件为例

``` js
// index.js
var a = require('./a');

a();
```

## Chunk
Chunk 指的是经过编译后的代码包，Webpack 会将每个由用户定义的模块，转换成 Chunk 的形式，这些 Chunk 最终汇总到一个数组中，供运行时调用

``` js
// 原始的 a.js
module.export = function() {
    alert('Hello');
};

// 经过编译后的 chunk
function(module, exports, __webpack_require__) {
    'use strict';

    module.exports = function () {
        alert('Hello');
    };
}
```

这里 Chunk 转换的过程中还会调用 Loaders，Loaders 的作用是将非 JS 资源转换成字符串，并且提供运行时解析这些字符串的功能

最终 Webpack 为模块代码外面包了一层 function，并且注入了 `module`、`exports` 和 `__webpack_require__` 3个参数，这些参数的定义看下文代码示例中的 `require` 方法


## Module
Module 是拥有如下结构的一个 `Object`，它的作用是保存 Chunk 的信息

``` js
{
    exports: {},
    id: 3,
    loaded: false
}
```
一个 Module 对应一个 Chunk，`exports` 保存了 Chunk 中输出的内容，`id` 为 Chunk 的索引，`loaded` 记录 Chunk 是否加载

## 它们是如何工作的？
Webpack Runtime 的一个简单实现

``` js
const chunks = [
    function(module, exports, __webpack_require__) {
        'use strict';

        module.exports = function () {
            alert('Hello');
        };
    },
    ...
];

function require(id) {
    // 定义 module 对象
    var module = {
        exports: {},
        id: moduleId,
        loaded: false
    };

    // 执行 chunk
    chunks[id].call(module.exports, module, module.exports, require);

    module.loaded = true;

    // 返回 module 的输出部分
	return module.exports;
}

// entry point
var a = require(0); // 这里的 require('./a') 会被替换为 chunk 的索引

a();
//
```
