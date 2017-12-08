---
layout: post
title: Pomelo研究笔记 - 启动与停止
tags: Pomelo
categories: blog
cover: "http://twitter.github.io/bootstrap/assets/img/examples/slide-02.jpg" 
headLine: "Pomelo"
---
Pomelo研究笔记：启动与停止
=====
代码Ref: 5f7bec1c9be5fa2f88a366b15085cf64d4e786d7

本分介绍了Pomelo的启动和停止的逻辑，以及lifecycle机制

### 初始化
Pomelo的初始化以require('pomelo').createApp()完成。   
createApp返回一个全局的对象，这个对象也可以通过require('pomelo').app访问，所以在实际使用中也都称之为app。   
createApp中完成一些初始化操作：
- 加载配置
    - app.master下存放的是master的配置，可以通过getMaster()访问
    - 加载servers配置，可以通过getServersFromConfig()访问
    - 加载lifecycle配置
    - 初始化logger配置
    - 处理程序参数，并将参数存储于curServer，可以通过getCurServer()获取。相关配置有：
        - serverType，默认为master，访问函数：getServerType
        - serverId，默认为master的id，访问函数：getServerId
        - mode，默认为cluster
        - masterha，默认为false
        - type，默认为all
        - startId/id，没有默认值，是指定server id的启动方式

另外在require('pomelo')时，也完成了一部分初始化操作
- 加载connectors
- 加载pushSchedulers
- 加载內置components
- 加载filters
- 加载rpc filters

### 启动流程
app.start(cb)启动服务器
- 启动服务器
    - 根据启动模式(type)，判断是哪种模式
      - all：启动servers.json里所有服务器
      - master：只启动master服务器
      - 其它：根据配置启动服务器
    - lib/master/starter.js 中实现了本地启动和远程ssh启动两种启动方式
    - 调用lifecycle的beforeStartup接口
- 加载内置的组件
    - pomelo.monitor, app.get('monitorConfig')
    - 对于master，启动以下组件
      - master, 使用配置：app.get('masterConfig')
    - 对于其它服务器，启动以下组件
      - proxy, 使用配置：app.get('proxyConfig')
      - pomelo.backendSession, app.get('backendSessionConfig')
      - pomelo.channel, app.get('channelConfig')
      - pomelo.server, app.get('serverConfig')
      - 对于提供了port（内部端口）的服务器，启动以下组件
        - remote, 使用配置：app.get('remoteConfig')
      - app.isFrontend()，对于前端服务器，启动以下组件
        - pomelo.connection, app.get('connectionConfig')
        - pomelo.connector, app.get('connectorConfig')
        - pomelo.session, app.get('sessionConfig')
        - pushScheduler, app.get('schedulerConfig') 优先于 app.get('pushSchedulerConfig')
    - 调用lifecycle的afterStartup接口
- 调用每个已经加载的组件的start方法，完成组件的启动

单独启动master的方式
- node app.js startId=master
- node app.js type=master

单独启动某个指定服务器的方式
- node app.js startId=xxxx
- node app.js id=xxxx

### 退出流程
app.stop(force)停止服务
- 启动一个定时器，到时间后会调用process.exit(0)退出进程   
- 调用lifecycle的beforeShutdown接口


### lifecycle
lifecycle是贯穿pomelo生命周期的四个回调函数，存储位置：app/servers/[serverType]/lifecycle.js   
需要导出四个函数：
- beforeStartup (app, startUp)
  - 加载配置之后，加载组件之前
  - startUp函数会开始组件的加载，完成服务器的启动，所以要记得调用这个函数
- afterStartup (app, done)
  - 加载组件之后调用
  - 记得调用done
- afterStartAll (app)
  - 全部服务器完成加载后的通知
- beforeShutdown (app, shutDown, cancelShutDownTimer)
  - cancelShutDownTimer是用来取消退出倒计时，如果停服时要做的扫尾工作时间比较久，记得调用这个方法
  - shutDown，开始退出。如果要退出的话，一定要记得调用shutDown




/////////////////////////////////////////
/**
 * Event definitions that would be emitted by app.event
 */
Pomelo.events = require('./util/events');


/**
 * connectors
 */
Pomelo.connectors = {};
Pomelo.connectors.__defineGetter__('sioconnector', load.bind(null, './connectors/sioconnector'));
Pomelo.connectors.__defineGetter__('hybridconnector', load.bind(null, './connectors/hybridconnector'));
Pomelo.connectors.__defineGetter__('udpconnector', load.bind(null, './connectors/udpconnector'));
Pomelo.connectors.__defineGetter__('mqttconnector', load.bind(null, './connectors/mqttconnector'));

/**
 * pushSchedulers
 */
Pomelo.pushSchedulers = {};
Pomelo.pushSchedulers.__defineGetter__('direct', load.bind(null, './pushSchedulers/direct'));
Pomelo.pushSchedulers.__defineGetter__('buffer', load.bind(null, './pushSchedulers/buffer'));


///////// components
lib/pomelo.js里，初始化了Pomelo.components
lib/application.js的init函数里，初始化乐app.components
这两个components是什么关系

/**
 * auto loaded components
 */
Pomelo.components = {};
/**
 * Auto-load bundled components with getters.
 */
fs.readdirSync(__dirname + '/components').forEach(function (filename) {
  if (!/\.js$/.test(filename)) {
    return;
  }
  var name = path.basename(filename, '.js');
  var _load = load.bind(null, './components/', name);
  
  Pomelo.components.__defineGetter__(name, _load);
  Pomelo.__defineGetter__(name, _load);
});


/////////

/**
 * auto loaded filters
 */
Pomelo.filters = {};
fs.readdirSync(__dirname + '/filters/handler').forEach(function (filename) {
  if (!/\.js$/.test(filename)) {
    return;
  }
  var name = path.basename(filename, '.js');
  var _load = load.bind(null, './filters/handler/', name);
  
  Pomelo.filters.__defineGetter__(name, _load);
  Pomelo.__defineGetter__(name, _load);
});

////////////
/**
 * auto loaded rpc filters
 */
Pomelo.rpcFilters = {};
fs.readdirSync(__dirname + '/filters/rpc').forEach(function (filename) {
  if (!/\.js$/.test(filename)) {
    return;
  }
  var name = path.basename(filename, '.js');
  var _load = load.bind(null, './filters/rpc/', name);
  
  Pomelo.rpcFilters.__defineGetter__(name, _load);
});

function load(path, name) {
  if (name) {
    return require(path + name);
  }
  return require(path);
}



/////////////////// cluster
lib/util/appUtil.js
loadServers:
if(server[Constants.RESERVED.CLUSTER_COUNT]) {
    utils.loadCluster(app, server, serverMap);
    continue;
}
CLUSTER_COUNT: 'clusterCount',
CLUSTER_PREFIX: 'cluster-server-',
CLUSTER_SIGNAL: '++',
CLUSTER: 'clusters',


/////////////////// logger
lib/util/appUtil.js
process.env.POMELO_LOGGER !== 'off'


////////////////// lifecycle
存于servers/[serverType]/lifecycle.js
里面每个成员都是函数
会加载并存储于app.lifecycleCbs