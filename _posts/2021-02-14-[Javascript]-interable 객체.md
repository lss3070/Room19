---
title: "[Javascript] Javascript에서의 iterable과 iterator
subtitle: "기본기"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    Interable
---

먼저 Interator란 뭘까?
위키 백과에 따르면 '반복자(iterator)는 객체 지향적 프로그래밍에서 배열이나 그와 유사한 자료 구조의 내부의 요소를 순회(traversing)하는 객체이다.' 라고 명시가 되어있다.
javascript에서도 이러한 Interator 기능이 정의되어 있는데 es6에 나온 iterable protocol이다
iterable protocol은 이것은 두가지 형태로 존재하는데, 이터러블(iterable)과 이터레이터(iterator)이다.이 프로토콜을 준수한 객체만이 `for..of`문으로 순회를 할 수 있으며 `Spread문법`의 피연산자가 될 수 있다.

Iterable protocol을 단순하게 말하면 데이터 컬렉션을 반복할 수 있게 하는 객체정도로 보면 됩니다.

`이터러블(iterable)`과 `이터레이터(iterator)` 살펴보기에 앞서 `for..of문`과 `Spread문법`에 대해 간단하게 알아보자.


for..of문
---
for..of문은 es6에 도입되었으며 각 요소들을 순회하며 특정 작업을 반복수행하는 loop문이다.
타입에 따라 사용할 수있는것과 없는게 구분이 되며 사용가능한 타입들을 살펴보자.

```js
// 배열
for (const val of ['a', 'b', 'c']) {
  console.log(val);// "a","b","c"
}
// 문자열
for (const val of 'abc') {
  console.log(val);//"a","b","c"
}
// Map
for (const [key, value] of new Map([['a', '1'], ['b', '2'], ['c', '3']])) {
  console.log(`key : ${key} value : ${value}`); // "key : a value : 1", "key : b value : 2", ...
}
// Set
for (const val of new Set([1, 2, 3])) {
  console.log(val);// 1, 2, 3
}
```
위 나와있는 타입이외에도 여러가지 타입이 지원이 되지만 이번 포스팅은 Interator에 대한 글이니 추후 javascript loop에 대해 포스팅하도록 하겠다.

Spread문법
---
Spread문법은 es6에 도입되었으며 문법의 기능으로는 여러개의 배열을 하나로 합치는 concat함수를 대체하거나 함수의 인자로 배열 내부에 있는 값들로 사용을 할때 쓰이는 apply 함수를 대체할 수 있습니다.
```js
function sum(x,y,z){
    return x+y+z;
}
let numbers=[1,2,3];

console.log(sum(...numbers)); // 6
console.log(sum.apply(null, numbers));// 6

let extensions=[...numbers,4,5,6]
console.log(extensions);//[1,2,3,4,5,6]
```



이터러블(iterable)
---
iterable protocol프로토콜에 기본적으로 이용되는 문법에 대해 알아보았으니 이제 `이터러블(iterable)`에 대해 알아보죠
`이터러블(iterable)`은 객체의 멤버변수를 반복할 수 있는 객체입니다. `이터러블(iterable)`이 사용되기 위해선 
`이터레이터(interator)`메서드가 구현되어 있어야하며 객체 프로퍼티에는 [Symbol.interator]를 추가하여야 합니다.

 `이터러블(iterable)`객체에서 배열은 대표적으로 사용되므로 배열을 통해 `이터러블(iterable)`이 뭔지에 대해 알아보자.

```js
const array = [1, 2, 3];
const obj = {a:1,b:2,c:3}

// 배열은 Symbol.iterator 메소드를 소유한다.
// 따라서 배열은 이터러블 프로토콜을 준수한 이터러블이다.
console.log(Symbol.iterator in array); // true
console.log(Symbol.iterator in obj); // true

// 이터러블 프로토콜을 준수한 배열은 for...of 문에서 순회 가능하다.
for (const item of array) {
  console.log(item);
}

for (const item of objarray) {
  console.log(item);
}
```

두개의 반복문을 비교해보자 
배열인 array 객체는 iteration 프로토콜 조건을 준수하기 때문에 `이터러블(iterable)`객체에서만 쓸 수 있는 for..of문을 
사용하지만 일반 객체인 obj는 iteration 프로토콜 조건을 준수하지 않았기 때문에 for..of문과 같은
반복문이 순회하지 않으며 spread문법의 대상으로도 사용할 수 없다.

여기서 말하는 iteration프로토콜의 조건은 다음과 같다
1. 객체는 [Symol.iterator]를 구현하고 있어야하며,`이터레이터(iterator)`를 리턴해야한다.
2. 그 `이터레이터(iterator)`객체는 next() 메서드가 구현되어야 하고,{value,done} 프로퍼티를 갖는 객체를 반환해야한다.


이런 일반객체도 위 조건에 맞춰 `이터러블(iterable)` 객체로 바꿀 수 있는데 그걸 `커스텀 이터러블(Custom iterable)`이라고 부른다.
위 조건은 만족하는 커스텀 이터러블을 만들기에 앞에서 이터레이터(iterator)에 대해 알아보고 넘어가자.


이터레이터(iterator)
---
`이터레이터(iterator)`는 특정 조건을 만족하는 next()메서드를 가지며 next()메서드를 호출하면 
`이터러블(iterable)`을 순회하며 {value,done} 프로퍼티를 갖는 객체를 반환하는 것이다.
즉 객체[Symbol.iterator]의 결과값은 `이터레이터(iterator)`라고 부르며 
즉 `이터레이터(iterator)`는 이어지는 반복 과정을 처리하는것을 뜻합니다.

```js
const array = [1, 2, 3];

// Symbol.iterator 메서드는 이터레이터를 반환한다.
const iterator = array[Symbol.iterator]();

// 이터레이터는 next 메서드를 가진다.
console.log('next' in iterator); // true

// 이터레이터가  순회를 하면서 next메서드를 호출할때마다
// {value,done} 프로퍼티 객체를 반환한다.
console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```


커스텀 이터러블(iterable)
---
```js
let obj = {
  from: 1,
  to: 5,

  [Symbol.iterator]() {
    this.current = this.from;
    return this;
  },

  next() {
    if (this.current <= this.to) {
      return { done: false, value: this.current++ };
    } else {
      return { done: true };
    }
  }
};

for (let num of range) {
  console.log(num); // 1, 2, 3, 4, 5
}
```
[Symbol.iterator]메소드는 next메소드를 갖는 `이터레이터(interation)`를 반환하여야 합니다.
그 후 `이터레이터(interation)`에서 next메소드는 {done,value} 프로퍼티를 가지는 객체를 반환합니다.
done의 treu값은 반복이 종료 되었다는 것을 의미하며 for..of문은 done 프로퍼티가 true가 나올때 까지 반복하는 것입니다.


<!-- 마치며...
---
막연하게 for..of문과 spread문을 쓰며 안에 구조에 대해선 두루뭉실하게 알고 있었는데
오늘 이렇게 이터러블과 이터레이터에 대해서 정리하는 글을 쓰며 정확하게 집고 알아가게 되었던 것 같다. -->




참고
---
<https://poiemaweb.com/es6-iteration-for-of#3-%EC%BB%A4%EC%8A%A4%ED%85%80-%EC%9D%B4%ED%84%B0%EB%9F%AC%EB%B8%94>







