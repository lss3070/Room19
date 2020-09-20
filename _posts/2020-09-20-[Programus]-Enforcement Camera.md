---
title: "[프로그래머스/JavaScript] 단속카메라"
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
고속도로를 이동하는 모든 차량이 고속도로를 이용하면서 단속용 카메라를 한 번은 만나도록 카메라를 설치하려고 합니다.

고속도로를 이동하는 차량의 경로 routes가 매개변수로 주어질 때, 모든 차량이 한 번은 단속용 카메라를 만나도록 하려면 최소 몇 대의 카메라를 설치해야 하는지를 return 하도록 solution 함수를 완성하세요.

제한사항

·차량의 대수는 1대 이상 10,000대 이하입니다.
·routes에는 차량의 이동 경로가 포함되어 있으며 routes[i][0]에는 i번째 차량이 고속도로에 진입한 지점, routes[i][1]에는 i번째 차량이 고속도로에서 나간 지점이 적혀 있습니다.
·차량의 진입/진출 지점에 카메라가 설치되어 있어도 카메라를 만난것으로 간주합니다.
·차량의 진입 지점, 진출 지점은 -30,000 이상 30,000 이하입니다.

입출력 예

routes                                          |   return
--------                                        |   ------
[[-20,15], [-14,-5], [-18,-13], [-5,-3]]	    |   2



풀이과정
-------

```cpp
function solution(routes) {
    let answer =1;
    routes.sort((a,b)=>{
        return a[0]-b[0];
    })
let standard= routes[0][1];

for(let i =0; i<routes.length-1;i++){
    if (standard > routes[i][1]) standard = routes[i][1];
     if (standard < routes[i + 1][0]) {
          answer++; 
          standard = routes[i + 1][1]; 
        }
}
    return answer
}
```

1. 진입지점을 기준으로 routes 배열을 오름차순 정렬해준다.

2. 단속카메라의 범위인 standard변수는 첫번째로 출발한 차량의 고속도로 나간 지점으로 지정해준다

3. 차량의 베열인 routes를 돌며 단속카메라의 범위와 현재 차량의 나간지점을 비교하여 더 작은 나간지점의 값들을
standard변수에 넣어준다.

4. 다음 차량의 진입지점이 단속카메라의 범위보다 클 경우 단속카메라 갯수를 증가시켜주고 단속카메라의 범위를 다음 자동차의 나간지점으로 지정해준다.






출처
---
https://programmers.co.kr/learn/courses/30/lessons/42884?language=javascript
