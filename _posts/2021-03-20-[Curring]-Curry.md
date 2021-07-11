---
title: "[JavaScript] Curry"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:

---

currying이란
===

Currying는 여러개의 인자를 가진 함수를 호출할 경우 적은 수의 파라미터를 인자로 받으면 누락된 파라미터를 인자로 받는 것을 뜻한다.
Javascript에서만 국한된 기술은 아니며 함수형프로그래밍에서 주로 쓰인다.

예를들어 f(a,b,c) 처럼 단일 호출로 처리하는 함수를 f(a)(b)(c)와 같이 각각의 인수가 호출 가능한 프로세스로 호출된 후 변환되는 것이다.


첫번쨰 예시
---

```js
    let poo =function(one){
        return funcion(two){
            console.log(`${one},${two}`)
        }
    }
    poo("one")("two");
```