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
function solution(jobs){
    let total= jobs.length;
    let answer=0;
    let tick=0;
    let queue= [];
    while(true){
        let index=0;
        while(true){
            if(index>=jobs.length)break;
            if(jobs[index][0]<=tick){
                queue.push(jobs.splice(index,1)[0])
            }else{
                index++;
            }
        }
        if(queue.length>0){
            let minIndex=0;
            for(let i=1;i<queu.length;i++){
                if(queue[minIndex][1]>queue[i][1]){
                    minIndex=1;
                }
            }
            tick+=queue[minIndex][1];
            answer += tick- queue[minIndex][0];
            queue.splice(minIndex,1);
        }
        else if(queue.length===0){
            if(jobs.length===0){
                break;
            }else{
                tick
++            }
        }
    }
    return parseInt(answer/total)
}
```






출처
---
https://programmers.co.kr/learn/courses/30/lessons/42627?language=javascript
