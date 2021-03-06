## 回调地狱解决方案汇总

回调地狱可以通过对回调函数命名，进行简单的优化，但这并没有从本质上改变编程体验。能够本质改变异步编程体验的解决方案有：‘
- 利用 `events`模块：这是一种发布订阅方式，让回调函数与调用者解耦
- 使用第三方库：Async
- 利用ES6的Promise
- 利用ES7的async/await语法糖：该方案目前渐渐成为主流

异步递归是解决实现异步控制流的基础

## 一 使用事件模块实现发布订阅

原生的回调函数：
```js
obj.api1(function(data1){
    obj.api2(data1, function(data2){
        obj.api3(data2, function(data3){
            obj.api4(data3, function(data4){
                callback(data4);
            });
        });
    });
}){

}
```

普通解耦：
```js
var fn1 = function(data1) {
    obj.api2(data1, fn2);
}

var fn2 = function(data2) {
    obj.api3(data2, fn3)
}

var fn3 = function(data3){
    obj.api4(data3, fn4)
}

var fn4 = function(data4){
    callback(data4)
}

obj.api1(fn1)
```

使用事件解耦：
```js
var emitter = new event.Emitter();

emitter.on('step1', function(){
    obj.api1(function(data1){
        emitter.emit('step2', data1);
    });
});

emitter.on('step2', function(data1){
    obj.api2(data1, function(data2){
        emitter.emit('step3', data2);
    });
});

emitter.on('step3', function(data2){
    obj.api3(data2, function(data3){
        emitter.emit('step4', data3);
    });
});

emitter.on('step4', function(data3){
    obj.api1(data3, function(data4){
        callback(data4);
    });
});

emitter.emit("step1");
```

从上述看出，使用事件的方式，需要预先设定执行流程，这是由发布订阅模式的机制决定的。

## 二 第三方库Async库

第三方库Async是ES7 async/await出现之前（node7.6），最常使用的异步库。  

```js
var async = require('async');

async.waterfall([
    function(callback) {
        callback(null, 'one', 'two');
    },
    function(arg1, arg2, callback) {
        // arg1 now equals 'one' and arg2 now equals 'two'
        callback(null, 'three');
    },
    function(arg1, callback) {
        // arg1 now equals 'three'
        callback(null, 'done');
    }
], function (err, result) {
    // result now equals 'done'
});
```

## 三 使用 Promise

由于事件模式必须预先设定执行流程，如下所示：
```js
$.get('/api', {
    sucess: onSuccess,
    error: onError,
    complete: onComplete
})
```

上面的三个异步调用必须严谨的设置目标，但是在Promise方案中，可以先执行异步调用，延迟传递处理，即Promise/Deferred模式，jQuery1.5利用该模式重构了ajax，如下所示：
```js
$.get('/api')
    .sucess(onSuccess),
    .error(onError),
    .complete(onComplete);
```

即使不调用`success()`，`error()`等方法，ajax也会执行，这样写起来感觉上更加优雅。该异步模型被CommonJS规范接受后，逐渐抽象出了Promise/A，Promise/B，Promise/D等异步模型。至于这些模型都是什么概念，请查阅本笔记ES6相关章节。  

Promise现在是ES6官方认可的社区异步方案，已进入标准化：
```js
new Promise(
  function (resolve, reject) {
    // 一段耗时的异步操作
    resolve('成功') // 数据处理完成
    // reject('失败') // 数据处理出错
  }
).then(
  (res) => {console.log(res)},  // 成功
  (err) => {console.log(err)} // 失败
)
```

详细使用参见ES6章节

## 四 Generator

generator也是ES6退出的异步解决方案，但是是过渡性方案，现在已经不再使用，async/await异步控制方案是当前最主流的异步控制方案

## 五 async/await

```js
function fn(){
    return new Promise((resolve, reject)=>{
        let sino = parseInt(Math.random() * 6 +1)
        setTimeout(()=>{
            resolve(sino)
        },3000)
    })
}
async function test(){
    let n = await fn()
    console.log(n)
}
test()
```
详细使用参见ES6章节