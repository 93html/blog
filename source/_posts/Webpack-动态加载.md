---
title: Webpack 动态加载
date: 2016-12-19 15:28:17
categories: 技术
tags: Webpack
---
在 Webpack 的使用过程中，一开始人们都习惯了将一切东西 `import` 进来，这样做很符合逻辑，但是随着业务量不断增加，会发现 bundle 的体积不断增大，导致首次加载非常缓慢，比如我有一个路由映射的对象是这样写的
``` javascript
...
'/shop': require('components/Shop/Home'),
'/shop/goods': require('components/Shop/Goods'),
'/shop/goods-detail': require('components/Shop/Detail'),
...
```

最终这些模块都会被打包进 bundle，而我想要的效果是每次只加载当前路由对应的模块，这一点 Webpack 已经为我们提供了解决方案
## [Code Splitting](http://webpack.github.io/docs/code-splitting.html)
Code Splitting 的做法是在代码中定义分离点，在这个分离点内依赖的模块，在编译阶段会被单独打包，并在运行时动态加载进来

Code Splitting 支持 AMD 和 CommonJs 两种风格
#### CommonJs:
``` js
// 参数一 module-a, module-b 作为依赖会首先加载
// 参数二是 callback 函数，参数 require 可以加载模块
// 参数三是为这个分离点模块命名
// 最终 module-a module-b module-c 会被打包成一个文件
require.ensure(["module-a", "module-b"], function(require) {
    var a = require("module-a");
    var c = require("module-c");
    // ...
}, 'bundle');
```
注意：require.ensure 的模块只会被下载下来，不会被执行，只有在回调函数使用require(模块名)后，这个模块才会被执行。

#### AMD:
``` js
// 参数一同样会首先被加载，并作为参数传入 callback
require(["module-a", "module-b"], function(a, b) {
    var c = require("module-c");
    // ...
});
```
注意 callback 是必须的

当代码运行时 Webpack 会在文档中动态插入
``` js
<script type="text/javascript" charset="utf-8" async="" src="1.bundle.js"></script>
```

Webpack 只会当代码运行时才去加载，假如说我定义了多个分离点
``` js
require.ensure([], function(require) {
    var a = require("module-a");
});

if (false) {
    require.ensure([], function(require) {
        var a = require("module-b");
    });
}
```
module-b 是不会被加载，利用这个功能，我们将那个路由映射修改一下
```js
...
'/shop': callback => require.ensure([], function(require) {
    callback(require("components/Shop/Home"));
}),
'/shop/goods': callback => require.ensure([], function(require) {
    callback(require("components/Shop/Goods"));
}),
'/shop/goods-detail': callback => require.ensure([], function(require) {
    callback(require("components/Shop/Detail"));
}),
...
```
在路由解析的代码也需要修改一下
``` js
// 原来的同步获取模块写法
var module = router.match(location.href);
render(module);

// 现使用异步方法获得
router.match(location.href, function(module) {
    render(module);
});
```
由于 require.ensure 写法过于冗长，我曾想把这段封装起来，但是发现是不可行的，require.ensure 方法十分特殊，必须以字符串作为参数
``` js
// 以下方式无效！
function getModule(path, callback) {
    require.ensure([], function(require) {
        callback(require(path));
    });
}

...
'/shop': callback => getModule("components/Shop/Home", callback),
...
```

## 请求路经配置
根据 Code Splitting 的原理，模块请求的默认 hostname 是和当前页面相同的，比如访问 `http://www.x.com/` 那么模块的请求就是 `http://www.x.com/1.bundle.js`

如果要设置在别的域获取，可以在 `output.publicPath` 中设置
``` js
...
output: {
    publicPath: 'http://cdn.x.com/'
}
...
```
那么请求就变成了 `http://cdn.x.com/1.bundle.js`

这是一种在编译阶段配置的方法，如果路经需要到运行时才能确定，可以在入口文件中设置
``` js
__webpack_public_path__ = `http://${config.host}:/`
```
详细用法参考 [https://webpack.github.io/docs/configuration.html#output-path](https://webpack.github.io/docs/configuration.html#output-path)
