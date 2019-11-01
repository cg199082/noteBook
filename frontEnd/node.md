# nodejs学习
参考文档：  
node.js之Express入门详解：https://segmentfault.com/a/1190000011654289  
Express/Node 入门：https://developer.mozilla.org/zh-CN/docs/Learn/Server-side/Express_Nodejs/Introduction


# 什么是nodejs
nodejs是一个开源的跨平台的JavaScript运行时环境，有了它，开发人员可以使用JavaScript来创建各种服务器端的应用程序。它运行于非浏览器环境（直接运行在计算机或者服务器系统上）,因此他去掉了一些浏览器端专用的JavaScript Api（如dom节点操作api）,同时加上了一些与操作系统交互相关的api（如用户处理http连接库和用于操作系统文件相关的库）
node的优点：
1. node为优化web的吞吐量和扩展而生，因此在处理web请求上性能更加的卓越
2. node自带的包管理工具npm提供了一套一流的包依赖解决方案和工具，将开发人员从繁杂的依赖关系管理中解脱出来，更关注于业务开发。
3. 由于nodejs脱离于浏览器端运行，有优良的可移植性，可以很方便的移植到各种操作系统平台上运行。
4. 有很活跃的第三方生态系统和开发社区

# 什么是express
express是目前最流行的node框架，是许多其他流行node框架的底层库。他提供了以下功能：  
1. 将不同的url路径匹配到不同的HTTP路由和后端的处理程序上
2. 在node上集成了“视图渲染引擎”，可以将数据插入到模板上来，组装完成后返回到请求端
3. 规范化管理web相关的设置项，涉及到的文件等。比如配置http监听端口，http请求头常量，定义渲染模板位置，定义静态文件位置。
4. 在处理请求的管道的任何位置，开发人员可以方便的插入额外的“中间件”，来加入直接的处理逻辑。

express本身是一个功能极简的框架，它主要是通过路由管理匹配和中间件功能来实现了web的编程功能。

> 中间件本身是一个函数，它可以访问web处理过程中的请求对象，响应对象和web应用中处于请求-响应过程中的其他中间件，一般被命名为next的变量。
中间件的功能：
1. 执行任何代码逻辑
2. 修改请求和响应对象
3. 终结请求-响应循环
4. 调用堆栈中的下一个中间件

按照应用类型来划分中间件大致可以分为以下几类：  
1. 应用级中间件
2. 路由级中间件
3. 错误处理中间件
4. 内置中间件、
5. 第三方中间件


## 应用级中间件
应用级中间件直接绑定到express对象上，使用express.use()和express.method()来绑定，常见的express自带的应用级中间件包括：
express.json()：处理json中间件
express.urlencoded()：处理urlencode中间件
```
var express = require('express');//express核心包
var path = require('path');//处理文件相关的核心包
var cookieParser = require('cookie-parser');//解析管理cookie相关的第三方中间件
var logger = require('morgan');//处理日志相关的第三方中间件

var app = express();
app.use(logger('dev'));
app.use(express.json());//处理json相关的中间件
app.use(express.urlencoded({ extended: false }));//处理url中参数encode相关的中间件
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));//管理express应用中静态文件相关的中间件
```

## 路由中间件
路由中间件并不直接绑定到express上，而是绑定到express的router对象上，路由级中间件使用router.use()或者router.verb()方法来进行绑定。
```
var express = require('express');
var router = express.Router();

// 一个中间件栈，显示任何指向 /user/:id 的 HTTP 请求的信息
router.use('/user/:id', function(req, res, next) {
  console.log('Request URL:', req.originalUrl);
  next();
}, function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});
```

## 错误处理中间件
错误处理中间件也是应用级中间件的一种，也是直接绑定到express对象上的，其绑定方式和应用级中间件的绑定方式完全一样，他和普通的应用级中间件的区别是：它的函数参数中多了一个error对象专门返回错误信息，用于后续的错误处理。下面是一个错误处理中间件的标准格式：  
```
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

## 内置中间件
目前express只有一个内置中间件express.static(root, [options])，他基于serve-static原理，主要用来负责管理express中的静态资源文件。


## 第三方中间件
express支持加载第三方中间件，通过加载第三方的中间件是express应用增加了更多的功能。
使用第三方中间件时，只需要通过npm安装后，在需要对应中间件的位置加载上该中间件就可以使用了，可以在应用级别加载，也可以在路由级别加载，其具体的加载方式和对应的应用级，路由级中间件的加载方式完全一致。
实际上，在创建express应用时，就已经引入了第三方中间件，通过查看package.json文件依赖

```
"dependencies": {
    "body-parser": "~1.18.2",
    "cookie-parser": "~1.4.3",
    "debug": "~2.6.9",
    "express": "~4.15.5",
    "jade": "~1.11.0",
    "morgan": "~1.9.0",
    "serve-favicon": "~2.4.5"
}
```

可以发现已经引入了request body解析中间件body-parser，cookie处理中间件cookie-parser，日志处理中间件morgan等

