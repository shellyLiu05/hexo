---
title: promise
date: 2018-07-26 09:16:36
tags:
---

javascript是异步单线程的，所有操作的返回结果都是在回调函数中进行处理。

所以对于依赖于上一步处理的结果进行下一步处理的情况，就会存在回调函数的多层嵌套，其可读性也会随着嵌套的增多而变差。如下所示

```javascript
functionA(para, function(res){
    functionB(para, function(res){
        functionC(para, function(res){
            functionD(para, function(res){
                ...
            });
        });
    });
});
```

而ES6提供了promise来供我们处理这种链式情况，使代码的可读性更好，并可以用异步的方式来处理同步问题。

## 简单的promise的使用

例1

```javascript
var promise1 = new Promise(function(resolve, reject) {
	setTimeout(function(){
		var num = Math.round(Math.random() * 10);
		if (num < 5) {
			resolve(num);
		} else {
			reject(num);
		}
	}, 1000);
});
console.log(promise1);
promise1.then(function(res){
    console.log('res:' + res);
}).catch(function(err){
    console.log('err:' + err);
});
```

当产生的随机数大于等于5时，返回结果：

```
Promise { <pending> }
err:9
```

当产生的随机数小于5时，返回结果：

```
Promise { <pending> }
res:3
```

例2

去掉上例中的`setTimeout`

```javascript
var promise1 = new Promise(function(resolve, reject) {
    var num = Math.round(Math.random() * 10);
    if (num < 5) {
      resolve(num);
    } else {
      reject(num);
    }
});
console.log(promise1);
promise1.then(function(res){
    console.log('res:' + res);
}).catch(function(err){
    console.log('err:' + err);
});
```

当产生的随机数大于等于5时，返回结果：

```javascript
Promise { <rejected> 7 }
err:7
```

当产生的随机数小于5时，返回结果：

```javascript
Promise { 4 }
res:4
```

例1和例2中如果随机数小于5，就会调用`resolve`函数，返回num的值，否则，就会调用`reject`函数返回num的值。

调用`Promise`后，通过`.then`来对`resolve`函数的结果进行处理，通过`.catch`来对`reject`函数返回的错误进行处理。

例1和例2`Promise`对象的状态不一样，后续在规则语法中说明。

## promise语法规则

### 语法

```javascript
new Promise( /* executor */ function(resolve, reject) { ... } );
```

`Promise`对象有两个参数`resolve`和`reject`。执行器（executor）通常开始一些异步的工作，当异步的工作处理完成后，处理成功则调用`resolve`函数来解决该`promise`，发生错误则调用`reject`拒绝该`promise`。

`Promise`有三种状态：

- pending: 初始状态
- fulfilled: 操作成功执行
- rejected: 操作执行失败

从上例的打印可以看到pending和rejected状态。执行成功时返回的fulfilled状态并未在打印中明显体现出来。

fulfilled状态是在https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise中说明的，很多博客上写的该状态叫resolved。称呼不一样，表征的状态是一致的。

### 方法

#### `Promise.resolve(value)`

返回一个resolved的Promise对象，可以直接使用`.then`方法进行后续处理。如`Promise.resolve(5).then(function(res){console.log(res)})`。

#### `Promise.reject(reason)`

返回一个rejected的Promise对象，可以直接使用`.catch`方法进行错误处理。

#### `Promise.all(iterable)`

该方法也返回一个Promise的对象。该方法可以用于多个promise的并行处理，对多个Promise进行处理，输入参数为多个Promise组成的数组，调用Promise.all后，输入的所有Promise都是异步执行，当输入的所有promise都成功执行（fulfilled状态）后，该Promise才成功执行（fulfilled状态）。一旦有一个Promise执行失败（rejected状态），该Promise直接转到失败状态（rejected状态）。

语言描述的有点绕，直接看例子。

```javascript
var promise1 = new Promise(function(resolve, reject) {
	setTimeout(function(){
		console.log('promise1');
		resolve(1);
	}, 1000);
});
var promise2 = new Promise(function(resolve, reject) {
		console.log('promise2');
		resolve(2);
});

var promise3 = '3'

var promise4 = Promise.reject('promise4 error');

Promise.all([promise1, promise2]).then(function(res){
	console.log('->test [promise1, promise2] result:');	
	console.log(res);
}).catch(function(err){
  console.log('->test [promise1, promise2] err:' + err);
});

Promise.all([promise3, promise4]).then(function(res){
	console.log('->test [promise3, promise4] result:');	
}).catch(function(err){
	console.log('->test [promise3, promise4] err:' + err);
});
```

返回结果：

```javascript
promise2
->test [promise3, promise4] err:promise4 error
promise1
->test [promise1, promise2] result:
[ 1, 2 ]
```

`Promise.all`接收一个promise数组，等待数组全部处理完成的结果，一旦某一个promise发生错误，即刻返回错误。

从执行结果可以看出，在Promise.all([promise1, promise2])中，promise1的执行在promise2之后，也充分说明promise数组中的所有promise是异步执行的。

Promise.all执行成功后返回的值是数组，数组值与输入参数数组中的promise一一对应，为每个promise成功执行后的返回值。

在Promise.all([promise3, promise4])中，promise4返回rejected，Promise.all也立即返回rejected。

#### `Promise.race(iterable)`

返回一个promise对象。该方法接受输入参数为promise数组，当promise数组中有一个最先resolved或是rejected，则该方法立即返回resolved或是rejected的Promise对象。

例子：

```javascript
var promise1 = new Promise(function(resolve, reject) {
	setTimeout(function(){
		reject('promise1');
	}, 1000);
});
var promise2 = Promise.resolve('promise2');

Promise.race([promise1, promise2]).then(function(res){
	console.log('->test race result:');	
	console.log(res);
}).catch(function(err){
  console.log('->test race err:' + err);
});
```

返回结果：

```javascript
->test race result:
promise2
```

如上所示，当promise2执行完成后，直接返回promise2的结果。

