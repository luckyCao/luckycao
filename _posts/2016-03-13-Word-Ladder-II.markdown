---
layout: post
title:  "Word Ladder II"
date:   2016-03-03 22:19:02 +0800
categories: algorithm
---
Given two words (beginWord and endWord), and a dictionary's word list, find all shortest transformation sequence(s) from beginWord to endWord, such that:

Only one letter can be changed at a time
Each intermediate word must exist in the word list
For example,

Given:
beginWord = "hit"
endWord = "cog"
wordList = ["hot","dot","dog","lot","log"]

{% highlight javascript %}
var findLadders = function(beginWord, endWord, wordList) {
    var res = [],
        resultSet = [],minLen;
    function diffOne(w1,w2){
        var i,j;
        for(i=0,j=0;i<w1.length;i++){
            if(w1[i]!=w2[i]){
                j++;
            }
        }
        return j==1?true:false;
    }
    function isValid(begin,word){
        var i;
        for(i=0;i<res.length-1;i++){
            if(res[i]==word){
                return false;
            }
        }
        return diffOne(begin,word);
    }
    function clone(o){
        var r=[],i;
        for(i=0;i< o.length;i++){
            r[i] = o[i];
        }
        return r;
    }
    function solve(begin,end){
        var i;
        for(i = 0;i<wordList.length;i++){
            //寻找与begin相差一个字母，和end相差字母最少的
            res.push(wordList[i]);
            if(isValid(begin,wordList[i])){
                if(wordList[i]== end){
                    resultSet.push(clone(res));
                    if(resultSet.length==1){
                        minLen = res.length;
                    }
                    else{
                        minLen = (minLen<res.length)?minLen:res.length;
                    }
                }
                else{
                    solve(wordList[i],end);
                }
            }
            res.pop();
        }
    }
    solve(beginWord,endWord);
    var answer = [];
    for(var i =0;i<resultSet.length;i++){
        if(resultSet[i].length==minLen){
            answer.push([beginWord].concat(resultSet[i]));
        }
    }
    return answer;
};
{% endhighlight %}

