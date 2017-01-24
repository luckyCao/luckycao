---
layout: post
title:  "实现一个简单sizzle选择器"
date:   2016-08-07 13:33:02 +0800
categories: sizzle
---
mark一下对sizzle选择器实现的理解，并且抽出一个简单的选择器，不考虑其对seed的优化，不考虑缓存，不考虑多个选择器（逗号分隔），不考虑位置选择器等复杂的选择器。
只实现一个支持 类选择器，标签选择器，后代元素选择器的lite-sizzle
{% highlight javascript %}
     <html>
        <head></head>
        <body>
            <div class='one'>
               <div>
                  <div>
                     <p class='two'></p>
                  </div>
               </div>
            </div>
        </body>
     </html>
{% endhighlight %}
根据以上的树形结构，如何实现一个能选择出 selecter = 'one p.two' 的lite-sizzle？
首先需要将selecter分解成token，如何分解不是这篇文章的重点，我们只要知道有一个tokenize函数可以帮你把selecter分解成tokens。
要分解的理由也很简单，我们需要找到一个元素，它要满足四个条件：

1.它是p标签；

2.它的class包含two；

3.找到他的祖先元素；

4，祖先元素的class包含one。
所以我们需要把selector分解，就是tokenize这个selector。
分解之后的tokens如图1

为了理解sizzle是怎么运作的有必要讲一下curry函数，curry函数是闭包的一个应用，其实就是返回一个函数的函数，js中函数是第一类变量，可以作为返回值：
{% highlight javascript %}
var Expr = {
  relative: {
    " ": { dir: "parentNode" }
  },
  filter: {
    "TAG": function( nodeNameSelector ) {
      var nodeName = nodeNameSelector.toLowerCase();
      return function( elem ) {
          return elem.nodeName && elem.nodeName.toLowerCase() === nodeName;
        };
    },
    "CLASS": function( className ) {
      var pattern = new RegExp( "(^|" + whitespace + ")" + className + "(" + whitespace + "|$)" );
      return function( elem ) {
        return pattern.test( typeof elem.className === "string" && elem.className || typeof elem.getAttribute !== "undefined" && elem.getAttribute("class") || "" );
      };
    }
  }
};
{% endhighlight %}
上面filter对象的两个属性就是curry函数，他们接收一个参数，返回一个函数。
filter.TAG('p')返回一个能匹配div标签的函数
filter.CLASS('one')返回一个能匹配一个class包含one的元素。
所以要匹配标签和类还是很简单的，难点在匹配关系选择器：后代元素选择器。

现在我们已经把selector转化成了tokens，已经知道怎么匹配标签和类了，那么怎么去实现选择器呢，先确定一下选择器的功能，
选择器是一个函数，他接收selector，tokens，和一个空数组results,并且把完成匹配的元素放到数组中。
怎么实现呢？首先选择器从右向左去匹配效率比较高，这个也是耳熟能详的概念了，花点时间就能想明白。
seeds可以取dom中的所有标签，很自然的想法是遍历seeds，这是一层循环，内部遍历tokens，逐个匹配。代码如下：

{% highlight javascript %}
    var seeds = document.getElementsByTagName('*'),
        len = seeds.length,matcher;
    for(var i = 0;i<len;i++){//遍历目标集合，
        var elem = seeds[i];
        for(var j=tokens.length-1;j>=0;j--){//针对tokens中的四个条件逐条验证
            if ( (matcher = Expr.relative[ match[j].type ]) ) {//祖先选择器' '
               while(elem = elem['parentNode']){

               }
            } else {//非关系选择器

            }
        }
    }
{% endhighlight %}

这个代码有个不可行的地方就在关系选择器这边，假定我们外层循环已经循环到<p class='two'></p>,内层循环已经进行到确定了这个元素是p标签，
它的class包含two，这个时候上面的代码执行到j=2。现在需要判断它是否有一个祖先元素，这个祖先元素的class包含two。这两个判断的逻辑很难处理，
首先要通过elem['parentNode']得到他的父元素，j---,然后再判断这个父元素的class是否包含two，如果不包含,就要j++回退到j==2的时候，找他的父父元素。。
所以这样写代码会逻辑判断非常复杂非常的难以维护。

那么应该怎么处理呢 ，sizzle的做法是用tokens这四个条件生成一个组合起来的选择器，再用这个选择器去选择seeds。也就是保留了上面代码的外层循环，
内部循环改成了，不直接去通过tokens中四个单独的条件一个个判断，而是把这四个条件对应的选择函数组合起来,生成了一个满足四个条件的匹配器。代码如下：
{% highlight javascript %}
function matcherFromTokens( tokens ) {
  var  matcher,len = tokens.length,i = 0,matchers = [];

  for ( ; i < len; i++ ) {
    if ( (matcher = Expr.relative[ tokens[i].type ]) ) {
      matchers = [ addCombinator(elementMatcher( matchers ), matcher) ];
    } else {
      matcher = Expr.filter[ tokens[i].type ].apply( null, tokens[i].matches );
      matchers.push( matcher );
    }
  }
  return elementMatcher( matchers );
}

function addCombinator( matcher, combinator ) {
  var dir = combinator.dir;
  return function( elem, context, xml ) {
    while ( (elem = elem[ dir ]) ) {
      if ( elem.nodeType === 1 ) {
        if ( matcher( elem, context, xml )) {
          return true;
        }
      }
    }
  };
}

function elementMatcher( matchers ) {
  return matchers.length > 1 ?
    function( elem, context, xml ) {
      var i = matchers.length;
      while ( i-- ) {
        if ( !matchers[i]( elem, context, xml ) ) {
          return false;
        }
      }
      return true;
    } :
    matchers[0];
}
{% endhighlight %}

matcherFromTokens函数的作用就是根据tokens中的四个条件生成一个终极匹配器，它从左到右遍历tokens，这样生成一个终极匹配器函数之后，
匹配是从右向左执行的。

1.matcherFromTokens遇到的第一个匹配器是.one,通过Expr.filter[ tokens[i].type ].apply( null, tokens[i].matches )生成了一个能匹配类包含one的小选择器，
把这个小的选择器放到数组matchers中。

2.matcherFromTokens遇到的第二个匹配器是关系选择器“ ”，  通过addCombinator(elementMatcher( matchers ), matcher)生成一个组合了的选择器，并放到matchers中。
addCombinator(elementMatcher( matchers ), matcher)这句执行的时候，首先执行的是elementMatcher( matchers )，当matchers的长度是1的时候，返回matchers[0]。
所以addCombinator(elementMatcher( matchers ), matcher)改写成addCombinator(matchers[0], matcher),其中matchers[0]就是能匹配类one的类选择器。
addCombinator(matchers[0], matcher)生成一个函数：
{% highlight javascript %}
function( elem, context, xml ) {
    while ( (elem = elem[ dir ]) ) {
      if ( elem.nodeType === 1 ) {
        if ( matcher( elem, context, xml )) {   //这里的matcher就是addCombinatorjie'so
          return true;
        }
      }
    }
}
{% endhighlight %}
这个函数的作用是接收一个元素，当这个元素的某个祖先元素包含类one的时候返回true，否则返回false

3.matcherFromTokens遇到的第三个匹配器是标签选择器 p，这时候他通过Expr.filter[ tokens[i].type ].apply( null, tokens[i].matches )生成一个小的标签选择器，并且放到数组matchers中。

4.matcherFromTokens遇到的第四个选择器是.two,这时候的处理和1，3相同

最后通过elementMatcher( matchers )返回一个满足tokens所表示的四个条件的选择器。这个选择器先判断元素是否包含类two，如果否返回false，如果是
再判断元素是否是p标签，如果否返回false，如果是再判断这个元素是否有一个祖先元素包含类one如果是返回true。

选择seeds到循环判断每个seeds是否符合matcherFromTokens生成的终极匹配器体现在compile函数中，代码如下，compile首先通过matcherFromTokens生成一个终极匹配器，
之后返回一个函数，返回的函数通过document.getElementsByTagName('*')获取到目标集合，然后遍历这个目标集合，用matcherFromTokens生成的终极匹配器去判断这个元素是否符合
否符合tokens所描述的四个条件，如果符合把元素放到results中。
{% highlight javascript %}
function compile( selector, match) {
  var fun,
    elementMatchers = [];
  fun = matcherFromTokens( match);
  elementMatchers.push( fun );
  return function( context, xml, results, outermost ) {
             var elem, j, matcher,
                 i = 0,
                 seeds = document.getElementsByTagName('*'),
                 len = seeds.length;
             for ( ; i !== len && (elem = seeds[i]) != null; i++ ) {
                 j = 0;
                 while ( (matcher = elementMatchers[j++]) ) {
                   if ( matcher( elem, context || document, xml) ) {
                     results.push( elem );
                     break;
                   }
                 }
             }
         };
};
{% endhighlight %}

完整的代码见
[react-test Chapter3](https://github.com/luckyCao/react-test)


