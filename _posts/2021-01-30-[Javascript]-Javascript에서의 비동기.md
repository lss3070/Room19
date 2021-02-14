---
title: "[Javascript] Javascript에서의 비동기 프로그래밍"
subtitle: "기본기"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    비동기프로그래밍
    Promise
    async/await
---

자바스크립트는 싱글 스레드에서 동작하는 언어입니다.그래서 기본적으로 동기방식으로 작동을 합니다.
동기방식이라함은 코드가 실행될때 현재 작업이 완료되야 다음줄로 넘어가는 것을 의미합니다.
즉 한번에 한가지일밖에 처리하지 못한다는 뜻이죠.

ex) 동기 : 요청을 보낸 후 응답을 무조건 받아야 다음 동작을 실행
    비동기 : 요청을 보낸 후 응답에 상관없이 다음 동작을 실행

이런 싱글 스레드 방식으로 작동하면 단점이 있는데 바로 비동기 방식으로 사용해야되는 경우입니다.

자바스크립트에서 비동기 함수를 사용하는 경우는 크게 세가지가 있습니다.
[dot] 네트워크 요청을을 호출하는 경우
[dot] 파일을 읽고 쓰는 파일 시스템의 경우
[dot] 시간지연을 사용하는 기능의 경우

그럼 동기방식으로 작동하는 프로그램을 굳이 왜 비동기방식으로 작동하게해서 코드를 작성하는데 힘들게 만들까요..?
동기방식의 프로그래밍을 할 경우 오래 걸리는 웹 요청이나 데이터 처리 작업을 위해 프로그램을 보류하는 상황이 발생하고 이로 인해 성능저하의 문제가 발생하게 됩니다.
그리고 또 다른 이유로는 거의 대부분의 모듈이나 라이브러리가 비동기로 구성이되어있기 때문이죠.😭😭😭


자바스크립트에서의 비동기 프로그래밍을 처리하는 방식은 크게 3가지가 있습니다. 콜백함수로,Promise,async/await 이 3가지를 살펴보기 앞서 간단하게 자바스크립트 비동기프로그래밍에 대해 살펴보죠


```js
console.log(`before ${new Date()}`);
function After(){
    console.log(`after ${new Date()}`)
}

setTimeout(After,10*1000) //10초
console.log("happen!");

```

출력문
```
before Sun Jan 31 2021 19:00:33 GMT+0900 (대한민국 표준시)
happen!
after Sun Jan 31 2021 19:00:43 GMT+0900 (대한민국 표준시)
```

비동기 프로그래밍의 기본적 예제인 setTimeout함수를 이용해서 만들어 보았습니다.
자바스크립트를 다룬지 얼마 안됐을 경우 setTimeout함수나 ajax 함수를 실행시켰을 때 실행순서로 인해 종종 당황했는 경험들이 있을겁니다.
우리가 작성한 코드가 순서대로 작동할 것이라고 생각을 했지만 원하는대로 실행되지 않고 비동기 방식으로 실행이 되죠.
만약 우리가 원하는 방식의 동기방식으로 프로그램이 실행됐다면 어떻게 됐을까요..??
첫째줄의 before라는 구문이 출력되고 컴퓨터는 10초동안 대기를 한 후 다음 줄의 함수를 실행시켰을겁니다. 대기하는 동안 어떠한 입력과 업데이트가 이루어지지 않을거고요.
결과값이 내가 원하게 나오든 나오지 않든 어떻게든 비동기 방식으로 프로그래밍을 구현하였습니다.
하지만 이런식의 콜백함수는 여러가지 문제점이 있습니다 한번 살펴보시죠


콜백함수
---
콜백함수를 이용해서 비동기프로그래밍을 구현하는것은 완벽하지도 않고 오래된 매커니즘입니다.
만약 한번에 여러가지의 콜백함수가 중첩이 되어있다면 관리하는데 상당히 어려워질겁니다.아래 예제를 살펴보죠

```js
    first(function(){
        second(function(){
            thrid(function(){
                //...
            })
        })
    })

```

이런 코드들을 그 유명한 콜백 헬이라 부릅니다. 중괄호로 끊없이 중첩된 코드들의 블록...
가독성도 가독성이지만 이렇게 만들고 나면 더욱 힘든것이 에러 처리에 관한 문제입니다. 
먄약 저기서 에러라도 난다면 읽기 힘들뿐더러 코드를 수정하는데 짜증만 날 것입니다.
또 다른 문제를 살펴보죠.

```js

const fs = require('fs');
function read(){
    try{
        fs.readFile("test.txt",function(e,data){
            if(e) throw e;
        });
    }catch(e){
        console.log(`warining,{e}`);
    }
    read();
}
```

위의 코드는 또다른 문제를 발생시킵니다 잘 작동 될것처럼 보이는 코드지만 실제로 동작을 시켜보면
작동이 되지않습니다.
왜일까요..??
예외처리부분에서의 문제인데 예외처리구문인 try catch구문은 블록이 같은 함수내에서만 작동합니다.
예외처리구문은 read함수안에 있지만 정작 예외처리가되는 'fs.read()'함수는 콜백으로 호출되는 익명함수 안에서 일어났죠
위 예제는 콜백함수는 아예 호출되지않는 경우를 방지하는 기능도 없습니다.

이런 부분들은 해결할 수 없는 문제는 아니지만 이러한 콜백 에러가 늘어날수록 관리하기 어려운 코드가 되며 유지보수나 관리하기
어려워집니다.이러한 문제점들을 보완하기 위해 Es6에서는 Promise가 나왔습니다.


Promise
---
Promise는 콜백의 단점을 해결하려는 시도 속에서 만들어 졌으며 프로미스는 일반적으로 안전하고 관리하기 쉬운 코드를 만들수 있게 해줍니다.

간단한 에제를 살펴보죠

```js
function start(param){
    return new Promise(function(resolve,reject){
        if(param=="ok"){
            resolve(param);     
            console.log("resolve");
        }
        else{
            reject(param)
        }
        
    })
}
start("ok").then(function(){
    console.log("success");
},function(){
    console.log("fail");
});
console.log("end");
```

new Promise(),resolve(),reject(),then()과 같은 프로미스 API를 사용하는 구조로 바뀌었습니다. 여기서 resolve(),reject(),then()에 대해 자세히 알아보죠

####프로미스 상태


promise를 사용할때 꼭 알아야 하는는 개념은 프로미스 코드를 실행할때마다 프로미스의 진행상태를 저장하는 프로미스 상태입니다.
상태를 저장하는 이유는 코드를 연속적으로 실행하지 않고 소스 코드 끝까지 내려갔다가 다시 올라와서 실행하므로 진행상태에 대한 정보가 필요하기 때문입니다. 프로미스의 상태는 크게 세가지로 분류됩니다.

Fufill : 성공
Pending : 이행
Reject : 실패

Pending(대기)
```js
return new Promise(function(resolve,reject){
    ///
})
```
pending(대기) 상태에서는 위 코드와 같이 new Promise()로 인스턴스를 생성합니다.
new Promise()메서드를 호출할 때엔 콜백함수를 선언할 수 있고 콜백함수의 인자값은 resolve와 reject입니다.

Pending(이행)
```js
function Pending(){
return new Promise(function(resolve,reject){
    resolve();
    });
};
Pending().then(function(value){
    console.log(value)
})

```
pending(이행)상태에서는 함수의 인자인 resovle를 실행하면 됩니다.성공적으로 실행됐을 경우에만 발생합니다.


Rejected(실패)
```js
function Rejected() {
  return new Promise(function(resolve, reject) {
    reject(new Error("error!!"));
  });
}
Rejected().then().catch(function(e){
    console.log(e)
})
```
블록의 코드 실행이 실패한 상태를 나타냅니다.reject를 호출하여 Promise를 실패상태로 만듭니다
catch()로 실패한 이유를 받을 수 있습니다.


프로미스를 도식화하면 아래와 같습니다.
 ![프로미스도식화]({{site.url}}/img/javascript/promise/promises.png)


이제 promise에 대한 기본사항도 알아보았으니 Promise를 이용하여 연속적으로 비동기 동작을 하는 함수를
만들어보죠

```js

new Promise(function(resolve,reject){
    setTimeout(function(){
        resolve(1)
    },5*1000);
})
.then(function(result){
    console.log(result)
    return result *2;
})
.then(function(result){
     throw new Error("second error!");
    console.log(result)
    return result*4;
})
.then(function(result){
    console.log(result);
    return result*8;
}).catch(function(error){
    console.log(error);
})
```
콜백함수로 위 구문을 구현하였으면 콜백안에 콜백 또 콜백안에 콜백...계속적으로 반복되었겠지만
Promise를 사용하여 그나마(??) 많이 깔끔해졌습니다.

//사족임 붙이지말자
위 예시를 보면서 느끼셨겠지만 코드에 대한 가독성은 많이 좋아졌지만 복잡한 함수에서 중간에 에러가 발생해서
꼬인다면 어떨까요... 일일히 디버거로 한줄한줄 뜯어봐야될겁니다.


async/await
---
드디어 aync/await 항목에 도착했네요!!
aync/awiat는 es8에 등장하게된 문법이고 이 특별한 문법을 사용하면 Promise를 좀 더 편하게 사용하실 수 있어요. 위의 방식들처럼 복잡하지 않고 깔끔하게 비동기적인 프로그래밍을 만들 수 있습니다.

```js
async function GetAuth(id){
    try{
        let auth = await GetUsr(id);
        if(auth===10) console.log("인증된 사용자입니다.");
        else conseol.log("인증되지않은 사용자입니다.");
    }catch(e){
        console.log(e);
    }
}
async function GetUsr(id){
    return id=="usr"?10:20
}
GetAuth("usr");

```
사용하는 방식도 간단합니다! 함수의 선언자 앞에는 async를 붙여주고 해당 함수를 호출하는 부분 앞에는 await를 붙여줍니다. (일반함수내에서 혼용해서 쓰실 수 없어요!!)
일반적으로 동작하는 방식과 비슷하지 않나요?? 블록내에서도 에러처리구문도 사용가능해서..
 더 진가를 발휘하는 순간은 여러개의 비동기 작업을 시작할 때입니다. 아래 예제를 살펴보죠

```js
async function GetUsr(id){
    let usrInfo={
        auth:null,
        name:null,
        address:null
    }
    try{

       usrInfo.auth = await GetAuth(id);
        usrInfo.name = await GetName(id);
        usrInfo.address = await GetAddress(id);
    }catch(e){
        console.log(e);
    }
    console.log(usrInfo);
    return usrInfo
}
async function GetAuth(id){
    return `${id}_auth`
}
async function GetName(id){
    return `${id}_name`
}
async function GetAddress(id){
    return `${id}_address`
}

GetUsr("usr");

```
이처럼 일반함수처럼 간단하게 비동기방식으로 프로그래밍을 만들 수 있습니다. 
굳이 기존의 비동기 처리 코드 방식으로 사고하지 않아도 만들수 있기 때문에 좀 더 편합니다.


마무리
---
Javascript의 비동기 문법을 처음 접하신다면 Promise에 대해서 이해하시는데 시간이 좀 걸리실겁니다.
하지만 본인의 소스코드에 녹여서 적용시켜보시면 금방 익히실 수 있을테니 너무 걱정안하셔도 됩니다.



참고 
---
러닝 자바스크립트 - 市이선 브라운
<https://velog.io/@two_jay/callback-%EA%B3%BC-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D>
<https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise>


