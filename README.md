Witch Node
=========

Witch Node是一个以Express框架开发的用于快速开发建站的程序，其目前主要用途是快速开发网站，他是在Express的基础上搭建了业务逻辑工具层，并对Express的使用进行了优化。

本系统旨在构建高性能web site,同时会对从开发，部署到维护的全过程进行实践性的介绍，同时期待大牛提出对这些方法的改进方案

注意：他不是一个cms系统，虽然开发出一个cms对认识Node和Express是非常有用，但我不会去构建，cms过度的封装只会影响性能，大型的web系统更需要Coder而不是cms。但构建cms也是容易的。

本系统只提供最基础的操作供业务逻辑来使用，比如以下方法：

> Session的构建和第三方存储(eg:Redis)
> MongoDB的配置和初始化
> Redis的连接池和get，set,remove等的封装
> 路由规则的重新分配，减少Express默认路由存在页面显示操作代码导致的页面代码过多的问题
> 引入xss过滤方法
> 编写的基础工具提供：字符串截取，时间换算，数据类型校验等方法

版本
=======
0.2.6


项目相关
==========

## 目录结构


   -----config //应用程序和中间件的配置

   -----controller //控制器

   -----logs  //日志存放目录

   -----lib //开发者自定义库文件

   -----test //单元测试等测试文件

   -----models  //模型

   -----public //静态文件目录

   ----routes  //路由

   ----views //模板

   ---app.js //启动文件

   ---package.json
##安装与使用
   1，下载到本地

   2，安装NPM
````js
   cd 文件存放目录
   npm install
````
   另外为了减少不必要的麻烦，如github响应速度慢，被墙等问题，最好设置npm安装是从中国镜像安装

   设置方法如下：
```````js
   npm config set registry http://registry.cnpmjs.org
   npm info underscore （如果上面配置正确这个命令会有字符串response）
````````
  3,安装Redis和mongodb，并修改config.js中的相关配置

  4，运行。注：本系统并没有写cluster的相关操作，生产环境请使用PM2，默认端口号3000，可在配置文件中修改
``````````js
  node app.js
``````````
********************
设置开发环境命令：

````js
linux：NODE_ENV=production

windows:set NODE_ENV=production
````

************************
本系统目前在windows和CentOS上做过测试，暂未发现问题。

生产环境请用PM2来做多进程和故障重启

本地开发可用supervisor来监控文件变化，同时Linux可以使用PM2

由于我的部署方案会完全剥离静态文件，会给静态文件添加域名，所以本系统不做gzip。js和css的合并，压缩，添加版本戳同属部署流程本系统也不开发相关功能。同时可以减少cpu负载

同时可以省掉部署环节和提高效率，默认提供7天的静态资源缓存。


##项目配置文件

````````js

    listenPort:3000,//监听端口

    uploadFolder:'/tmp/upload', //文件上传的临时目录

    postLimit:1024*1024*100,//限制上传的postbody大小，单位byte

    webTitle:'左盐的node',//网站标题

    staticMaxAge:604800000, //静态文件的缓存周期，建议设置为7天，单位毫秒

    md5Salt:'XDq-MW.Q',//供后端加密使用的盐

    keySalt:'H0UK*Lwd',//供前端加密使用的盐

    loginTimes:3,//登录次数，超出则锁定

    lockUserTime:1800,//锁定时间，单位秒

    webDomain:'192.168.1.202:3000',//网站主域名，用于判断Referer

    //session配置

    sessionExpire:43200000, //session过期时间，12小时，ms

    clearSessionSetInteval:120000, //自动清理垃圾session时间，2小时

	sessiconSecret: 'H0UK*Lwd', //session加密密匙

    //mongodb 配置
       MongodbConnectString:'mongodb://192.168.1.207:10000,192.168.1.207:20000,192.168.1.207:30000/zuoyan?safe=true&replicaSet=Friend&slaveOk=true&w=2&wtimeoutMS=2000&maxPoolSize=15', //MongoDB连接字符串

    //redis配置

    redisPort:'6379',

    redisIp:'192.168.1.207',

    redisMaxPoll:500,//redis最大连接池

    redisTimeOut:600000,//连接过期时间，过期连接将被删除//单位ms//现在定义为10分钟

    redisDataBase:0//默认使用的redis数据库下标，默认从0开始

````````

##项目模块介绍
####路由规则
Express默认的路由规则是形如这样的：

````````js
module.exports = function(app) {
  app.get('/', function (req, res) {
	//TODO:
  });
  app.get('/reg', function (req, res) {
    //TODO
  });
  app.post('/reg', function (req, res) {
   //TODO
  });
}
``````````

这样的结构虽然每个单元都很好理解，但把TODO补完，页面将变的复杂无比，满眼都是一坨一坨的代码，就算是拆分了文件，这个路由js仍然存在了很多的页面逻辑。为了避免这个非常不优雅的问题，我改写了下路由的运行方法(没有更改Express，确保无障碍升级)。

````````js
module.exports=function(app){
    app.all('*',function(req,res){
        var upath=req.path,
            urlpath=upath.split('/');
        urlpath.shift();
        if(urlpath[urlpath.length-1]==''){
            urlpath.pop();
        }
       if(upath=='/'){
               urlpath=new Array('index','index');
           }
       if(urlpath.length==1){
               urlpath.push('index');
         }
       require('../controller/'+urlpath[0])[urlpath[1]](req, res);
    });
}
```````````
app.all方法是Express原生语法，作用是接管所有页面请求（当然都是非静态的），然后把请求的目录组装成数组，然后通过

``````js
require('../controller/'+urlpath[0])[urlpath[1]](req, res);
`````
去执行代码
比如：
首页，'/' -----> ['index','index']  --->执行/controller/index.js 里的index方法
登录，'/login'------>['reg','index']  ---->执行/controller/login.js 里的index方法
登录按钮post地址,'/login/login' ----['login','login'] --->执行/controller/login.js 里的login方法
***********
这样就把所有本来该写在路由js的页面逻辑全部分担到了controller里的js文件里，各个页面的逻辑都不掺和，一下子感觉高雅了很多，而且两级目录(一文件名。一操作方法)在实际项目中已经基本够用了。

页面访问权限判断也将会在此完成。

这样就把所有本来该写在路由js的页面逻辑全部分担到了controller里的js文件里，各个页面的逻辑都不掺和，一下子感觉高雅了很多，而且两级目录(一文件名。一操作方法)在实际项目中已经基本够用了

EJS语法
==========

EJS只有三种标签：
> <% code %>：JavaScript 代码。

> <%= code %>：显示替换过 HTML 特殊字符的内容。

> <%- code %>：显示原始 HTML 内容。

><%- include  src %> :加载模版

注意： <%= code %> 和 <%- code %> 的区别，当变量 code 为普通字符串时，两者没有区别。当 code 比如为 \<h1\>hello\</h1\> 这种字符串时，<%= code %\> 会原样输出 \<h1\>hello\</h1\>，而 <%- code %\> 则会显示 H1 大的 hello 字符串。 

性能测试
====
> 测试方法：AB

> 测试环境：CentOS 6.5

> 机器配置： 内存：1.8GB，处理器：Intel(R) Celeron(R) E3300 2.5GHz,双核  -----配置真够差劲的，无奈

************************

测试1：
> 输出 Hello！！

> 并发数 100

> 测试时间：15s

结果：RPS：1456.98， TPR：0.686，FAIL:0

测试2：

> 输出 首页，包含redis操作

> 并发数 100

> 测试时间：15s

结果：RPS：1246.06， TPR：0.803，FAIL:0

*******************************************
测试用例也很简单，ab运行也是在本机，性能上都会有影响。只是心里大致有谱

####未完待续




