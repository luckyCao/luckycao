---
layout: post
title:  "三步实现promise-第二步"
date:   2016-07-27 23:09:02 +0800
categories: javascript promise
---
第二步
{% highlight javascript %}
   (1)let promise = new Promise((resolve,reject)=>{
      setTimeout((value)=>{
         resolve(10)
      },1000)
   })
   promise.then((value)=>{
     console.log(value);
     return ++value;
   },()=>{}).then(value=>{
     console.log(value)
   })
{% endhighlight %}

第二步要实现的是then能链式调用当上述代码执行的时候 打印出 10 11
也就是说then要返回一个有then方法的对象，即then的内部新建了一个promise对象并返回
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
     this.done = function(onFulfilled,onRejected){
       handle({
         onFulfilled:onFulfilled,
         onRejected:onRejected
       })
     }
     this.then = function(onFulfilled,onRejected){
       var self = this;
       return new Promise((resolve,reject)=>{
         self.done((value)=>{//注意这里(2)
           resolve(onFulfilled(value))
           //这句代码是关键所在
           //(1)这里新建了一个promise对象记为(p1)，当p1的resolve被执行的时候(2)会被执行，
           //然后p1的onFulfilled会被执行打印log 10
           //p1的then新建了一个promise对象记为(p2),
           //当p1的resolve被执行的时候(2)会被执行resolve(onFulfilled(value))会被执行
           //这里的resolve是p2的因此p2的onFulfilled会被执行打印log 11
         },()=>{})
       })
     }
     doResolve(fn,fulfill,reject)
   }
   export default Promise
{% endhighlight %}




