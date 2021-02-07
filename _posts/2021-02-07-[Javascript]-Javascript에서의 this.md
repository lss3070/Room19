---
title: "[React]Javascript this와 this 바인딩 우선순위"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    this
---

Javascript를 사용하다보면 this를 많이들 사용해본 경험이 있을거다.
this란
Javascript에서의 this는 상황별로 지칭하는 대상도 다르고 우선순위도 다르다
차근차근 예시를 들어가며 알아보자.

this 혼자 쓰인경우
---
아무것도 없는 about:blank에서 this를 출력해 봅시다.

 ![GlobalScope]({{site.url}}/img/javascript/this/GlobalScope_this.png)

위 그림과 같이 this는 스코프범위내에서의 객체를 뜻하며 아무것도 선언되지 않는 about:blank창에서는 객체가 window이기 때문에 this는 window객체를 출력합니다....수정!
그렇다면 다른 객체안에서의 this는 어떤식으로 표현이 될까요??


일반적인 바인딩
===

```cpp
var temp1 =10;
console.log(this.temp1);

function callThis() {
  this.temp2 = 10;
  console.log(this); // window 객체
}

console.log(this.temp2); // undefined

callThis();

console.log(this.temp);// 10
```
위 예제를 살펴봅시다
여기서 알수 있듯 전역으로 선언한 변수들은 전역객체(window)에 저장되는것을 알수 있습니다.
또한 함수내에서도 마찬가지입니다. callThis함수 또한 어떠한 객체안에서도 담겨있지 않기 때문에
상위 객체인 window객체 안에서만 움직이는것을 알 수 있습니다.

이제 window객체가 아닌 다른 일반 객체 안에서의 예시들을 살펴보죠.


암시적(implicit) 바인딩..
---
```cpp
var name ="kimchi"
function temp() {
  console.log(this.name);
}

const person = {
  name: "kim",
  showInfo1:temp,
  showInfo2() {
    console.log(this.name);
  },
  children:{
      name:"park",
      showInfo1(){
          console.log(this.name);
      }
  }
}
person.showInfo1();
person.showInfo2();
person.children.showInfo1();

````
person.showinfo1()메서드를 호출하였을 때 name이 window객체에서 kimchi를 가르키지 않고
person객체안에서의 kim을 가르키는걸 볼 수 있습니다. 
그렇습니다 객체 안에서의 this는 해당 객체를 가르키고 따라서 temp의 this객체는 person 객체안에서 사용되었기 때문에 person객체 안에서의 name인 kim을 가르키게 됩니다.

그리고 밑에 children객체 안에서의 name 호출 구문을 보시면 person객체의 name이 아닌
children객체의 name인 park를 호출하는것을 알 수 있습니다. 이를 통해 this는 무조건
객체안에서의 ~것을 알 수 있습니다.
...



명시적(explicit) 바인딩.
--- 
명시적 바인딩의 종류는 3가지이다.call,apply,bind를 이용하여 인자를 this
apply,call,bind메서드를 이용하여 인자를 this로 만들어주는 기능을 뜻합니다.
먼저 간단한 예제를 살펴보죠.

call
---

```cpp
var a=1;
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2
};
foo.call(obj); // 2
```
foo함수를 실행했을때 call메서드를 이용해서 실행을해주면 call의 인자인 obj객체가 this로 지정이 되기 때문에 a값은 전역객체로 선언된 1이 아니고 obj객체안의 2가 출력이 됩니다.


```cpp
function foo(temp,dummy){
    console.log(`${this},${temp},${dummy}`);
}
foo.call('console','log','!'); // console,log,!

```
첫번쨰 인자값은 무조건 this객체를 가리키며 두번쨰 인자값부터는 call메서드를 호출한 함수의 인자값을 순서대로 넣는다고 생각하면 된다.

apply
---
call과 비슷하지만 인자값이 배열로 들어간다는것에 차이가 있다.
```cpp

function foo(temp,dummy){
    console.log(`${this},${temp},${dummy}`);
}
foo.apply('console',['log',"!"]); // console,log,!
```

bind
---
bind메서드는 call,apply와 달리 새롭게 바인딩한 함수를 만들어 줍니다.

```cpp
let foo = {
    name: 'kim'
};

let poo = {
    name: 'lee',
    show: function() {
        console.log(this.name);
    }
};
poo.show(); // lee
let koo = poo.show.bind(foo);
koo(); // kim
```

생성자에서의 this
===

```cpp
var name="park"
function foo(name) {
  this.name = name;
}
var kim = new foo('kim');

var park = foo('lee');
 
console.log(kim.name); //kim
console.log(window.name); //lee
console.log(park.name); //error!


````
생성자 함수가 생성하는 객체로 this는 바인딩 됩니다.
생성자가 아닌 함수에서 호출하는 경우에는 this는 당연히 전역객체인 window에 바인딩이 됩니다.
다른 예제를 살펴보죠


```cpp
function foo(a) {
  this.a = a;
  this.qwer = 20;
}

var bar1 = new foo(2);
console.log(bar1.a); // 2
console.log(bar1.qwer); // 20

// 1. 새 객체가 만들어짐
var obj = {};
// 2. 새로 생성된 객체의 Prototype 체인이 함수의 프로토타입과 연결됨
Object.setPrototypeOf(obj, foo.prototype); // 프로토타입을 연결합니다. 이 글에서는 무시해도 상관없습니다.
// 3. 1에서 생성된 객체를 context 객체로 사용(명시적으로)하여 함수가 실행됨
foo.call(obj, 2);
// 4. 이 함수가 객체를 반환하지 않는 한 1에서 생성된 객체가 반환됨
var bar2 = obj; // 여기서 foo는 반환(return)이 없으므로 인스턴스가 생성(된 것처럼 동작)

console.log(bar2.a); // 2
console.log(bar2.qwer); // 20
```
이걸 보면 마치 js안에서도 class가 있는것처럼 보입니다.



이벤트리스너의서의 this
===
```cpp
var body = document.querySelector('body')
body.addEventListener('click', function () {
  console.log(this); //<body></body>
});
```
리스너 안에서의 this는 간단합니다.
이벤트 리스너에서의 this는 이벤트를 받는 HTML 요소를 가리킵니다.


binding의 우선순위
===
자 여러가지 형태의 this의 바인딩 방법을 알아보았는데요
이 바인딩들도 우선순위가 있습니다.
new 바인딩 - 명시적 바인딩 - 암시적 바인딩 - 기본 바인딩 이 순서대로 우선순위가 정해지며
아래 예제들을 살펴보며 어떻게 우선순위가 적용되는지 살펴보죠.

기본바인딩은 암시적 바인딩의 일부기 떄문에 패스하고
먼저 암시적 바인딩과 명시적 바인딩을 비교해보죠

명시적바인딩과 암시적 바인딩의 우선순위 비교
---

```cpp
function foo() {
  console.log(this.name)
}

var obj = {
  name: "kim",
  foo: foo,
}

obj.foo() // kim
obj.foo.call({ name: "lee" }) // lee
```
obj.foo()는 암시적 바인딩이 되어 obj객체가 바인딩이 되며 foo함수는 obj객체안의 name을 출력한다.
call메서드를 통해 alice 객체를 명시적으로 바인딩하면 obj를 통해 foo함수를 실행했더라도 명시적으로 바인딩 함수로 인해 this.name은 alice로 바인딩이 된다.
따라서 우리는 명시적 바인딩이 우선순위가 더 높은것을 알 수 있습니다.

new바인딩과 명시적 바인딩비교
---

```cpp
function foo(name) {
  this.name = name
}

var obj1 = {}

var poo = foo.bind(obj1)
poo("chris")
console.log(obj1.name) // chris

var obj2 = new helloFn("alice")
console.log(obj1.name) // chris
console.log(obj2.name) // alice
```




화살표함수 this
===

arrow함수의 this는 조금 특별합니다.
일반 함수와는 다르게 arrow함수는 함수안에서의 this를 가지고 있지 않습니다.대신 화살표 함수에서 this는 자신을 감싼 정적 범위lexical context가지며
전역 코드에서는 전역 객체를 가리킵니다.

```cpp
function foo() {
  setTimeout(function callback() {
    console.log(this.name)
  })
}

var obj = {
  name: "kim",
   foo: foo,
}

var name = "park"

foo() // 'park'
obj.foo() // 'park'
foo.call({ name: "lee" }) // 'park!'

```
위에서의 callback함수는 실행시점에서 컨텍스트 범위가 지정됩니다. 그리고 
callback함수는 setTimeout메서드가 실행시키는 것이고 
windos의 내장 함수인 setTimeout메서드는 callback형태로 실행시킬것이므로
this는 전역 객체로 바인딩이 된다.

```cpp
function foo() {
    setTimeout(() => {
        console.log(this.name)
    })
}
var obj = {
    name: "kim",
    foo: foo
}

var name = "park"

foo() // 'park'
obj.foo() // 'kim'
foo.call({name: "lee"}) // 'lee!'
```
arrow함수의 this는 상위 스코프 범위를 사용하며 foo를 호출했을 경우에는 상위 객체가 window인것을 확인 할 수 있으며 obj.foo는 암시적으로 바인딩된 obj객체 그리고 foo.call은 명시적으로 사용된 것을 알 수 있습니다.








this는 최대한 명시적으로만 사용하는게 좋습니다 ㅠㅜ


참고
---
https://yuddomack.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-this%EC%9D%98-4%EA%B0%80%EC%A7%80-%EB%8F%99%EC%9E%91-%EB%B0%A9%EC%8B%9D


https://jeonghwan-kim.github.io/2017/10/22/js-context-binding.html#%EC%95%94%EC%8B%9C%EC%A0%81-%EB%B0%94%EC%9D%B8%EB%94%A9%EA%B3%BC-%EB%AA%85%EC%8B%9C%EC%A0%81-%EB%B0%94%EC%9D%B8%EB%94%A9%EC%9D%98-%EC%9A%B0%EC%84%A0%EC%88%9C%EC%9C%84