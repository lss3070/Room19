---
title: "[JavaScript] 이벤트위임과 캡처링버블링"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    -이벤트 위임
    -이벤트 캡처링
    -이벤트 버블링
---

ㄸㄷ
---
프로그래머스 dev-matching을 준비를 하다 이벤트 위임을 필요할 때마다 찾아서 쓰는 수준이여서 한번 제대로 정리를 할 필요가 있을것 같아 이렇게 글을 남긴다.



Javascript의 이벤트 등록
---

```js


```

이벤트 버블링
-------

이벤트 버블링은 javascript의 각 요소에서 이벤트가 발생되면 상위의 요소들에게 전달하는 것입니다.
예제를 한번 살펴보죠

```html
<style>
 .bubbling {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<div class="bubbling three" onclick="console.log(this.className)">three
    <div class="bubbling two" onclick="console.log(this.className)">two
        <div class="bubbling one" onclick="console.log(this.className)">one</div>
    </div>
</div>
```

<html>
<style>
 .bubbling {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<div class="bubbling three" onclick="console.log(this.className)">three
    <div class="bubbling two" onclick="console.log(this.className)">two
        <div class="bubbling one" onclick="console.log(this.className)">one</div>
    </div>
</div>

</html>
제일 하위단의 one을 클릭하게 되면 one의 상위요소인 two three도 같이 실행이 됩니다.어떻게 one 하나만 클릭을 했는데 다른 요소들까지 이벤트가 발생하게 되는것일까요?? 실행순서를 차근차근 정리를하며 알아보죠

1. one을 클릭 하였을때 요소에 등록이 되어있는 onclick 이벤트헨들러가 실행이 됩니다.
2. one의 부모요소인 two에서도 onclick 이벤트 헨들러가 실행이됩니다.
3. twodml 부모요소인 three에도 onclick이벤트 핸들러가 실행이 됩니다.
4. 최상단의 document태그를 만날때까지 onclick 이벤트가 실행이 됩니다.

이러한 동작의 흐름을 이벤트 버블링이라고 지칭합니다. 하위 요소에서 상위요소로 이벤트가 전달이 되는것을 뜻하죠.

이벤트 캡처링
---
이벤트 캡처링은 버블링과 반대되는 개념으로 javscript의 한 요소에서 이벤트가 발생되면 하위의 요소에게 이벤트가 절달되는것을 뜻합니다.

```html
<style>
 .capturing {
    margin: 10px;
    border: 1px solid blue;
  }
</style>
<script>

let capturing = document.getElementsByClassName('capturing');
capturing.forEach(function(div) {
	div.addEventListener('click', function(){
		capture: true // default 값은 false입니다.
	});
});
</script>

<div class="capturing three" onclick="alert('three')">three
    <div class="capturing two" onclick="alert('two')">two
        <div class="capturing one" onclick="alert('one')">one</div>
    </div>
</div>
```
<html>
<style>
 .capturing {
    margin: 10px;
    border: 1px solid blue;
  }
</style>
<script>
  window.addEventListener('DOMContentLoaded', (event) => {
    let capturing = document.getElementsByClassName('capturing');
    for(let element of capturing){
      element.addEventListener('click', e=>console.log(element.className)
      ,{capture:true})
    };
});
</script>

<div class="capturing three">three
    <div class="capturing two">two
        <div class="capturing one">one</div>
    </div>
</div>
</html>

이벤트 캡처링은 자동으로 설정되는 버블링과 달리 capture라는 값을 true로 줘야 합니다.


이벤트 중단
---

vanilla 스크립트로 작업을 하다보면 의도치 않게 버블링이 되는 경우가 많이 있습니다. 이러한 경우를 방지하기 위해 stopPropagation()메서드를 사용합니다, stopPropagation()메서드는 이벤트 버블링이든 캡처링이든 이벤트 헨들러가 작동하는 해당 요소에서만 작동하게끔 하고 다른 요소로 이벤트가 전달되지 않게 합니다.

```html
<style>
 .capturing {
    margin: 10px;
    border: 1px solid blue;
  }
</style>
<script>
  window.addEventListener('DOMContentLoaded', (event) => {
    let stop = document.getElementsByClassName('stop');
    for(let element of stop){
      element.addEventListener('click', e=>{
        console.log(element.className)
        e.stopPropagation();
        })
    };
});
</script>
    <div class="stop two">two
        <div class="stop one">one</div>
    </div>
```
<html>
<style>
 .stop {
    margin: 10px;
    border: 1px solid blue;
  }
</style>
<script>
  window.addEventListener('DOMContentLoaded', (event) => {
    let stop = document.getElementsByClassName('stop');
    for(let element of stop){
      element.addEventListener('click', e=>{
        console.log(element.className)
        e.stopPropagation();
        })
    };
});
</script>
    <div class="stop two">two
        <div class="stop one">one</div>
    </div>
</html>


Event Delegate
---
자 우리는 이벤트 위임에 대해 알아보는 선수작업을 마쳤습니다. 이벤트 버블링과 캡처링을 이용하면 이벤트 위임을 구현할 수 있는데요


먼저 간단한 예제를 살펴보자


```html
<body>
  <table>
    <tr>
      <td>one</td>
      <td>two</td>
      <td>three</td>
    </tr>
    <tr>
      <td>four</td>
      <td>five</td>
      <td>six</td>
    </tr>
    <tr>
      <td>siven</td>
      <td>eight</td>
      <td>nine</td>
    </tr>
  </table>
</body>
```
<html>
<script> 
window.addEventListener('DOMContentLoaded', (event) => {
    let table = document.getElementById('table');
  table.onclick=function(e){
    let target = e.target;
    if(target.tagname!='TD') return;

    console.log(target.innerHTML);
  }
  });

</script>
<body>
  <table id="table">
    <tr>
      <td>one</td>
      <td>two</td>
      <td>three</td>
    </tr>
    <tr>
      <td>four</td>
      <td>five</td>
      <td>six</td>
    </tr>
    <tr>
      <td>siven</td>
      <td>eight</td>
      <td>nine</td>
    </tr>
  </table>
</body>
</html>



참고
---
<https://ko.javascript.info/bubbling-and-capturing>