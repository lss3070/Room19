---
title: "[프로그래머스/JavaScript] 섬연결하기"
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
        let hKey;
        let hValue;
        
        if(hashMap.get(costElement[0])<hashMap.get(costElement[1])){
            hKey= costElement[1];
            hValue=costElement[0];
            
        }else{
            hKey=costElement[0];
            hValue= costElement[1];
        }
        let checkvalue= hashMap.get(hKey);
        hashMap.forEach((value,key)=>{
            checkvalue ==value? hashMap.set(key,hashMap.get(hValue)):""
           });
         
           hashMap.set(hKey,hashMap.get(hValue))

        answer+=costElement[2]
    }
});
    return answer;
}
```

크루스칼 알고리즘을 이용하여 푸는 문제이다. 탐욕적인 방법(Greedy Method)을 이용하여 네트워크의 모든 정점을 최소 비용으로
연결하는 최적해답을 찾는 방법이다.


1. costs의 배열을 두섬을 연결하는 값을 기준으로 오름차순 정렬을 시켜준다.

2. 노드 값을 기준으로 HashTable을 만들어준다. (각 노드가 HashTable의 key값이며, 해당 노드의 연결된 최상의 노드의 값이
value값이다.)

3. costs배열의 루프문을 돌려 각 노드의 최상위 노드의 값이 같은지 확인하여준다.(각 노드의 최상위 값이 같을 경우 사이클이 형성되어 넣을 수 없음)

4. 최상위 노드의 값이 같이 않는 경우 costElement의 두개값 중에 큰값을 hKey, 작은 값을 hValue로 지정한다
 
5. HashTable을 돌려 4번항목에서 지정한 hKey값이 기존 최상위 노드의 값보다 작은 경우 hValue를 최상위 노드의 값으로 지정해준다.

6. HashTable에 해당 hKey값과 hValue값을 넣어주고 answer 변수에 비용을 더해준다.


참고
---
https://ko.wikipedia.org/wiki/%ED%81%AC%EB%9F%AC%EC%8A%A4%EC%BB%AC_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98

출처
---
https://programmers.co.kr/learn/courses/30/lessons/42579?language=javascript
