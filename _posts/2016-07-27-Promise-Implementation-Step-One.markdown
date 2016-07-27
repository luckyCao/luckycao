---
layout: post
title:  "三步实现promise-第一步"
date:   2016-07-27 23:09:02 +0800
categories: javascript promise
---
promise的实现参照[Forbes Lindesay](https://www.promisejs.org/implementing/)
mark一下理解过程

第一步
{% highlight javascript %}
   let promise = new Promise((resolve,reject)=>{
      setTimeout((value)=>{
         resolve(10)
      },1000)
   })
   promise.then((value)=>{
      console.log(value);
   },()=>{})
{% endhighlight %}

从上面的代码可以看出
我们需要实现一个构造函数(记为Promise)
Promise接收一个函数(记为fn)为参数，并且立刻执行fn，从resolve(10)可以看出来fn的两个入参resolve和reject都是函数，
resolve和reject都是Promise传给fn的。
Promise返回一个对象，这个对象有一个then方法,它的两个入参也是函数

重点是当resolve(10)被执行时then的第一个入参(onFulfilled)被执行,log被打印
要实现这个功能点，需要在then的实现中把then接收的两个入参(onFulfilled,onRejected)存在一个变量中，
执行resolve的时候把onFulfilled从变量中取出来并执行
实现如下
{% highlight javascript %}
   const PENDING   = 0;
   const FULFILLED = 1;
   const REJECTED  = 2;
   function Promise(fn){
     let state = PENDING,
         value = null,
         handlers = [];
     function handle(obj){
       if(state === PENDING){
         handlers.push(obj)
       }
       if(state == FULFILLED){
         obj.onFulfilled(value);
       }
       if(state == REJECTED){
         obj.onRejected(value);
       }
     }
     function fulfill(val){
       state = FULFILLED;
       value = val;
       handlers.map(handler=>{
         handle(handler)
       })
     }
     function reject(val){
       state = REJECTED;
       value = val;
       handlers.map(handler=>{
         handle(handler)
       })
     }
     function doResolve(fn,fulfill,reject){
       fn(fulfill,reject)
     }
     this.then = function(onFulfilled,onRejected){
       handle({
         onFulfilled:onFulfilled,
         onRejected:onRejected
       });
     }
     doResolve(fn,fulfill,reject)
   }
   export default Promise
{% endhighlight %}







