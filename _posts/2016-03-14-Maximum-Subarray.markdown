---
layout: post
title:  "最大连续子串"
date:   2016-03-14 23:39:02 +0800
categories: algorithm
---
求一个数组的最大子数组和


{% highlight javascript %}
/**
 * 无法根据条件确定最大子串的起始位置，
 * 所以必须遍历数组，求和寻找和为正数的数组，
 * 用maxSoFar去记录最大的正数和
 * @param arr
 * @returns {{sum: number, begin: number, len: number}}
 */

function solve(arr){
    var i,maxSoFar=0,maxTemp= 0,
        n= 0,count= 0,maxNeg=arr[0];
    for(i=0;i<arr.length;i++){
        maxTemp =maxTemp+arr[i];
        count++;
        if(maxTemp<0){
            maxTemp=0;
            n = i+1;
            count = 0;
        }
        if(maxSoFar<maxTemp){
            maxSoFar = maxTemp;
        }
        if(arr[i]>maxNeg){
            maxNeg = arr[i];
        }
    }
    if(maxSoFar==0){
        return {sum:maxSoFar,begin:n,len:count}
    }
    //如果没有正数子串，就返回数组最大元素
    else{
        return maxNeg;
    }

}

{% endhighlight %}

