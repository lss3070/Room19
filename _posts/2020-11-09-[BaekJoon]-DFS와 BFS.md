---
title: "[백준/JavaScript] DFS와 BFS"
subtitle: "알고리즘 풀이"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    -백준,
    -DFS,
    -BFS,
    -Algorithm
---


문제 설명
-------
그래프를 DFS로 탐색한 결과와 BFS로 탐색한 결과를 출력하는 프로그램을 작성하시오. 단, 방문할 수 있는 정점이 여러 개인 경우에는 정점 번호가 작은 것을 먼저 방문하고, 더 이상 방문할 수 있는 점이 없는 경우 종료한다. 정점 번호는 1번부터 N번까지이다.

입력
-------
첫째 줄에 정점의 개수 N(1 ≤ N ≤ 1,000), 간선의 개수 M(1 ≤ M ≤ 10,000), 탐색을 시작할 정점의 번호 V가 주어진다. 다음 M개의 줄에는 간선이 연결하는 두 정점의 번호가 주어진다. 어떤 두 정점 사이에 여러 개의 간선이 있을 수 있다. 입력으로 주어지는 간선은 양방향이다.

입출력 예

- ![Image Alt 텍스트]({{site.url}}/img/algorithm/DFS_and_BFS.png);

풀이과정
-------

```cpp
var length;
var startpoint;
var dfslist=[];
var bfslist=[];

function solution(){
    startpoint=parseInt(input.shift().split(" ")[2]);

    input= input.map(element=>{
        return [parseInt(element.split(" ")[0]),parseInt(element.split(" ")[1])];
    })

    input.sort((a,b)=>{
        if(a[0]==b[0]){
            return a[1]-b[1];
        }else
            return a[0]-b[0];
    });

    
    dfs([],[startpoint]);
    bfs([],[startpoint],0);
    console.log(dfslist.join(" "));
    console.log(bfslist.join(" "));
}

function dfs(array,valuelist){
    valuelist = Array.from(new Set(valuelist));  
    let value = valuelist.shift()
    array.push(value);
    let tmp=[]
    input.some(element=>{
        if(element[0]==value&&!array.includes(element[1]))
            tmp.push(element[1]);
        
        if(element[1]==value&&!array.includes(element[0]))
            tmp.push(element[0]);
        
    });   
    tmp.sort((a,b)=>a-b);
    if(tmp.length==0)tmp =valuelist
    else tmp =tmp.concat(valuelist);
    
    if(tmp.length!=0) dfs(array,tmp);
    else dfslist= array;
    
}

function bfs(array,bfsq,index){
    let tmplist=[]
    bfsq.forEach(element=>{
        array.push(element);
    });
  
        input.forEach(ielement=>{
            if(ielement[0]==array[index]&&!array.includes(ielement[1])&&!tmplist.includes(ielement[1]))
                tmplist.push(ielement[1])
            if(ielement[1]==array[index]&&!array.includes(ielement[0])&&!tmplist.includes(ielement[0]))
                tmplist.push(ielement[0]);
        });
   
    index++;

    tmplist.sort((a,b)=>a-b);
    if(tmplist.length==0&& array.length==index) bfslist=array
    else bfs(array,tmplist,index);
}
```





출처
---
https://www.acmicpc.net/problem/1260