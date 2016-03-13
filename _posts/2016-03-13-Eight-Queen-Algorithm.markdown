---
layout: post
title:  "八皇后问题"
date:   2016-03-03 18:55:02 +0800
categories: algorithm
---
问题描述:在8×8格的国际象棋上摆放八个皇后，使其不能互相攻击，即任意两个皇后都不能处于同一行、同一列或同一斜线上，问有多少种摆法

{% highlight javascript %}
//八皇后问题
//用数组[x1,x2,x3,...]代表棋盘，
// 数组的下标y1,y2...分别代表棋盘的列，
// 数据下标对应的数值x1,x2...代表棋盘queen所在的行
    function output(){
        console.log(count++);
        console.log(res);
    }
    var res=[],
        count = 0;

    function isValid(row,col){
        //数组中的每个数都应该不同，（不能同行）
        //数组下标和其对应的数组的和不能相同，差不能相同，（不能在同一条斜线上）
        var i;
        for(i = 0;i<col; i++ ){
            if(res[i] == res[col]){
                return false;
            }
            if(((res[i]+i)==(res[col]+col))||((res[i]-i)==(res[col]-col))){
                return false;
            }
        }
        return true;
    }
    //在col-1列已经放置好queen的情况下，放置col列的queen
    function solve(col,n){
        var row;
        //每一列有八个行可以挑选放置
        for(row = 0;row<n ;row++){
            res[col] = row; //将queen放在col列的row行
            if(isValid(row,col)){
                if((col==(n-1))&&(res.length == n )){
                //这里的col＝(n-1)条件很重要，就是开始漏掉了调试了好久。。
                    output();
                }else{
                    solve(col+1,n);
                }
            }
        }

    }
    solve(0,8);
{% endhighlight %}

