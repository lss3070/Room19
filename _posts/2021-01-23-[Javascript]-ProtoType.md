---
title: "[Javascript] Prototype이란"
subtitle: "기본기"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    Prototype
---

자바스크립트의 프로토타입을 처음 접한건 상속을 위해 prototype만을 알아보고 급하게 구현했던 걸로 기억합니다.
그때 당시로는 그냥 prototype은 그냥 javascript객체 상속을 위해 쓰는 내부 함수구나 하고 넘어갔었고 또
작년 기술면접을 준비하며 정의에 대해서 공부를 하며 달달 외우기만 했던 것 같습니다.

이때까지 뭔가를 위해서 급하게 달달 외운듯한 것 같아 온전히 내것이 되지않은듯해 이번 포스팅과 추후 포스팅을 통해
스터디를 해보기로했다.
이 포스팅을 통해 프로토타입에 대해 정확하게 알고가는 계기가 되면 좋겠으며 다른 사람들에게도 도움이 되길 바랍니다.


본문에서 ECMAScript에서 사용되는 PrototypeLink인[[Prototype]] 프로퍼티는 본문에서 **__proto__**로 명시하겠습니다.

프로토타입 프로그래밍이란
---
객체의 원형인 프로토타입을 이용하여 새로운 객체를 만들어내는 프로그래밍 기법이다. 이렇게 만들어진 객체 역시 자기자신의 프로토타입을 갖는다. 이 새로운 객체의 원형을 이용하면 또 다른 새로운 객체를 만들어 낼수도 있으며 이런 구조로 객체를 확장하는 방식을 프로토타입 기반 프로그래밍이라고 한다.


 ![멍청짤]({{site.url}}/img/meme/멍청짤.png)

<center>(음... 이게 뭔소리래)</center>
<center>밑에서 하나하나 풀어서 알아보도록 하죠!</center>


__proto__와 prototype 프로퍼티
---
일반적으로 자바스크립트의 객체의 구성요소로는**__proto__**라고 하는 객체원형에대한 연결을 나타내는 프로토타입 링크와 함수를 나타내는 prototype이라는 프로토타입 객체가 존재합니다.
javascript에서 객체 생성시 생성된 객체는 프로토 타입의 링크가 생성된 함수의 프로토타입 객체를 참조하는 형태를 가지게 됩니다.
자바스크립트의 모든 객체는 자신을 생성한 생성자 함수의 prototype프로퍼티가 가리키는 프로토타입객체를 부모 객체로 설정하는
**__proto__**로 연결을 한다. **__proto__**는 또한 상속을 위해 사용이 된다.
이 규칙을 적용해서 다음 코드를 살펴보죠


```cpp
function Human(name){
    this.name=name
}
let cloneHuman= new Human("kim");
console.dir(Human);
console.dir(cloneHuman);

```
![__proto__&prototype 예제]({{site.url}}/img/javascript/prototype/prototype_step3.png)

여기서 Human 생성자 함수의 **prototype 프로퍼티**는 Human.prototype와 연결되어 있고
Human 생성자 함수로 생겨난 cloneHuman 객체 역시 Human.prototype와 **__proto__**로 연결되어있다.

결국 Human 생성자의 **prototype 프로퍼티**나 cloneHuman의 **__proto__**는 같은 프로토타입 객체를 가르키고 있게되는 것이며
자바스크립에서 객체를 생성하는건 생성자 함수의 역할이지만 생성된 객체의 실제 부모 역할을 하는건 생성자가 아닌
생성자 **prototype**과 연결된 객체의 **prototype객체**이다.

이것은 마치 객체지향언어의 상속 개념과 같아 부모 객체의 프로퍼티를 자신의 것처럼 쓸 수 있으며 
이러한 부모 객체를 **prototype객체**라고 부릅니다.

이를 바탕으로 자바스크립트에서 상속을 구현할 수 있습니다.


자바스크립트에서의 상속
---
자바스크크립에선 부모역할을 하는 프로토 타입의 객체를 기존 객체지향언어에서의 상속받은 자식클래스가 부모클래스의 메서드에 접근하는것 처럼
부모 객체에 접근이 가능합니다.
 이것을 가능케 하는것이 바로 프로토타입 체이닝이며 생성된 객체 형식에 따라 사용하는 방식이 조금 다릅니다.
하지만 '모든 객체는 자신을 생성한 생성자 함수의 **prototype 프로퍼티**를 가르키는 객체를 자신의 프로토타입 객체로 취급한다'
라는 프로토타입 기본원칙은 같기 때문에 큰차이는 없다.
객체 형식에 따라 리터럴 방식의 체이닝과 생성자 함수형식의 체이닝이 존재한다.먼저 리터럴 방식의 prototype 체이닝을 알아보자.

리터럴 형식으로 생성된 객체의 prototype 체이닝
---

```cpp
let human={
    name:"kim",
    age:30
}
human.toString();
console.log(human);
```
!!먼저 prototype 객체는 class와 마찬가지로 메서드를 가질수 있습니다. 

위의 예제는 리터럴 형식으로 human객체를 생성하고 human객체의 메서드를 출력한 것입니다.
여기서 human.prototype을 이용해서 toStirng 함수를 추가하지도 않았는데 왜 작동하는것일까요...
그 이유는 바로 human객체의 **__proto__**에 이미 정의가 되어있으며 human 객체의 부모 객체에서 상속을 받아
toString 메서드를 호출 하였기 때문이다.
더 자세히 알아보기 위해 아래 그림을 참고하자

 ![리터럴예제]({{site.url}}/img/javascript/prototype/prototype_step1.png)

위의 실행결과를 보면 알 수 있듯이 age,name 프로퍼티 이외에도 **__proto__**라는 프로퍼티가 있는 것을 알 수 있다.
여기서 **__proto__**에 정의 되어있는것은 Object prototype객체이며 human객체의 부모객체이다.
(Object는 자바스크립트에서 기본적으로 제공되는 함수입니다!)
 **__proto__**를 통해 부모 객체와 연결되어 있으며 Object prototype에는 toString()이라는 메서드가 명시되어 있기 때문에 Object prototype 객체를 상속받은 human객체는 human.toString()메서드를 호출해도 에러가 나지 않는것이다.

위의 관게를 도식화 하면 이렇다.
 ![리터럴예제]({{site.url}}/img/javascript/prototype/prototype_step2.png)

다음으로는 생성자 함수 prototype체이닝에 알아보자


생성자 함수로 생성된 객체의 prototype 체이닝
---

```cpp
function Human(name,age){
    this.name=name;
    this.age=age;
}
Human.prototype.nameRepetition=function(){
    return this.name;
}

Object.prototype.testMethod=function(){//      ···①
    return "test..."
}

let cloneHuman = new Human("kim",30);  
console.dir(cloneHuman.hasOwnProperty("name")); //result :  true

console.dir(cloneHuman.nameRepetition()) //result : kim
console.dir(cloneHuman.testMethod()); //result : test...   ···②

Human.prototype.name="android";//     ···③
console.dir(Human.prototype.nameRepetition());//result : android     ···④
```

위 예제를 보시면 cloneHuman 객체의 생성자는 Human함수이며 따라서 위에서 정의한 바와 같이 생성자 함수 객체의 
prototype(Human.prototype)프로퍼티가 가리키는 객체가 된다.

cloneHuman.hasOwnProperty()메서드를 호출할때 cloneHuman에 hasOwnProperty란 메서드를 정의하지 않았기 때문에
프로토타입 체이닝으로 cloneHuman에과 연결된 부모객체중에서 hasOwnProperty메서드를 찾는다. 부모인 Human.prototype객체에는
없지만 Human객체와 또 연결된 Object.prototype객체에서 찾는다. Object.prototype에는 hasOwnProperty가 정의가 되어있기 때문에
정상적으로 cloneHuman.hasOwnProperty('name')은 true 값을 출력한다.
따라서 ①번줄에서 정의한 testMethod도 위에 설명한 바와 같이 Object.prototype객체에서 상속을 받아 cloneHuman.testMethod()에도 ②번줄
잘 작동한다.

④번줄의 Human.prototype.nameRepetition()은 prototype 체이닝이 아니라 바로 Human.prototype 객체에 접근하여 값을 호출한다.
이 경우에는 nameRepetition메서드의 반환되는 this.name값은 ③줄에서 bind된 android값이다. 여기서 알수 있는것은 prototype으로 this 객체에
바인딩 시킬수 있다는 것이다.



위의 코드를 그림으로 표현하면 다음과 같다.
 ![함수 형식의 prototype상속]({{site.url}}/img/javascript/prototype/prototype_step4.png)



마치며...
---
프로토타입 방식이 이게 설명해서 풀어내려고하면 어렵지 몇번 프로토타입을 이용한 상속을 구현하다보면
그리 어렵지만은 않을것입니다.(프로그래밍은 선 반복숙달후 이해지 않겠어요?!)

앞으로 javascript에 관한 기본 주제들을 가지고 포스팅할 예정입니다. 

모두 즐거운 개발되시길 바라며 화이팅입니다!


참고 
---
인사이드 자바스크립트 - 市고현준,송형주
![https://yuddomack.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-Prototype%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85](https://yuddomack.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-Prototype%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85)
![https://poiemaweb.com/js-prototype](https://poiemaweb.com/js-prototype)
