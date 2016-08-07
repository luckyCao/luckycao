---
layout: post
title:  "实现一个简单的react-redux"
date:   2016-08-07 13:33:02 +0800
categories: 深入react技术栈
---
react-redux是一个将react和redux很好的结合起来的一个工具，他可以免去你自己写下面的代码，
你可以不用手动监听store，不用将store传到子组件，然后显试的调用dispatch(action)。react-redux帮你做了这些工作。代码如下
{% highlight javascript %}
    let store = createStore(counter)
    class RootC2 extends React.Component {
        render(){
            return(
                <Provider store={store}>
                    <Test />
                </Provider>
            )
        }
    }
    class Test extends React.Component {
        render () {
            return (
                <section className="grid-box">
                    <div className="info-text ma-lr14">
                        <p>{this.props.count}</p>
                    </div>
                    <button onClick={this.props.add}>+</button>
                    <button onClick={this.props.reduce}>-</button>
                </section>
            );
        }
    }
    const mapStateToProps = (state)=>{
        return {count:state}
    }
    export default connect(mapStateToProps,actionCreaters)(Test)
{% endhighlight %}

react-redux由provider和connect组成
Provider所做的事是通过[context](https://facebook.github.io/react/docs/context.html)把props传递给子孙组件。
一旦你想获取到context中的props，你需要connect(mapStateToProps,actionCreaters)(Test)，把你自定义的组件作为输入值传入connect
mapStateToProps定义了你要从provider中传的props所拿的值，而connect会根据actionCreaters生成相应的dispatch函数例如，如果actionCreaters是
{% highlight javascript %}
    {
        add:(){
                return {
                    type:'ADD'
                }
        }
        reduce:(){
                return {
                    type:'REDUCE'
                }
        }
    }
{% endhighlight %}
则connect会根据actionCreaters生成
{% highlight javascript %}
    {
        add:(){
           return dispatch(add.apply(undefined, arguments));//add.apply中的add就是上面的add
        },
        reduce:(){
           return dispatch(reduce.apply(undefined, arguments));
        },
    }
{% endhighlight %}
并且connect会爸生成的这个对象作为props传递给自定义组件Test，所以Test的代码有
{% highlight javascript %}
    <button onClick={this.props.add}>+</button>
    <button onClick={this.props.reduce}>-</button>
{% endhighlight %}

connect还会调用store的subscribe方法监听store的变化，一旦dispatch(action)被调用，connect中的this.handleChange
就被调用，handleChange通过store.getState()获取最新数据并调用connect的setState导致connect的render被调用
{% highlight javascript %}
    trySubscribe() {
        this.unsubscribe = this.store.subscribe(this.handleChange.bind(this))
        this.handleChange()
    }
    componentDidMount() {
        this.trySubscribe()
    }
    handleChange() {
        const storeState = this.store.getState()
        this.setState({ storeState })
    }
{% endhighlight %}
connect（简化版）的render方法如下，会从更新了的store中获取mapStateToProps指定获取的值，并返回自定义组件的虚拟dom
{% highlight javascript %}
   render() {
       this.configureFinalMapState(this.store);
       this.configureFinalMapDispatch(this.store);
       this.mergeProps();
       this.renderedElement = createElement(WrappedComponent,this.mergedProps)
       return this.renderedElement
   }
{% endhighlight %}

完整的代码见
[react-test Chapter2](https://github.com/luckyCao/react-test)