---
layout: w
title: promise实现
date: 2021-04-15 17:03:59
categories: javascript
tags: es6
---

## promise实现

### promise标准

[promiseA+规范](https://promisesaplus.com/)

Promise表示一个异步操作的最终结果。与Promise最主要的交互方法是通过将函数传入它的then方法从而获取得Promise最终的值或Promise最终拒绝（reject）的原因。

#### 1. 术语

    promise是一个包含了兼容promise规范then方法的对象或函数，
    thenable 是一个包含了then方法的对象或函数。
    value 是任何Javascript值。 (包括 undefined, thenable, promise等).
    exception 是由throw表达式抛出来的值。
    reason 是一个用于描述Promise被拒绝原因的值。

#### 2. 要求

1. Promise状态:
   * 一个Promise必须处在其中之一的状态：pending, fulfilled 或 rejected.
   * 如果是pending状态,则promise：
        可以转换到fulfilled或rejected状态。
   * 如果是fulfilled状态,则promise：
        * 不能转换成任何其它状态。
        * 必须有一个值，且这个值不能被改变。
   * 如果是rejected状态,则promise：
        * 不能转换成任何其它状态。
        * 必须有一个原因，且这个值不能被改变。

#### 3. then方法

一个Promise必须提供一个then方法来获取其值或原因。
Promise的then方法接受两个参数：

```javascript
    promise.then(onFulfilled, onRejected)
```

**参数可选**
onFulfilled 和 onRejected 都是可选参数。

* 如果 onFulfilled 不是函数，其必须被忽略
* 如果 onRejected 不是函数，其必须被忽略
  
**onFulfilled 特性**
如果 onFulfilled 是函数：

* 当 promise 执行结束后其必须被调用，其第一个参数为 promise 的终值
* 在 promise 执行结束前其不可被调用
* 其调用次数不可超过一次

**onRejected 特性**
如果 onRejected 是函数：

* 当 promise 被拒绝执行后其必须被调用，其第一个参数为 promise 的据因
* 在 promise 被拒绝执行前其不可被调用
* 其调用次数不可超过一次

**多次调用**
then 方法可以被同一个 promise 调用多次

* 当 promise 成功执行时，所有 onFulfilled 需按照其注册顺序依次回调
* 当 promise 成功执行时，所有 onFulfilled 需按照其注册顺序依次回调

**返回**
then 方法必须返回一个 promise

```javascript
    promise2 = promise1.then(onFulfilled, onRejected);
```

* 如果 onFulfilled 或者 onRejected 返回一个值 x ，则运行下面的 Promise 解决过程：Resolve(promise2, x)
* 如果 onFulfilled 或者 onRejected 抛出一个异常 e ，则 promise2 必须拒绝执行，并返回拒因 e
* 如果 onFulfilled 不是函数且 promise1 成功执行， promise2 必须成功执行并返回相同的值
* 如果 onRejected 不是函数且 promise1 拒绝执行， promise2 必须拒绝执行并返回相同的据因

#### 4. Promise解决过程

Promise 解决过程是一个抽象的操作，其需输入一个 promise 和一个值，我们表示为 [[Resolve]](promise, x)，如果 x 有 then 方法且看上去像一个 Promise ，解决程序即尝试使 promise 接受 x 的状态；否则其用 x 的值来执行 promise 。

这种 thenable 的特性使得 Promise 的实现更具有通用性：只要其暴露出一个遵循 Promise/A+ 协议的 then 方法即可；这同时也使遵循 Promise/A+ 规范的实现可以与那些不太规范但可用的实现能良好共存。

运行 [[Resolve]](promise, x) 需遵循以下步骤：

##### x 与 promise 相等

* 如果 promise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 promise

##### x 为 Promise

* 如果 x 为 Promise ，则使 promise 接受 x 的状态：

* 如果 x 处于等待态， promise 需保持为等待态直至 x 被执行或拒绝
* 如果 x 处于执行态，用相同的值执行 promise
* 如果 x 处于拒绝态，用相同的据因拒绝 promise

##### x 为对象或函数

如果 x 为对象或者函数：

* 把 x.then 赋值给 then
* 如果取 x.then 的值时抛出错误 e ，则以 e 为据因拒绝 promise
* 如果 then 是函数，将 x 作为函数的作用域 this 调用之。传递两个回调函数作为参数，第一个参数叫做 resolvePromise ，第二个参数叫做 rejectPromise:
  * 如果 resolvePromise 以值 y 为参数被调用，则运行 [[Resolve]](promise, y)
  * 如果 rejectPromise 以据因 r 为参数被调用，则以据因 r 拒绝 promise
  * 如果 resolvePromise 和 rejectPromise 均被调用，或者被同一参数调用了多次，则优先采用首次调用并忽略剩下的调用
  * 如果调用 then 方法抛出了异常 e：
    * 如果 resolvePromise 或 rejectPromise 已经被调用，则忽略之
    * 否则以 e 为据因拒绝 promise
  * 如果 then 不是函数，以 x 为参数执行 promise
* 如果 x 不为对象或者函数，以 x 为参数执行 promise

<!-- more -->

#### 5. 实现代码

```javascript
    const PENDING = 'PENGDING'; // 默认等待状态
    const FULLFILLED = 'FULLFILLED'; // 成功态
    const REJECTED = 'REJECTED' // 失败态

    // 考虑 x 可能是外部的promise
    function resolvePromise(x, promise2, resolve, reject){
        // promiseA+ 规范 ：If promise and x refer to the same object, reject promise with a TypeError as the reason
        if (x === promise2){
            return new TypeError('循环引用')
        }
        // 1. 如果x 是一个普通值 则直接调用resolve即可
        // 2. 如果x 是一个promise那么应该采用这个promise的状态 决定调用的是 resolve还是reject
        if(((typeof x === 'object' && x !== null)) || (typeof x == 'function')){
            // 是对象或者函数才有可能是个promise
            let called = false;
            try {
                // 取then方法 并且用try捕捉可能出现的错误 防止别人的promise中设置了取值的限制
                // 比如：Object.defineProperty(x,'then',{
                //     get(){
                //         if(times ==2){
                //             throw new Error()
                //         }
                //     }
                // })
                let then = x.then 
                if (typeof then == 'function'){
                    // 用call不用x.then是为了少取一次then少触发get方法
                    then.call(x,y=>{ // y也可能是个prmosie 要递归处理
                        if (called) return;
                        called = true
                        // 不停的解析成功的promise中返回的成功值，直到这个值是一个普通值
                        resolvePromise(y, promise2, resolve, reject);
                    },r=>{
                        if (called) return;
                        called = true
                        reject(r);
                    })
                }
            }catch(e){
                if (called) return;
                called = true
                reject(e); // 让promise2 变成失败态
            }
        }else {
            // x 是一个普通值 
            resolve(x)
        }
    }
    class Promise{
        constructor(executor){
            this.status = PENDING;
            this.value = undefined; // 成功时的值
            this.reason = undefined;  // 失败时的值
            // 存在异步逻辑时先存放回调函数 ，异步完成时在resolve/reject再遍历其中的回调依次执行
            this.onResolvedCallbacks = [];
            this.onRejectedCallbacks = [];
            // 调用resolve和reject可以将对应的结果暴露在当前的promise实例上

            // 为什么resolve，reject不写在原型上 因为  每个promise有自己的resolve，reject
            const resolve = (value)=>{ 
                // 只有状态在PENDING时才能修改状态 保证只能调用一次resolve/reject,状态修改不可逆，
                if(value instanceof Promise){
                    return value.then(resolve,reject)
                }
                if (this.status === PENDING){
                    this.value = value;
                    this.status = FULLFILLED;
                    this.onResolvedCallbacks.forEach(fn=>fn())
                }
            }
            const reject = (reason) => {
                if (this.status === PENDING) {
                    this.reason = reason;
                    this.status = REJECTED;
                    this.onRejectedCallbacks.forEach(fn => fn())
                }
            }
            try {
                // 默认new Promise中的函数会立即执行
                executor(resolve, reject)
            }catch(e){
                // throw new Error或者执行出错时，需要将错误传递到reject中，执行失败的逻辑
                reject(e)
            }
        
        }
        
        // 在then方法(成功和失败)中 返回一个promise， promose会采用返回的promise的成功的值或失败原因, 传递到外层下一次then中

        // 1. then方法中 成功的回调或者失败的回调返回的是一个promise，那么会采用返回的promise的状态，走外层下一次then中的成功或失败， 同时将promise处理后的结果向下传递
        // 2.then方法中 成功的回调或者失败的回调返回的是一个普通值 （不是promise） 这里会将返回的结果传递到下一次then的成功中去
        // 3.如果在then方法中 成功的回调或者失败的回调 执行时出错会走到外层下一个then中的失败中去
        then(onFulfilled,onRejected){
            debugger
            // 可选参数 then中的onFulfilled,onRejected为空则将上个then中的value或者reason传递到下个then中
            onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v=>v;
            onRejected = typeof onRejected === 'function' ? onRejected : e => {throw(e)}
            console.log(onFulfilled)
            let promise2 = new Promise((resolve,reject)=>{
                if (this.status === FULLFILLED){
                    setTimeout(() => {
                        try {
                            let x = onFulfilled(this.value)
                            resolvePromise(x, promise2, resolve, reject)
                        }catch(e){
                            reject(e)
                        }
                    }, 0);
                }
                if (this.status === REJECTED) {
                    setTimeout(() => {
                        try {
                            let x = onRejected(this.reason)
                            resolvePromise(x, promise2, resolve, reject)
                        } catch (e) {
                            reject(e)
                        }
                    }, 0);
                }
                // 执行then方法状态是PENDING时 表示存在异步逻辑 使用发布订阅模式先将回调保存,异步结束调用resolve/reject再处理
                if (this.status === PENDING) {  
                this.onResolvedCallbacks.push(()=>{
                    setTimeout(() => {
                        try {
                            let x = onFulfilled(this.value)
                            resolvePromise(x, promise2, resolve, reject)
                        } catch (e) {
                            reject(e)
                        }
                    }, 0);
                })
                    this.onRejectedCallbacks.push(() => {
                        setTimeout(() => {
                            try {
                                let x = onRejected(this.reason)
                                resolvePromise(x, promise2, resolve, reject)
                            } catch (e) {
                                reject(e)
                            }
                        }, 0);
                    })
                }
            })
            return promise2
        }
        catch(errFn){
            return this.then(null, errFn)
        }

        static resolve(value){
            return new Promise((resolve,reject)=>{
                resolve(value)
            })
        }

        static reject(reason){
            return new Promise((resolve,reject)=>{
                reject(reason)
            })
        }

        static all(promises){
            return new Promise((resolve,reject)=>{
                let result = []
                let index = 0
                function process(v, k) {
                    result[k] = v;
                    if (++index == promises.length) { // 解决多个异步并发问题 靠计数器
                        resolve(result);
                    }
                }
                for (let i = 0; i < promises.length; i++) {
                    let p = promises[i]
                    if(p && typeof p.then === 'function'){
                        p.then(data=>{
                            process(data, i);// 异步的
                        },reject)
                    }else {
                        process(p, i);// 同步的
                    }
                }
            })
        }
    }
    module.exports = Promise
```

#### 6.promise 使用例子

```javascript
    const Promise = require('./myPromise')

    let promise  = new Promise((resolve,reject)=>{
        setTimeout(() => {
            reject('第一个error')
            resolve('ok')
        }, 1000);
    })
    // // 
    let promise2 = promise.then(value=>{
        console.log(value,'success')
        // return new Promise((r,e)=>{e('e')})
    },reason=>{
        console.log(reason,'fail')
        return new Promise((resolve,reject)=>{
            reject('新promise')
        })
    })
    promise2.then().then().then(value => {
        console.log(value, 'success2')
    }, reason => {
        console.log(reason, 'fail2')
    })

    promise2.then().then().then(value => {
        console.log(value, 'success2')
    }).catch(err => {
        console.log(err, 'err');
    })

    Promise.resolve(321312312).then(data=>{
        console.log('Promise.resolve',data)
    })


    Promise.resolve(new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('延时')
        }, 2000);
    })).then(data=>{
        console.log(data)
    })

    Promise.reject('错误').catch(err => {
        console.log('Promise.reject', err)
    })


    const fs = require('fs').promises


    // Promise.all 表示全部成功才成功， 如果一个失败了 则失败
    Promise.all([fs.readFile(__dirname + '/5.class.js', 'utf8'), fs.readFile(__dirname + '/4.reduce.js', 'utf8'), 11]).then(data => {
        console.log(data);
    }).catch(err => {
        console.log(err)
    })
    Promise.resolve(1).then(2).then(Promise.resolve(3)).then(x=>console.log(x))
```
