---
title: "[프로그래머스/JavaScript] 베스트앨범"
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
n개의 섬 사이에 다리를 건설하는 비용(costs)이 주어질 때, 최소의 비용으로 모든 섬이 서로 통행 가능하도록 만들 때 필요한 최소 비용을 return 하도록 solution을 완성하세요.

다리를 여러 번 건너더라도, 도달할 수만 있으면 통행 가능하다고 봅니다. 예를 들어 A 섬과 B 섬 사이에 다리가 있고, B 섬과 C 섬 사이에 다리가 있으면 A 섬과 C 섬은 서로 통행 가능합니다.

제한사항

섬의 개수 n은 1 이상 100 이하입니다.
costs의 길이는 ((n-1) * n) / 2이하입니다.
임의의 i에 대해, costs[i][0] 와 costs[i] [1]에는 다리가 연결되는 두 섬의 번호가 들어있고, costs[i] [2]에는 이 두 섬을 연결하는 다리를 건설할 때 드는 비용입니다.
같은 연결은 두 번 주어지지 않습니다. 또한 순서가 바뀌더라도 같은 연결로 봅니다. 즉 0과 1 사이를 연결하는 비용이 주어졌을 때, 1과 0의 비용이 주어지지 않습니다.
모든 섬 사이의 다리 건설 비용이 주어지지 않습니다. 이 경우, 두 섬 사이의 건설이 불가능한 것으로 봅니다.
연결할 수 없는 섬은 주어지지 않습니다.

입출력 예

n     |    costs                       |   return
--  |    --------                     |   ------
4 |	 [[0,1,1],[0,2,2],[1,2,5],[1,3,1],[2,3,8]]  | 4



풀이과정
-------

```cpp
function solution(n, costs) {
    var answer = 0;
    costs.sort((a,b)=>{
return a[2] - b[2]
});
let hashMap =new Map();

costs.forEach((element)=>{
    hashMap.set(element[0],element[0]);
    hashMap.set(element[1],element[1]);
})

costs.forEach((costElement)=>{
     if(hashMap.get(costElement[0])!=hashMap.get(costElement[1])){
        let big;
        let small;
        
        if(hashMap.get(costElement[0])<hashMap.get(costElement[1])){
            big= costElement[1];
            small=costElement[0];
            
        }else{
            big=costElement[0];
            small= costElement[1];
        }
        let checkvalue= hashMap.get(big);
        hashMap.forEach((value,key)=>{
            checkvalue ==value? hashMap.set(key,hashMap.get(small)):""
           });
           hashMap.set(big,hashMap.get(small))

        answer+=costElement[2]
    }
});
    return answer;
}
```



1. genres값들을 기준으로 plays값들을 HashTable 형식으로 정리해준다. HashTable value값은 배열 형식으로 저장시킨다.

2. HashTable 내부 value배열의 값들은 비교할수 있게끔 내림차순으로 정렬시켜주며 동일한 값들은 순번을 기준으로 오름차순 정렬을 한다.

3. 정렬되어진 HashTable을 value배열 값들의 합계 순으로 재정렬 해준다.

4. HashTable을 루프문으로 돌려 인덱스 값들을 결과값에 할당시켜준다.


처음에 막무가내로 풀었다가 코드가 너무 길어져서 순서대로 정리하며 다시 풀었나갔다 ㅠ.
초반에 장르 2개 노래 2개라는 것에 꽂혀 장르가 속한 곡이 하나일때를 생각을 못하고 풀어버려
계속 해맸다....이것만 아니였으면 금방 풀었을건데 문제 풀때 꼼꼼히 읽도록 습관화 해야될 것 같다.
문제를 풀때 장르에 속한 곡이 하나라는 것과 문제 설명대로 천천히 신경쓰면서 풀면 잘 풀수 있는 문제인것 같다.

참고
---

출처
---
https://programmers.co.kr/learn/courses/30/lessons/42579?language=javascript
