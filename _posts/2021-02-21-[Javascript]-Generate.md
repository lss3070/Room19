---
title: "[Javascript] Javascript에서의 generate"
subtitle: "기본기"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    generate
---


이전 포스팅에서 봤던 이너레이터가 값을 읽어오기 위한 인터페이스라면 제너레이터는 값을 쓰기 위한 인터페이스이다.
제너레이터는 또한 interator protocol 규약을 준수해서 반복문인 for..of문과 Spread문법에서 사용할 수 있다.


제너레이터 함수
---
제너레이터는 일반적인 함수와는 다르게 * 함수뒤나 함수명 앞에 * 를 붙입니다.

```js
function* test() {
  return 1;
}
const gen = test();
console.log(gen);//test {<suspended>}
```
test 함수를 호출했는데 왜 값이 3 반환이 안될까??

제너레이터 함수는 일반적인 함수와 다르게 함수를 호출하게 되면 함수가 실행이 되지않고 실행을 처리하는 제너레이터 객체가 반환이 됩니다.

 ![프로미스도식화]({{site.url}}/img/javascript/generate/step1.png)


이 안에 객체를 사용하기 위해선 next란 메서드와 yield란 객체가 필요합니다.


yield,next
---
yield 와 next는 제너레이터에서의 중요한 메서드입니다.
yield는 제너레이터 함수가 실행될때 debuging을 할때의 break point처럼 중간에 함수를 정지를 시키며 yield 뒤에 오는 표현식이 반환됩니다.
제너레이터 함수가 실행되기 위해선 next라는 함수로 실행을 시켜줘야된다.

```js
function* test() {
    yield 1;
    yield 2;

  return 3;
}
const gen = test();
console.log(gen.next());//{value: 1, done: false}
console.log(gen.next());//{value: 2, done: false}
console.log(gen.next());//{value: 3, done: true}
console.log(gen.next());//{value: undefined, done: true}
```
next메서드를 호출하게 되면 yield<value>문을 만날때까지 실행이 계속됩니다.
yield<value> 문을 만나게 되면 함수의 실행이 멈추고 value 값이 반환이 됩니다.
 
위 예제에서도 gen함수의 next메서드를 호출하였을 때 순차적으로 yield에 멈추게 되고 yield의 value값이 반환이 됩니다.
마지막으로는 return문에 다다르고 제너레이터가 종료가 됩니다.
제너레이터가 종료된 후 next메서드를 호출하여도 value 값은 undefinded입니다.

제너레이터의 이터러블 구현
---

```js
function* test() {
    yield 1;
    yield 2;

  return 3;
}

let gen = test();

for(let value of gen) {
  console.log(value); // 1, 2가 출력됨
}

```

제너레이터도 이터러블처럼 fot..of 반복문을 사용해서 값을 얻을 수 있습니다.
하지만 정작 return 값은 반환이 안되죠 그 이유는 done:ture이면 마지막 value값은 무시가 되기 때문입니다. 그러므로 반복문에서 제너레이터를 사용할 경우 모든 값은 yield로 반환해주어야 합니다.
제너레이터 함수는 이터러블을 구현 할 수 있다.이터레이션 프로토콜 규약에 맞게 작동하는 반복문을 만들어보자.

```js
const test = (function* () {
  let cur = 0;

  while (true) {
    cur+=1
    yield cur;
  }
}());

// infiniteFibonacci는 무한 이터러블이다.
for (const num of test) {
  if (num > 10) break;
  console.log(num);
}
```

또한 Spread문법을 이용하여 작성할수 있다.
```js
const test = function* (max) {
  let cur = 0;

  while (true) {
    cur+=1
    if (cur >= max) return; // 제너레이터 함수 종료
    yield cur;
  }
};

console.log([...test(10)]);

```

예외처리
---
제너레이터의 인자값은 yield의 value 값이 지정됩니다.
그런데 인자 값으로도 errer 값을 던질 수 있습니다.
에러를 yield 안으로 전달하려면 generator.throw(err)를 호출해야 합니다. generator.throw(err)를 호출하게 되면 err는 yield가 있는 줄로 던져집니다.

```js
function* test() {
  let result = yield 1; // Error in this line
}

let gen = test();

let value = gen.next();

  gen.throw(new Error("에러 발생!!"));

```

비동기처리
---
또한 제너레이터는 Promise처럼 비동기 처리를 동기 방식으로 구현할 수 있다.

```js

function getUser(genObj, username) {
  fetch(`https://api.github.com/users/${username}`)
    .then(res => res.json())
    .then(user => genObj.next(user.name));
}

const gen = (function* () {
  let user;
  user = yield getUser(g, 'kakao');
  console.log(user); // kakao

  user = yield getUser(g, 'naver');
  console.log(user); // Naver
}());

gen.next();
```

