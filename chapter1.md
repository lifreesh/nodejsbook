

# Node.js in practice(intermediate)
##  1. Node Foundations

###1.1 Features of node.js

* 非阻塞I/O none-blocking IO
* 利于爬虫
* 与JSON 配合的很好

nodejs 主要特性

* node.js 的标准库由两部分构成
/Users/lishuang/Desktop/屏幕快照 2015-10-27 下午9.55.57.png

![image](/Users/lishuang/Desktop/test1.png)

binary libraries  与 core modules

binary libraries 为二进制文件  core modules 是用js 编写


###  1.2 建立一个新程序

1. 创建文件夹 nodeap
2. 创建一个app.js 文件 （如果不创建， npm init 后 main 入口会为index.js）
3. 在app.js 文件中写代码
4. npm init ，会自动生成package.json 文件(入口为 app.js)


## 2.Node’s global objects and methods

### 2.1 Modules

1. Installing and loading modules

   * 如果想查找modules: npm search express (正则表达式： npm search /^express$/ )  
   * 本地安装  npm install module-name
   * 全局安装（nodemon）npm install -g mudule-name
  
2. Creating and managing modules

	* 使用require ,返回一个对象（object）
	* 一旦一个module 被required了，它将被缓存，这意味着你多次require 返回的将是缓存下来的模块
    * delete require.cache[require.resolve('./myclass')]; 删除被缓存的module
    
3. Loading a group of related modules

    * 文件夹中创建一个index.js 的文件，在index.js文件中组织其他文件
    * 或者创建一个package.json 文件，指明main属性
    
    案例如下： 
    
    ![image](/Users/lishuang/Desktop/tree.tiff)
    
    index.js 文件中的代码如下
    
    ```
    module.exports ={
		one: require('./one'),
	 	two: require('./two')
     }
    ```
    
    在app.js 中
    
    ```
    var mymodule = require('./mymodule');
    mymodule.one();
    mymodule.two();
  
    ````
    
     或者在mymodule 文件夹中添加一个package.json 文件：(测试无效)
     
4. Working with paths
	* 使用__dirname 和__filename  来定位文件
	指的是当前文件所在的文件夹路径名
	
	可以使用
	
	```
	varview=__dirname+'/views/view.html'
	```
	 或者使用path模块
	 
	```
	 path =require('path')
	 path.join(__dirname, 'views', 'view.html');.
	```

### 2.2 Standard I/O and the console object
     
  1. Reading and writing to standard I/O
  
  ```
  process.stdin.resume();
  process.stdin.setEncoding('utf8');

  process.stdin.on('data',function(text){
	process.stdout.write(text.toUpperCase());
  });

```

视图如下： 
![image](/Users/lishuang/Desktop/nodeprocess.tiff
)
        

2. Benchmarking a program

使用console.time('label')
  console.timeEnd('label')
  
  
### 2.3 Operating system and command-line integration

1. Getting platform information

   * process  一般是用来处理与操作系统相关的信息
   
   * 使用 process.arch 与 process.platform 属性来处理相关信息
   process.arch  查看当前处理器平台
   
   'arm', 'ia32', or 'x64'.
   
   *  process.memoryUsage()  内存使用情况
   
2. Passing command-line arguments

process.argv

### 2.4 Delaying execution with timers

1.setTimeout 与 Function.prototype.bind()
 
 
 


## 3. Buffers

Buffer 是全局类型，可以用来处理二进制数据

```
var fs = require('fs');
var greet = fs.readFileSync(__dirname+'/greet.txt','utf8');
console.log(greet);

var greet2 = fs.readFile(__dirname+'/greet.txt',function(err,data){
	console.log(data.toString());
});
console.log('Done');

fs.readFile(__dirname+'/greet.txt',function(err,buf){
	if(err) console.log("read file error");
	console.log(buf.toString());
	console.log(Buffer.isBuffer(buf));  
	console.log(Buffer.byteLength(buf.toString(),'utf8'));
})

```
  
   
 Buffer.isBuffer(obj);  //判断对象是否为buffer <br>
 Buffer.byteLength(str,'utf8') ,区别于str.length(一个字符可能占用多个字节)
 
 给出一个写图片的例子
 
 ```
 var mine = 'test1.png';
 var encoding = 'base64';

 var data = fs.readFileSync(__dirname+"/test1.png").toString(encoding); //字符串

 var buf = new Buffer(data,encoding);//创建buffer 

  fs.writeFileSync(__dirname+'/copy.png',buf);
  

 ```
 
 
## 4.Events

关于EventEmitter模块

```
 var util = require('util');
var EventEmitter = require('events');

function AppleMusic(){
	EventEmitter.call(this);  // 继承了EventEmitter的ownProperty
	this.name = "You are my sunshine";
	this.playing =false;
}
util.inherits(AppleMusic,EventEmitter); //  继承了 EventEmitter 的原型

var music = new AppleMusic();

music.on('play',function(){
	this.playing = true;
})

music.on('play',function(track){
	this.playing = true;
	console.log("The track is: "+ track);
})
music.on('stop',function(){
	this.playing = false;
	console.log("Stoped the music")
})

music.emit('play',"你发如雪，就没了离别，我爱你如三千东流水") ;
setTimeout(function(){
	music.emit('stop');
},5000);
 
```

## 5.Streams
### 5.1 Built-in streams
 
 在上文中我们知道
 fs.readFile() 于fs.readFileSync() 都是将数据全部读到内存中去，也就是Buffer中，
 
 如果有一种方法，能够读取一部分数据（chunk）,然后处理它（process it）,然后继续读进来更多的数据
 
  以下链接提供了一个如何在node.js 中使用streams 的指南
 [https://github.com/substack/stream-handbook/](https://github.com/substack/stream-handbook/)

 图片如下：
 
 ![image](/Users/lishuang/Desktop/pipe2.png)
 
 使用管道
 
 要注意 request 与 response 都是stream,所以可以使用pipe 将他们连接起来
 
 使用pipe 以后，不用讲chunk 读到内存，直接发给res即可

``` 
   var  http = require('http');
   var fs = require('fs');
   var zlib = require('zlib');
   var server = http.createServer(function(req,res){
	res.writeHead(200,{"content-encoding": "gzip"})
	fs.createReadStream(__dirname+'/index.html').pipe(zlib.createGzip()).pipe(res);
   }).listen(1337);
   
```

处理error

```
var  http = require('http');
var fs = require('fs');
 
var server = http.createServer(function(req,res){
	res.writeHead(200,{"content-encoding": "gzip"})
	var stream = fs.createReadStream(__dirname+'/index.html') ;
	stream.on('error',function(err){
		console.log(err.trace());
		console.log('Error',err);
		console.log('Error stack: ',err.stack);
	})	
}

````

### 5.2 Third-party modules and streams


## 6.File system

## 7.Networking

## 8.Child processes
   


  
  




 







     


 





