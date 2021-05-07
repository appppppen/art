## Kue简介
Kue是一个由redis支撑的优先作业基地，为node.js创建

预提示这是最新的Kue文档，为兼容性请确保阅读更改列表。

### 安装
`$ npm install kue`

### 功能
- 延迟作业
- 并行工作负载分配
- 作业事件和进度发布订阅
- 丰富的集成用户界面
- 无限滚动
- UI的进度指示
- 作业的具体记录
- Redis驱动
- 具有回退任选重试
- 全文搜索功能
- RESTful JSON API
- 优雅的工人关机

## 创建作业

初步使用kue.createQueue()创造作业队列：
```javascript
var kue = require('kue') , jobs = kue.createQueue();
```
jobs.create()用作业类型（“电子邮件”）进行调用，然后任意作业数据将返回Job，然后可以对其进行save()编辑，并将其添加到redis中，默认优先级为“正常”。该save()方法可以选择接受回调，error如果出现问题，则以响应。该title键是特殊情况的键，将显示在UI内的作业列表中，使查找特定作业更加容易。
```
var job = jobs.create('email', {
    title: 'welcome email for tj'
  , to: '[email protected]'
  , template: 'welcome-email'
}).save( function(err){
   if( !err ) console.log( job.id );
});
```
### 作业优先级
要指定作业的优先级，只需priority()使用映射到数字的数字或优先级名称来调用该方法。
```javascript
jobs.create('email', {
    title: 'welcome email for tj'
  , to: '[email protected]'
  , template: 'welcome-email'
}).priority('high').save();
```
默认优先级映射如下：
```javascript
{
    low: 10
  , normal: 0
  , medium: -5
  , high: -10
  , critical: -15
};
```
### 失败尝试
默认情况下，作业仅进行一次尝试，也就是说，当它们失败时，它们会被标记为失败，并一直保持这种状态，直到您进行干预为止。但是，Kue允许您指定此设置，这对于诸如传输电子邮件之类的工作很重要，而失败后通常可以重试而不会出现问题。为此，请.attempts()使用数字调用该方法。
```javascript
 jobs.create('email', {
     title: 'welcome email for tj'
   , to: '[email protected]'
   , template: 'welcome-email'
 }).priority('high').attempts(5).save();
```
### 故障延迟
作业重试尝试一旦失败就立即进行，没有延迟，即使您的作业通过设置了延迟Job#delay。如果要延迟失败后作业的重新尝试（称为退避），则可以通过Job#backoff不同的方式使用方法：
```javascript
    // Honor job's original delay (if set) at each attempt, defaults to fixed backoff
    job.attempts(3).backoff( true )

    // Override delay value, fixed backoff
    job.attempts(3).backoff( {delay: 60*1000, type:'fixed'} )

    // Enable exponential backoff using original delay (if set)
    job.attempts(3).backoff( {type:'exponential'} )

    // Use a function to get a customized next attempt delay value
    job.attempts(3).backoff( function( attempts, delay ){
      return my_customized_calculated_delay;
    })
```
在最后一种情况下，将在每次重新尝试时调用提供的函数以获取当前尝试延迟值。

### 作业日志
特定于作业的日志使您可以在作业生命周期的任何时间向UI公开信息。为此，只需调用即可job.log()，该方法接受消息字符串以及类似sprintf的支持的变量参数：
```javascript
job.log('$%d sent to %s', amount, user.name);
```
### 作业进度
作业进度对于长时间运行的作业（例如视频转换）非常有用。要更新作业的进度，只需调用job.progress(completed, total)：
```javascript
job.progress(frames, totalFrames);
```
### 作业事件
Job通过Redis pubsub在实例上触发特定于作业的事件。当前支持以下事件：

- `failed` the job has failed and has no remaining attempts
- 'failed attempt' the job has failed, but has remaining attempts yet
- `complete` the job has completed
- `promotion` the job (when delayed) is now queued
- `progress` the job's progress ranging from 0-100

例如，这可能类似于以下内容：
```javascript
var job = jobs.create('video conversion', {
    title: 'converting loki\'s to avi'
  , user: 1
  , frames: 200
});

job.on('complete', function(result){
  console.log("Job completed with data ", result);
}).on('failed', function(){
  console.log("Job failed");
}).on('progress', function(progress){
  process.stdout.write('\r  job #' + job.id + ' ' + progress + '% complete');
});
```
请注意，在工作进程重新启动时，不能保证会收到作业级别的事件，因为该进程将丢失对特定Job对象的引用。如果您想要更可靠的事件处理程序，请查找[Queue Events](https://wohugb.gitbooks.io/kue/content/create_jobs/README.html#queue-events)。

### 集体事件
队列级事件提供对前面提到的作业级事件的访问，但是范围仅限于Queue实例以在“全局”级应用逻辑。一个示例是删除已完成的作业：
```javascript
jobs.on('job complete', function(id,result){
  kue.Job.get(id, function(err, job){
    if (err) return;
    job.remove(function(err){
      if (err) throw err;
      console.log('removed completed job #%d', job.id);
    });
  });
});
```
可用的事件与“作业事件”中提到的事件相同，但是以“ job”为前缀。

### 延迟作业
通过调用'.delay（ms）'方法，连续作业可以预设为任意远时间，传递相对于now微秒数字。这会自动将标记Job为“已延迟”。
```javascript
var email = jobs.create('email', {
    title: 'Account renewal required'
  , to: '[email protected]'
  , template: 'renewal-email'
}).delay(milliseconds)
  .priority('high')
  .save();
```
当使用连续作业时，我们必须使用计时检测延期作业，如果已经超过预定时间推进他们。Queue#promote(ms,limit)定义了这个setInterval，默认每5妙检测前200个作业。
```javascript
jobs.promote();
```
## 处理作业

使用Kue，处理作业很简单。首先创建一个Queue实例，就像创建作业一样，为我们提供对redis的访问权限，然后jobs.process()使用关联的类型进行调用。请注意，与名称所createQueue暗示的不同，它当前返回一个单例Queue实例。因此，您只能Queue在node.js进程中配置和使用单个对象。

在下面的示例中，我们将回调传递done给email，当发生错误时，我们调用done(err)以告诉Kue发生了某些事情，否则，我们done()仅在作业完成时调用。如果此功能响应错误，它将显示在UI中，并且作业将被标记为失败。
```javascript
var kue = require('kue')
 , jobs = kue.createQueue();

jobs.process('email', function(job, done){
  email(job.data.to, done);
});
```
工人可以将作业结果作为第二个参数传递来完成，done(null,result)以将其存储在Job.result密钥中。result还通过complete事件处理程序传递，以便作业生产者可以根据需要接收它。

### 处理并发
默认情况下，对的调用jobs.process()一次只会接受一个作业进行处理。对于诸如发送电子邮件之类的小任务，这并不理想，因此我们可以通过传递数字来指定此类型的最大活动作业：
```javascript
jobs.process('email', 20, function(job, done){
  // ...
});
```
### 暂停处理
工人可以暂时停下来并恢复他们的活动。就是说，在调用之后，pause直到resume被调用，他们将不会在其进程回调中收到任何作业。pause功能正常停机这个工人，并且使用同样的内部的功能shutdown在方法正常关机。
```javascript
jobs.process('email', function(job, done, ctx){
  ctx.pause( function(err){
    console.log("Worker is paused... ");
    setTimeout( function(){ ctx.resume(); }, 10000 );
  }, 5000);
});
```
### 更新进度
对于“真实的”示例，假设我们需要从大量带有node-canvas的幻灯片中编译PDF 。我们的工作可能包含以下数据，请注意，通常您不应该自己在工作中存储大数据，最好存储ID之类的引用，并在处理时将其拉入。
```javascript
jobs.create('slideshow pdf', {
    title: user.name + "'s slideshow"
  , slides: [...] // keys to data stored in redis, mongodb, or some other store
});
```
通过该job.data属性，我们可以在处理时在单独的进程中访问相同的任意数据。在该示例中，我们一张一张地渲染幻灯片，以更新作业的日志和流程。
```javascript
jobs.process('slideshow pdf', 5, function(job, done){
  var slides = job.data.slides
    , len = slides.length;

  function next(i) {
    var slide = slides[i]; // pretend we did a query on this slide id ;)
    job.log('rendering %dx%d slide', slide.width, slide.height);
    renderSlide(slide, function(err){
      if (err) return done(err);
      job.progress(i, len);
      if (i == len) done()
      else next(i + 1);
    });
  }

  next(0);
});
```
### 正常关机
从Kue 0.7.0开始，Queue#shutdown(fn, timeout)添加了a ，表示所有工作人员在当前活动完成后停止处理。工人们将等待timeout毫秒等待其活动作业被完成，或者将其标记为failed具有关闭错误原因的活动作业。当所有工人告诉K时，他们被叫停了fn。
```javascript
var queue = require('kue').createQueue();

process.once( 'SIGTERM', function ( sig ) {
  queue.shutdown(function(err) {
    console.log( 'Kue is shut down.', err||'' );
    process.exit( 0 );
  }, 5000 );
});
```

## Redis连接设置
默认，苦厄使用客户端默认设置连接Redis的（默认端口6379，默认主机127.0.0.1，默认前缀q）。Queue#createQueue(options)在options.redis键上接受redis的连接选项。
```javascript
var kue = require('kue');
var q = kue.createQueue({
  prefix: 'q',
  redis: {
    port: 1234,
    host: '10.0.50.20',
    auth: 'password',
    db: 3, // if provided select a non-default redis db
    options: {
      // see https://github.com/mranney/node_redis#rediscreateclientport-host-options
    }
  }
});
```
prefix控制在Redis里使用的键名。默认，只是简单的q。除非您需要将一个Redis实例用于多个应用程序，否则通常不应更改前缀。对于在您的主应用程序中提供隔离的测试平台，它也很有用。

### 使用Unix域专有连接
由于node_redis支持Unix域套接字，因此您也可以告诉Kue这样做。有关您的Redis服务器配置，请参见unix-domain-socket。
```javascript
var kue = require('kue');
var q = kue.createQueue({
  prefix: 'q',
  redis: {
    socket: '/data/sockets/redis.sock',
    auth: 'password',
    options: {
      // see https://github.com/mranney/node_redis#rediscreateclientport-host-options
    }
  }
});
```
### 更换Redis的客户端模块
任何符合（或在适应时）符合[node_redis](https://github.com/mranney/node_redis) API的node.js redis客户端库 都可以注入到Kue中。您只应提供一个createClientFactory功能作为redis连接工厂，而不要提供node_redis连接选项。

以下是使redis-sentinel能够连接到Redis Sentinel进行自动主/从故障转移的示例代码。
```javascript
var kue = require('kue');
var Sentinel = require('redis-sentinel');
var endpoints = [
  {host: '192.168.1.10', port: 6379},
  {host: '192.168.1.11', port: 6379}
];
var opts = options || {}; // Standard node_redis client options
var masterName = 'mymaster';
var sentinel = Sentinel.Sentinel(endpoints);

var q = kue.createQueue({
   redis: {
      createClientFactory: function(){
         return sentinel.createClient(masterName, opts);
      }
   }
});
```
**请注意** ，<0.8.x应重构所有客户端代码，以将redis选项传递给，Queue#createQueue而不是使用猴子补丁样式覆盖，redis#createClient否则它们会与Kue分离0.8.x。

## 用户界面

UI是一个小[Express](http://github.com/visionmedia/express)应用，要启动它只需运行一下内容，根据需要更改端口等。
```javascript
var kue = require('kue');
kue.createQueue(...);
kue.app.listen(3000);
```
标题默认为“ Kue”，要更改它请调用：
```javascript
kue.app.set('title', 'My Application');
```
**注意** 如果您使用非默认Kue选项，请访问kue.app前必须调用kue.createQueue(...)。

## JSON API

UI同时暴露了一组JSON接口，为UI所用。

### GET / job / search？q =
查询作业，例如“ GET / job / search？q = avi video”：

["5", "7", "10"]
默认kue为了搜索索引所有作业数据对象，但是可以通过调用Job#searchKeys来自定义，来告诉kue那些作业数据的键创建索引如：
```javascript
var kue = require('kue');
jobs = kue.createQueue();
jobs.create('email', {
    title: 'welcome email for tj'
  , to: '[email protected]'
  , template: 'welcome-email'
}).searchKeys( ['to', 'title'] ).save();
```
你也可设置坐标局部搜索索引来优化redis内存：
```javascript
var kue = require('kue');
q = kue.createQueue({
    disableSearch: true
});
```
### GET /统计
带状态计数当前响应，以及工人活动时间以几为单位：
```javascript
{"inactiveCount":4,"completeCount":69,"activeCount":2,"failedCount":0,"workTime":20892}
```
### GET / job /：id
通过:id：获取作业
```javascript
{"id":"3","type":"email","data":{"title":"welcome email for tj","to":"[email protected]","template":"welcome-email"},"priority":-10,"progress":"100","state":"complete","attempts":null,"created_at":"1309973155248","updated_at":"1309973155248","duration":"15002"}
```
### GET / job /：id / log
获取:id的日志：
```javascript
['foo', 'bar', 'baz']
```
### GET /jobs/:from..:to/:order？
获取指定:from到:to区间的作业，例如“ /jobs/0..2”， :order可能是“ asc”或“ desc”：
```javascript
[{"id":"12","type":"email","data":{"title":"welcome email for tj","to":"[email protected]","template":"welcome-email"},"priority":-10,"progress":0,"state":"active","attempts":null,"created_at":"1309973299293","updated_at":"1309973299293"},{"id":"130","type":"email","data":{"title":"welcome email for tj","to":"[email protected]","template":"welcome-email"},"priority":-10,"progress":0,"state":"active","attempts":null,"created_at":"1309975157291","updated_at":"1309975157291"}]
```
### GET / jobs /：状态/：从..：到/：命令？
同上，受限于以下:state：

- active
- inactive
- failed
- complete
### GET /jobs/:type/:state/:from..:to/:order？
同上，然而，有利于:type何:state。

### 删除/ job /：id
删除作业:id：

$ curl -X DELETE http://local:3000/job/2
{"message":"job 2 removed"}
### 职位/职位
创建作业：
```javascript
$ curl -H "Content-Type: application/json" -X POST -d \
    '{
       "type": "email",
       "data": {
         "title": "welcome email for tj",
         "to": "[email protected]",
         "template": "welcome-email"
       },
       "options" : {
         "attempts": 5,
         "priority": "high"
       }
     }' http://localhost:3000/job
{"message":"job 3 created"}
```

## 大规模并行处理

下面的示例演示了如何使用增加的CPU分配作业进程负载。请参见替换模块文档，使用它更详细的例子。

群集时.isMaster，文件是在主进程的上下文中执行的，在这种情况下，您可以执行只需要一次的任务，例如启动与Kue捆绑在一起的Web应用程序。else块中的逻辑是按每个worker执行的。
```javascript
var kue = require('kue')
  , cluster = require('cluster')
  , jobs = kue.createQueue();

var clusterWorkerSize = require('os').cpus().length;

if (cluster.isMaster) {
  kue.app.listen(3000);
  for (var i = 0; i < clusterWorkerSize; i++) {
    cluster.fork();
  }
} else {
  jobs.process('email', 10, function(job, done){
    var pending = 5
      , total = pending;

    var interval = setInterval(function(){
      job.log('sending!');
      job.progress(total - pending, total);
      --pending || done();
      pending || clearInterval(interval);
    }, 1000);
  });
}
```
这将为email您的每个计算机CPU核心创建一个作业处理器（工作人员），每个处理器您可以处理10个并发电子邮件作业，从而导致10 * N在N核心计算机中处理的并发电子邮件作业总数。

现在，当您在浏览器中访问Kue的UI时，您将看到作业的处理N速度大约提高了几倍！（如果您有N核心）。

## 保护 Kue

通过使用应用程序安装，您可以自定义Web应用程序，启用TLS或添加其他中间件（如Connect的）basicAuth()。
```javascript
var app = express.createServer({ ... tls options ... });
app.use(express.basicAuth('foo', 'bar'));
app.use(kue.app);
app.listen(3000);
```
截屏
- Kue[介绍](http://www.screenr.com/oyNs)
- Kue API [演练](http://vimeo.com/26963384)
- Kue [教程2]https://www.ctolib.com/kue.html