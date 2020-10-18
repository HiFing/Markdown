# Node.js

## 交互式解释器

### 多行表达式
```
$ node
> var x = 0
> do {
...x++
...console.log(x);
...} while(x < 5);
```
### 下划线变量

下划线保存上次运算的结果

```
$ node
> var x = 0
> var y = 20
> x + y
> var sum = _
> console.log(sum)
20
```

## 回调与事件监听

异步编程依托于回调实现，但不能说使用了回调后程序就实现异步化。

![img](https://www.runoob.com/wp-content/uploads/2015/09/event_loop.jpg)

```javascript
var events = require('events');
// 创建eventEmitter对象
var eventEmitter = new events.EventEmitter();

var connectHandler = function connected() {
   console.log('OK');
   // 触发data_received事件 
   eventEmitter.emit('data_received');
}

// 绑定connection事件处理程序
eventEmitter.on('connection', connectHandler);

// 使用匿名函数绑定data_received事件
eventEmitter.on('data_received', function(){
   console.log('数据接收成功。');
});

// 延迟1s触发connection事件 
setTimeout(function() {
    eventEmitter.emit('connection');
},1000)
// eventEmitter.emit('error'); 会触发报错
```

EventEmitter 对象如果在实例化时发生错误，会触发 error 事件。当添加新的监听器时，newListener 事件会触发，当监听器被移除时，removeListener 事件被触发。

## Buffer

```js
// const buf1 = Buffer.alloc(10); 创建一个长度为10，用0填充的buffer
const buf = Buffer.from([1,2,3,4,5]);
const json = JSON.stringify(buf);
console.log(json);

const copy = JSON.parse(json,(key,value) => {
    console.log(key,value,"hello");
    return value && value.type === 'Buffer' ?
        Buffer.from(value) : value;
});

console.log(copy);
```

https://www.runoob.com/nodejs/nodejs-buffer.html

## Stream

绑定了监听器

```js
// 读文件
var fs = require('fs');
var data = '';
var readStream = fs.createReadStream('input.txt');
readStream.setEncoding('UTF8');
readStream.on('data', function(chunk){
    data += chunk;
});
readStream.on('end',function(){
    console.log(data);
});

readStream.on('error',function(err){
    console.log(err.stack);
});
console.log("程序执行完毕");

// 写文件
var fs = require('fs');
var data = 'good, today is Sunday'

var writeStream = fs.createWriteStream('output.txt');

writeStream.write(data,'UTF8');
writeStream.end();

writeStream.on('finish', function(){
    console.log('写入完成');
});

writeStream.on('error', function(err){
    console.error(err);
});

// 管道
var fs = require('fs');
var readerStream = fs.createReadStream('input.txt');
var writeStram = fs.createWriteStream('output.txt');

readerStream.pipe(writeStream);
console.log('finished');

// 链式编程
// 压缩文件
var fs = require('fs');
var zlib = require('zlib');
fs.createReadStream('input.txt').pipe(zlib.createGzip()).pipe(fs.createWriteStream('input.txt.gz'));
// 解压文件
fs.createReadStream('input.txt.gz').pipe(zlib.createGunzip()).pipe(fs.createWriteStream('ouput.txt'));
```

## 模块

先在另一个文件中定义模块的内容

```js
function Hello(){
    var name;
    this.setName = function(thyName){
        name = thyName;
    };
    this.sayHello = function(){
        console.log('hello ',name);
    };
};
// module.exports提供整个模块 exports提供方法和成员变量
module.exports = Hello;
```

```js
function execute(someFunc,val){
    someFunc(val);
}
execute(function(word){
     console.log('hello',word);
}, 'Missy');
```

## 匿名函数

```js
function execute(someFunction, value) {
  someFunction(value);
}

execute(function(word){ console.log(word) }, "Hello");
```

