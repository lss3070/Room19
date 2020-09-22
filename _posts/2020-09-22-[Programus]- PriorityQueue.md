---
title: "[프로그래머스/JavaScript] 이중우선순위큐"
subtitle: "알고리즘 풀이"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    -Algorithm
---


문제 설명
-------
이중 우선순위 큐는 다음 연산을 할 수 있는 자료구조를 말합니다.

입출력 예

명령어                                          |   수신 탑(높이)
--------                                        |   ------
I 숫자	    |  큐에 주어진 숫자를 삽입합니다.
D 1 | 큐에서 최댑값을 삭제합니다.
D -1 | 큐에서 최솟값을 샂게합니다.

이중 우선순위 큐가 할 연산 operations가 매개변수로 주어질 때, 모든 연산을 처리한 후 큐가 비어있으면 [0,0] 비어있지 않으면 [최댓값, 최솟값]을 return 하도록 solution 함수를 구현해주세요.

제한사항
ㆍoperations는 길이가 1 이상 1,000,000 이하인 문자열 배열입니다.
ㆍoperations의 원소는 큐가 수행할 연산을 나타냅니다.
원소는 “명령어 데이터” 형식으로 주어집니다.- 최댓값/최솟값을 삭제하는 연산에서 최댓값/최솟값이 둘 이상인 경우, 하나만 삭제합니다.
ㆍ빈 큐에 데이터를 삭제하라는 연산이 주어질 경우, 해당 연산은 무시합니다.


입출력 예

operations                  |    return
["I 16","D 1"]              |[0,0]
["I 7","I 5","I -5","D -1"] |[7,5]

풀이과정
-------

```cpp
function solution(operations) {
    var answer = [];
    
    operations.forEach((element)=>{
        let elementArray = element.split(" ");
        if(elementArray[0]=="I"){
            answer.push(elementArray[1]);
        }else{
                     if(elementArray[1]==1){
               let max= answer.reduce((pre,cur,i)=>{
                    return pre[0]<cur? [cur,i]:pre 
                },[0,0]);
                answer.splice(max[1],1);

            }else if(elementArray[1]==-1){
                let min= answer.reduce((pre,cur,i)=>{
                    return parseInt(pre[0])>parseInt(cur)? [cur,i]:pre 
                },[999,0]);
                answer.splice(min[1],1);
            }
        }
    });
       if(answer==0) return [0,0];
    else return [Math.max.apply(null,answer),Math.min.apply(null,answer)];

}
```







출처
---
https://programmers.co.kr/learn/courses/30/lessons/42628?language=javascript