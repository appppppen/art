我们都知道js是单线程模型。也就是说一次只能处理一件事情，前面的事情没有完毕，后面的事情要等待前面的事情处理完毕后才能执行。随着多核CPU的出现，我们可以最大限度的利用cpu多核，来提高js的性能。

Worker接口可以创建后台任务。即可以给js运行新增线程。用于处理一些耗时、耗费性能的任务（异步的除外）。       

要解决的问题是：1.解决页面卡死问题。2.发挥多核CPU的优势，提高js性能。

## 一、Web Worker 具体用法
``` javascript
//main.js 主线程
//传递一个数字给worker线程。worker线程计算处理完毕后，返回一个信息给主线程。
if (window.Worker) {   //这一步比较重要，兼容性判断。
        //1.创建一个worker线程。
	const myWorker = new Worker("worker.js");
        //2.向worker线程发送数据，值可以是number,string,boolean ,file,blob 等
	  myWorker.postMessage(10000000000);
	
        //3.监听后台任务，
	myWorker.onmessage = function(e) {
	    result.textContent = e.data;
	    console.log('Message received from worker');
	}

       //4.当离开页面的时候，或者需要结束worker时(比如任务完成时)，
        //可以结束Worker线程，不必占用资源       
       // myworker.terminate();


       //5.当myWorker异常时的时候，会触发onerror事件    
        myWorker.onerror = function() {
             console.log('There is an error with your worker!');
        }
} else {  //这一步非常重要。具体代码根据需要更具自己的业务写。
	console.log('Your browser doesn\'t support web workers.')
}
复制代码//worker.js  worker 线程
// 程序处理完毕后返回一个结果给主线程。//0 可以加载其他js进来，比如ajax.
//importScripts('ajax.js','b.js')  

//1.监听主线程
onmessage = function(e) {
  console.log('Worker: Message received from main script');

    接收来自主线程发送过来的数据
    let num = e.data;
//使用for循环模拟一个耗时、耗性能的任务。（如果这个for循环放在主线程，那么页面很可能会卡死，
//影响用户体验）。for(let i = 0;i<=num;i++){
    if(i==num){
       //2.向主线程发送数据
        postMessage('任务完成啦！')
    }
}
//3.worker 线程也可以调用close 方法来结束worker线程。
// close()


 //4.同样的，在worker 线程中也可以监听错误信息。
onerror = function(err){
    console.log(err)
}
```
## 二、使用场景
接下来说说哪些任务比较适合使用Worker。这个是最重要的一点。1.如果是很消耗主线程性能的程序。可以考虑使用Web Worker。可以防止页面卡死情况。2.弹屏，webaudio（两个群友的看法）。www.bilibili.com/  就有使用Worker。但由于代码被压缩，混淆，未能看出其详细使用情况。欢迎评论，多多指教Worker的应用场景。
## 三、注意事项
- 1.非同源限制(本地代码如果以盘符的方式打开也不行，因为没有源啦)  
 
主线程代码与Worker线程代码必须同源才能一起正常工作。
- 2.DOM、脚本限制

“Worker 线程所在的上下文环境与主线程不一样，无法读取主线程所在网页的 DOM 元素，也无法使用document、window这些对象。但是，Worker 线程可以使用navigator，location,XMLHttpRequest。除此之外，Worker 线程不能执行alert()、confirm()等方法.
- 3.任务顺序

Worker线程任务需要等待主线程任务结束才能进行。
- 4.Worker结束  

可以主动关闭Worker线程。如果是多页应用的话，离开了Worker页面，Worker 也不会工作。
## 四、其他
1.关于Ajax轮询放在Worker线程里面的看法。 
       
ajax 本来是异步操作，有自己的线程，而且兼容性很好。如果使用Worker的话，需要考虑兼容性。所以ajax轮询放在Worker里面得不偿失。是不明智的。

2.关于Worker中在创建Worker的看法。
       
由于Worker中在创建Worker兼容性差，不推荐使用。
