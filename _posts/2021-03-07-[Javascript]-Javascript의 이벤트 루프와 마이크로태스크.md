---
title: "[JavaScript] 이벤트 루프"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:

---


자바스크립트는 싱글 스레드 기반의 언어입니다.싱글 스레드는 한번에 하나의 호출스택을 사용하며 이러한 방식은 한번에 하나의 동작만을 사용할 수 있음을 의미합니다.그럼 자바스크립트에서는 어떻게 동시성을 지원할까요??
바로 이벤트루프라는 것을 이용하여 비동기 방식으로 동시성을 지원합니다.


이벤트 루프
---
하지만 자바스크립트 엔진 자체에서 지원해주는 것이 아니라 엔진을 구성하는 환경 인 브라우저나 Node.js에서 지원이 된다.
간단하게 브라우저에서 어떤식으로 동시성이 지원되는지 살펴보자

 ![eventloop]({{site.url}}/img/javascript/eventloop/step1.png)

heap javascript에서의 변수,함수 저장 호출등의 작업이 발생하는 공간
call stack 함수 호출시 실행 컨텍스트가 만들어지며 이렇게 만들어진 것들이 stack을 구성한다
webapi 웹 브라우저에 구현 된 API로 DOM 이벤트와 XMLHttp,Timer이벤트 등이 있다.
event loop
콜스택이 비어 있으면 태스크 큐에 있는 콜백 함수를 처리한다.
task queue



단일 호출 스택
---
자바스크립에서 함수가 실행되는 방식은 Run to Completion이다.이것은 함수가 동작하는 도중 다른 작업이 중간에 끼어들지 못하고 작업이 완료되고 나야 다른작업이 실행될 수 있는 것을 의미한다.



이벤트 루프의 종류
---
이벤트 루프의 종류로는 매크로 태스크 큐와 마이크로 태스크 큐가 있다
매크로 태스크 큐는 settimeout이나 setInterval와 같은 메서드를 정의하는 이벤트 루프이고
마이크로 태스크 큐는 Promise,MutationObserver 와 같은 메서드를 정의한다
이 두 이벤트 루프는 서로의 우선순위가 다르며 매크로 태스크 큐가 먼저 앞에 와 있어도 마이크로 태스크 큐가 우선순위가 높다.
아래 예제를 살펴보자.


```js
setTimeout(()=> console.log('매크로 태스크 큐!'),0);
Promise.resolve().then(() => console.log('마이크로 태스크 큐!'));
```
위 구문을 실행해보면 마이크로 태스크 큐가 먼저 실행된 것을 알 수 있다.
```
마이크로 태크크 큐!
매크로 태스크 큐!
```

![eventloop]({{site.url}}/img/javascript/eventloop/step2.gif)


이벤트 루프의 동작방식
---



![eventloop]({{site.url}}/img/javascript/eventloop/step4.png)

![eventloop]({{site.url}}/img/javascript/eventloop/step3.gif)


태스크 큐
마이크로 태스크큐 v8
매크로 태스크큐


엔진은 특정태스크를 처리하는 ㄴ동안 렌더링은 절대 일어나지않음..





참고
---
<https://ko.javascript.info/event-loop>