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
스트리밍 사이트에서 장르 별로 가장 많이 재생된 노래를 두 개씩 모아 베스트 앨범을 출시하려 합니다. 노래는 고유 번호로 구분하며, 노래를 수록하는 기준은 다음과 같습니다.

1.속한 노래가 많이 재생된 장르를 먼저 수록합니다.
2.장르 내에서 많이 재생된 노래를 먼저 수록합니다.
3.장르 내에서 재생 횟수가 같은 노래 중에서는 고유 번호가 낮은 노래를 먼저 수록합니다.

노래의 장르를 나타내는 문자열 배열 genres와 노래별 재생 횟수를 나타내는 정수 배열 plays가 주어질 때, 베스트 앨범에 들어갈 노래의 고유 번호를 순서대로 return 하도록 solution 함수를 완성하세요.

제한사항
·genres[i]는 고유번호가 i인 노래의 장르입니다.
·plays[i]는 고유번호가 i인 노래가 재생된 횟수입니다.
·genres와 plays의 길이는 같으며, 이는 1 이상 10,000 이하입니다.
·장르 종류는 100개 미만입니다.
·장르에 속한 곡이 하나라면, 하나의 곡만 선택합니다.
·모든 장르는 재생된 횟수가 다릅니다.

입출력 예

genres                                          |    plays                        |   return
----------                                      |    --------                     |   ------
["classic", "pop", "classic", "classic", "pop"] |	 [500, 600, 150, 800, 2500]	  | [4, 1, 3, 0]



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
