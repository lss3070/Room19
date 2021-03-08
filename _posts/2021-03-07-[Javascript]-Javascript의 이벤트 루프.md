---
title: "[JavaScript] 이벤트 루프"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:

---

자바스크립트는 싱글 스레드 기반의 언어입니다.싱글 스레드는 한번에 하나의 호출스택을 사용하며 이러한 방식은 한번에 하나의 동작만을 사용할 수 있음을 의미합니다.
그럼 자바스크립트에서는 어떻게 동시성을 지원할까요??
바로 이벤트루프라는 것을 이용하여 비동기 방식으로 동시성을 지원합니다.


이벤트 루프
---
하지만 자바스크립트 엔진 자체에서 지원해주는 것이 아니라 엔진을 구성하는 환경 인 브라우저나 Node.js에서 지원이 됩니다.
간단하게 브라우저에서 어떤식으로 동시성이 지원되는지 살펴보기에 앞서 브라우저의 환경 구조에 대해 알아보죠.

 ![eventloop]({{site.url}}/img/javascript/eventloop/step1.png)

heap javascript에서의 변수,함수 저장 호출등의 작업이 발생하는 공간
call stack 함수 호출시 실행 컨텍스트가 만들어지며 이렇게 만들어진 것들이 stack을 구성한다
webapi 웹 브라우저에 구현 된 API로 DOM 이벤트와 XMLHttp,Timer이벤트 등이 있다.
event loop
콜스택이 비어 있으면 태스크 큐에 있는 콜백 함수를 처리한다.
task queue
이벤트 루프에서 첫 번째 태스크를 가져오는 것이 아니라, 태스크 큐에서 실행 가능한(runnable) 첫 번째 태스크를 가져오는 것이다. FIFO형식으로 제일 오래된 태스크를 가지고와서 처리한다.


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
자바스크립에서 함수가 실행되는 방식은 Run to Completion이다.이것은 함수가 동작하는 도중 다른 작업이 중간에 끼어들지 못하고 작업이 완료되고 나야 다른작업이 실행될 수 있는 것을 의미합니다.

![eventloop]({{site.url}}/img/javascript/eventloop/step4.png)


```js
function foo() {
    boo();
    console.log('foo');
}
function boo() {
    console.log('boo');
}

function zoo() {
    console.log('zoo');
}

setTimeout(zoo, 1);
foo();
```
1. setTimeout함수는 이벤트 요청 후 스택에서 바로 제거 됩니다.(매크로태스크큐는 스택이 비워진 상태에서 실행이 된다.)
2. 그다음 foo함수가 스택에 추가됩니다.
3. foo함수의 내부 기능들이 실행이 됩니다.(boo함수가 있기때문에 스택에서 제거 되지 않습니다.)
4. boo함수가 스택에 추가되고 boo함수 내부에 더 실행할 함수가 없으면 boo함수는 실행이 되조 스택에서 제거 됩니다.
5. foo함수의 기능들이 다 수행되면 foo함수도 스택에서 제거됩니다.
6. zoo함수가 스택에 추강됩니다.
7. zoo함수안에 다른 함수가 없는 경우 zoo함수도 제거 됩니다.

<!-- 이벤트 루프의 작업 방식은 간단합니다. 작업(태스크)가 들어오기 까지 기다렸다가
태스크가 들어오게 되면 처리하고 처리할 작업(태스크)가 없는 경우에는 움직이지 않습니다.  -->

![eventloop]({{site.url}}/img/javascript/eventloop/step3.gif)

<!-- 
태스크 큐
마이크로 태스크큐 v8
매크로 태스크큐


엔진은 특정태스크를 처리하는 ㄴ동안 렌더링은 절대 일어나지않음..

 -->

참고
---
<https://ko.javascript.info/event-loop>
<https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif>