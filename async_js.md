

### 几种常见的异步的情景
- setTimeout，setInterval
- xhr(async)
- jsonp
- 事件：ondomready、onload、onclick、onmouseover
- animation

### 大原则
- 单线程执行（不要担心被打断）

example1：
```javascript
for(var i=0;i<=3;i++){
     setTimeout(function(){
          console.log(i);
     },0);
}
while(i<10000000){
    i++;
}
```

example2：

```javascript
var start = new Date;
setTimeout(
    function(){
        var end = new Date;
        console.log('Time elapsed:', end - start, 'ms');
    }
    , 500);
while (new Date - start < 1000) {};
```

- 不要假定几个异步函数的执行顺序
- 尽量不要写出“金字塔”型程序
```javascript
step1(function(result1) {
  step2(function(result2) {
    step3(function(result3) {
      // and so on...    });
  });
});
```
- 异步代码的异常处理
  - try的范围
  - 没有stacktrace
  - throw
  - onerror

## 异步代码的常见模式

### 事件机制
  - 同义词：emit=trigger, on=bind
  - node的EventEmitter
```javascript
var util = require("util");  
var events = require("events");//EventEmitter通过events模块来访问  
  
function MyStream() {//新建一个类  
    events.EventEmitter.call(this);  
}  
  
util.inherits(MyStream, events.EventEmitter);//使这个类继承EventEmitter  
  
MyStream.prototype.write = function(data) {//定义一个新方法  
    this.emit("data", data);//在此触发名为"data"事件  
}
  
var stream = new MyStream();  
  
stream.on("data", function(data) {//注册监听器，监听名为"data"事件  
    console.log('Received data: "' + data + '"');  
})  
stream.write("It works!"); // Received data: "It works!"  
```
  - 一个简单的EventEmitter实现
```javascript
function EventEmitter(){
  this.handlers=[];
}
EventEmitter.prototype.on = function(eventType, handler) {
  if (!(eventType in this.handlers)) {
    this.handlers[eventType] = [];
  }
  this.handlers[eventType].push(handler);
  return this;
};
EventEmitter.prototype.emit = function(eventType) {
  var handlerArgs = Array.prototype.slice.call(arguments, 1);
  for (var i = 0; i < this.handlers[eventType].length; i++) {
    this.handlers[eventType][i].apply(this, handlerArgs);
  }
  return this;
};
```
  - jquery的事件
    - triggerHandler和one
    - delegate
```javascript
$(window).on("swap_end",function(){
	if(bds.se.trust){
		bds.se.trust.init();
	}
    bds.se.heightControl.init();
	//dom ready后重新通过浏览器宽度计算结果容器尺寸，用于解决百度浏览器右侧滚动条占据浏览器宽度而引起的计算问题。
	bds.util.setContainerWidth();
});
$(window).trigger("swap_end");
```


### promise，deferred
  - done，fail，always
  - reject，resolve
  - when，几个promise的合并
  - [阮一峰写的deferred对象详解](http://www.ruanyifeng.com/blog/2011/08/a_detailed_explanation_of_jquery_deferred_object.html)

### jquery的动画queue

## 陷阱
- setTimeout0是有延迟的
- 可能异步，可能同步的函数（base64里使用deferred的例子）
```javascript
var _allDeferred=allDeferred=$.when.apply($,preXhrs).always(function(){
    if(_allDeferred!==allDeferred){
        return;
    }
    if(_opt.b64Exp==2){
        imgLoad("left");
    }
    imgLoad("right");
});
```

### 超越单线程（worker，了解）
