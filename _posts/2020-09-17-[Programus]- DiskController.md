---
title: "[프로그래머스/JavaScript] 디스크 컨트롤러"
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
하드디스크는 한 번에 하나의 작업만 수행할 수 있습니다. 디스크 컨트롤러를 구현하는 방법은 여러 가지가 있습니다. 가장 일반적인 방법은 요청이 들어온 순서대로 처리하는 것입니다.

예를들어

- 0ms 시점에 3ms가 소요되는 A작업 요청
- 1ms 시점에 9ms가 소요되는 B작업 요청
- 2ms 시점에 6ms가 소요되는 C작업 요청

와 같은 요청이 들어왔습니다. 이를 그림으로 표현하면 아래와 같습니다.
- ![Image Alt 텍스트]({{site.url}}/img/algorithm/algorighm_diskcontroller1.png)


한 번에 하나의 요청만을 수행할 수 있기 때문에 각각의 작업을 요청받은 순서대로 처리하면 다음과 같이 처리 됩니다.
- ![Image Alt 텍스트]({{site.url}}/img/algorithm/algorighm_diskcontroller2.png);


- A: 3ms 시점에 작업 완료 (요청에서 종료까지 : 3ms)
- B: 1ms부터 대기하다가, 3ms 시점에 작업을 시작해서 12ms 시점에 작업 완료(요청에서 종료까지 : 11ms)
- C: 2ms부터 대기하다가, 12ms 시점에 작업을 시작해서 18ms 시점에 작업 완료(요청에서 종료까지 : 16ms)



이 때 각 작업의 요청부터 종료까지 걸린 시간의 평균은 10ms(= (3 + 11 + 16) / 3)가 됩니다.

하지만 A → C → B 순서대로 처리하면
- ![Image Alt 텍스트]({{site.url}}/img/algorithm/algorighm_diskcontroller3.png);

- A: 3ms 시점에 작업 완료(요청에서 종료까지 : 3ms)
- C: 2ms부터 대기하다가, 3ms 시점에 작업을 시작해서 9ms 시점에 작업 완료(요청에서 종료까지 : 7ms)
- B: 1ms부터 대기하다가, 9ms 시점에 작업을 시작해서 18ms 시점에 작업 완료(요청에서 종료까지 : 17ms)


이렇게 A → C → B의 순서로 처리하면 각 작업의 요청부터 종료까지 걸린 시간의 평균은 9ms(= (3 + 7 + 17) / 3)가 됩니다.

각 작업에 대해 [작업이 요청되는 시점, 작업의 소요시간]을 담은 2차원 배열 jobs가 매개변수로 주어질 때, 작업의 요청부터 종료까지 걸린 시간의 평균을 가장 줄이는 방법으로 처리하면 평균이 얼마가 되는지 return 하도록 solution 함수를 작성해주세요. (단, 소수점 이하의 수는 버립니다)

풀이과정
-------

```cpp
function solution(genres, plays) {
    var answer = [];
    let hashtable = new Map();
    
      for(let i=0; i<genres.length;i++){
        let value =hashtable.get(genres[i])
        if(value!=null) hashtable.set(genres[i],[[plays[i],i], ...value])
        else hashtable.set(genres[i],[[plays[i],i]]);
    }
    
    hashtable.forEach((value,key)=>{
        value= value.sort((a,b)=>{
           if(a[0]>=b[0]) return a[0]==b[0]? a[1]-b[1]:-1
           else return 1;
        });
    });

    hashtable = new Map([...hashtable].sort((a,b)=>{
        let aMax= a[1].reduce((pre,cur)=>{return pre+cur[0]},0);
        let bMax= b[1].reduce((pre,cur)=>{return pre+cur[0]},0);
        return aMax>bMax? -1:1
    }))
    
    hashtable.forEach((value,key)=>{
        if(value.length>1) answer.push(value[0][1],value[1][1]);
        else if(value.length==1) answer.push(value[0][1]);
    })
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



출처
---
https://programmers.co.kr/learn/courses/30/lessons/42579?language=javascript
