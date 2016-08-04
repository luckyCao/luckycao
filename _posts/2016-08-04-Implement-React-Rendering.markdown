---
layout: post
title:  "react的渲染流程是如何实现的"
date:   2016-08-04 20:09:02 +0800
categories: 深入react技术栈
---
如今react俨然已成前端开发的标配，但是知道react是怎样从jsx到虚拟dom再从虚拟dom生成dom的人不多。从15年接触react到现在一直想看
react源码，但是一看到chrome开发者工具上的调用栈就望而却步，感谢react-lite的实现者，让我理解了react的渲染流程。mark一下我的理解，
并且抽出一个简单的纯粹的render流程（不考虑生命周期，state之类的）

jsx的语法是这样的
{% highlight javascript %}
    class root2 extends React.Component {
      render () {
        return (
          <Layout>
            <Test></Test>
          </Layout>
        );
      }
    }
{% endhighlight %}

用jsx translator翻译之后如下，这里我做了简化处理，用了自定义的简单的构造函数。
{% highlight javascript %}
    function root1(props){
        return {
            props:props,
            render:function() {
                return  createElement(
                    Layout,
                    null,
                    createElement(Test)
                );
            }
        }
    }
{% endhighlight %}

这里reactElement的作用就是生成虚拟dom，虚拟dom到底是个啥，其实它就是个javascript对象~，这个对象的属性有props，vType，type，
而props也是个对象，它有children属性也有其他的比如className，onClick之类的，因为这里讲的是渲染流程，所以我们只需要知道props里面有
children就行了。
所以上面的render方法执行之后就生成了如下的虚拟dom,root1这个自定义的组件的render函数执行后生成了一个虚拟dom，这个虚拟dom的type是
layout1组件，props里有一个属性children，这个children也是个虚拟dom，它的类型是自定义组件test1，vtype是2。所以虚拟dom也是树型结构的
{% highlight javascript %}
    {
      vtype:3,
      type:function layout1(props)...,
      props:{
        children:{
          vtype:3,
          type:function test1(props)...,
          props:{}
        }
      }
    }
{% endhighlight %}


再来看一下layout1自定义组件
jsx语法：
{% highlight javascript %}
    class layout2 extends React.Component {
      render () {
        return (
          <div className='page-container'>
            <div className='view-container'>
              {this.props.children}
            </div>
          </div>
        );
      }
    }
{% endhighlight %}
翻译之后（简化版本）
{% highlight javascript %}
    function layout1(props){
        return {
            props:props,
            render:function() {
                var children = this.props.children;
                return createElement(
                    'div',
                    { className: 'page-container' },
                    createElement(
                        'div',
                        { className: 'view-container' },
                        children
                    )
                );
            }
        }
    }
{% endhighlight %}

layout1的render方法生成的虚拟dom是：
{% highlight javascript %}
    {
        vtype:2,
        type:'div',
        props:{
            className:'page-container',
            children:{
                vtype:2,
                type:'div',
                props:{
                    className:'view-container',
                    children:{
                        type:function test1(props)...,
                        vtype:3,
                        props:{

                        }
                    }
                }
            }
        }
    }
{% endhighlight %}

所以说虚拟dom的vtype是3的时候对应的type是自定义组件，vtype是2的时候是对应的type是div之类的浏览器原生组件。

上面做了一下jsx，对应的js，虚拟dom的介绍，也知道了createElement的作用就是生成虚拟dom，那么重点来了，虚拟dom又是怎么转化成dom的呢
这就涉及到一个递归函数initVnode，initVnode接收一个参数：虚拟dom，返回一个参数dom

{% highlight javascript %}
   var dom = initVnode(虚拟dom)
{% endhighlight %}

initVnode函数接收一个虚拟dom，首先判断vtype是几，
如果是3 那么对应的type就是自定义的组件，就需要new一个对象，然后调用这个对象的render方法，这个组件的render方法返回的仍然是个虚拟dom，
这时候就轮到递归上场了，调用自己去把这个虚拟dom转化成dom节点，并返回这个节点
{% highlight javascript %}
     let component = new Component(vNode.props);
     let virtualNode = component.render();
     return initVnode(virtualNode);
{% endhighlight %}


如果是2 那么对应的type是浏览器原生组件，例如div，这个时候就document.createElement(type),注意到虚拟dom的props属性中的children属性，
这个children也是虚拟dom，children有可能是一个虚拟dom，有可能是一个虚拟dom数组，先把children转化成数组，
遍历这个数组，用initVnode把虚拟dom转化成dom节点，再把dom节点appendChild到他们的父组件上，并返回父组件
{% highlight javascript %}
     let type = vNode.type;
     let node = document.createElement(type);
     let children = vNode.props.children;
     if(!(children instanceof Array)){
       children = [children]
     }
     let length = children.length;
     for(var i=0;i<length;i++){
         node.appendChild(initVnode(children[i]));
     }
     return node
{% endhighlight %}

完整的代码见
[react-test Chapter1](https://github.com/luckyCao/react-test)











