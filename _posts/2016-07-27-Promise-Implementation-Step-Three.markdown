---
layout: post
title:  "三步实现promise-第三步"
date:   2016-07-27 23:09:02 +0800
categories: javascript promise
---
第三步
不要怕麻烦从(1)->(12)过一遍 这里的函数作用域是关键
一旦理解了(1)到(12)的注释 就理解了promise的实现
{% highlight javascript %}
   let promise = new Promise((resolve,reject)=>{
   //(1)这里创建了一个promise对象记为(p1)
     setTimeout((value)=>{
       //(4)一秒钟后p1的resolve(10)被执行
       resolve(10)
     },1000)
   })
   //(2)这里p1的then函数创建了一个promise对象p2
   promise.then((value)=>{
     //(5)因为p1的resolve被执行所以p1相应的onFulfilled函数被执行打印了log 10
     console.log(value);
     //(6)接下来新建了一个promise对象(记为p4)，重点是这个对象返回了
     return new Promise((resolve,reject)=>{
       setTimeout((value)=>{
         resolve(20)//(9)再一秒之后p4的resolve被执行
       },1000)
     });
   },()=>{}).then(value=>{ //(3)这是p2的then函数,它创建创建了一个promise对象p3
     console.log(value)//(12)打印log 20
   })
{% endhighlight %}


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
  //第三布的关键就在resolve
  function resolve(val){
    if(!val){
      return;
    }
    let then = getThen(val);
    //当onfulfilled返回的是promise的情况
    if(then){
      //(8)这里的作用域是p2,而这个then是p4的,val就是p4,p4的then方法被调用
      then.call(val,(value)=>{//(10)p4的onFulfilled方法被调用
        fulfill(value);//(11)p2的fulfill被执行(这里是关键，这里的fulfill的作用域是p2)
      },()=>{})
    }
    else{
      fulfill(val);
    }
  }
  function getThen(val){
    return val.then?val.then:null;
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
      self.done((value)=>{//(2)
        resolve(onFulfilled(value))
        //(7)p1相应的onFulfilled函数返回了p4,并进入p2的resolve方法
      },()=>{})
    })
  }
  doResolve(fn,resolve,reject)
}
export default Promise
{% endhighlight %}





