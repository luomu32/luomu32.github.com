# ECMAScript6

##新特性

### 默认值

```js
function count(height=10){
    console.log(height);
}
```

ES5：

```js
function count(height){
    var height=height||10
    console.log(height)
}
```

###Let和Const

Const关键字用于声明一个常量。

### 字符串模版

使用\`字符串`来引用字符串，字符串内可以注入变量，不需要字符串拼接

### 箭头函数



### 模块

```javascript
export var port=3000;
export function getName(){
    
}
```

```js
import {port,getName} from 'host'
console.log(port)
```

或者

```js
import * as host from 'host'
console.log(host.port)
```

### 类

###Promise

```javascript
let promise = new Promise((resolve,reject)=>{
    
})
promise.then(()=>{
    //resolve
},()=>{
    //reject
})
```

Promise用于一步操作。包含then方法，catch方法，all方法，race方法。

常见的场景是Ajax请求，当一个Ajax请求的参数依赖于上一个Ajax请求的返回结果时，那么代码会变的很不优雅。

###Async

## 转化

目前由于大多数浏览器对ES6不支持或支持不完全，为了兼容性，需要使用一些框架将ES6代码转化成对应的ES5。Babel就是一个比较常用的转码器。

`npm install --save-dev babel-cli`

在全局安装babel-cli，这意味着，如果项目要运行，全局环境必须有Babel，也就是说项目产生了对环境的依赖， 并且这样做也无法支持不同项目使用不同版本的Babel。所以官网**强烈建议**我们使用后一种方式，在项目中安装。 