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

<div class="bubbling three" onclick="alert(this.className)">three
    <div class="bubbling two" onclick="alert(this.className)">two
        <div class="bubbling one" onclick="alert(this.className)">one</div>
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

<div class="bubbling three" onclick="alert(this.className)">three
    <div class="bubbling two" onclick="alert(this.className)">two
        <div class="bubbling one" onclick="alert(this.className)">one</div>
    </div>
</div>

</html>
제일 하위단의 one을 클릭하게 되면 one의 상위요소인 two three도 같이 실행이 됩니다.어떻게 one 하나만 클릭을 했는데 다른 요소들까지 이벤트가 발생하게 되는것일까요?? 그 이유는 브라우저의 이벤트 발생순서에 있습니다. 브라우저에서는 이벤트가 발생할 시에 이벤트를 취상위에 있는 화면요소까지 이벤트를 전달시킵니다.


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
    for(let element of capturing){
      element.addEventListener('click', e=>alert(element.className)
      ,{capture:true})
    };
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
      element.addEventListener('click', e=>alert(element.className)
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

 자바스크립트로 작업을 하다보면 의도치 않게 버블링이 되는 경우가 많이 있습니다. 이러한 경우를 방지하기 위해 stopPropagation()메서드를 사용합니다, stopPropagation()메서드는 이벤트 버블링이든 캡처링이든 이벤트 헨들러가 작동하는 해당 요소에서만 작동하게끔 하고 다른 요소로 이벤트가 전달되지 않게 합니다.

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
        alert(element.className)
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
        alert(element.className)
        e.stopPropagation();
        })
    };
});
</script>
    <div class="stop two">two
        <div class="stop one">one</div>
    </div>
</html>


이벤트 위임
---
자 우리는 이벤트 위임에 대해 알아보는 선수작업을 마쳤습니다. 이벤트 버블링과 캡처링을 이용하면 이벤트 위임을 구현할 수 있는데요

먼저 간단한 예제를 살펴보죠


```html
<script>
 let table = document.getElementById('table');
  table.onclick=function(e){
    let td = e.target.closest('td');
    if(td&&table.contains(td)) alert(td.innerHTML);
  }
</script>
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
    let td = e.target.closest('td');
    if(td&&table.contains(td)) alert(td.innerHTML);
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

각 td객체에 이벤트 헨들러를 할당할 수도 있지만 그러한 방식은 이쁘지도 않고 시간도 오래걸릴 수 있습니다.
하지만 각요소들의 공통조상이 존재하고 태그명에 같은 경우 loop문을 통해 이벤트 위임을하여 간단하게 코드를 짤 수 있습니다.

1. closest 메서드는 대상의 상위요소 중 selector와 일치하는 가장 가까운 요소를 반환합니다.
2. td가 조건에 부합하여야지 
3. 선택된 td의 값을 console.log로 출력해 줍니다.

이런방식으로 이벤트 위임을하게 되면 td의 개수와 상관없이 모든 td태그에 클릭이벤트를 빠르게 구현 할 수 있습니다.

다른방식의 이벤트 위임
---

위의 td태그 처럼 똑같은 기능을하는 태그가 아닌 서로 다른 요소들에게는 어떤식으로 이벤트 위임을 할 수 있을까요??
제일 간단한 방법으로는 각각의 요소에 이벤트 핸들러를 할당하는겁니다. 하지만 이러한 방법보다 깔끔한 방법이 있습니다. 바로 data-action 속성에 호출할 메서드를 할당해 주는 방법이죠!!

```html
<script>
class Menu{
  constructor(element){
    this._element = element;
    element.onclick = this.onClick.bind(this);
  }

  save(){
    alert('save');
  }
  load(){
    alert('load');
  }
  search(){
    alert('search');
  }
  onClick(event){
    let action = event.target.dataset.action;
    if(action){
      this [action]();
    }
  }
}

new Menu(menu);
</script>
<div id="menu">
  <button data-action="save">save</button>
  <button data-action="load">load</button>
  <button data-action="search">search</button>
</div>
```
<html>
  <script>
 window.addEventListener('DOMContentLoaded', (event) => {
  let menu = document.getElementById("menu");
  new Menu(menu);
  });
  class Menu{
  constructor(element){
    this._element = element;
    element.onclick = this.onClick.bind(this);
  }
  save(){
    alert('save');
  }
  load(){
    alert('load');
  }
  search(){
    alert('search');
  }
  onClick(event){
    let action = event.target.dataset.action;
    if(action){
      this.[action]();
    }
  }
}
  </script>

  <div id="menu">
    <button data-action="save">save</button>
    <button data-action="load">load</button>
    <button data-action="search">search</button>
  </div>
</html>


es6에서 추가된 class를 이용하여 이벤트 위임을 구현해 보았습니다.
이전의 코드들보다 훨씬 더 깔끔해진 것 같죠??
이러한 방식으로 코드를 짜면 언제든지 추가 및 제거를 할 수 있어 코드를 수정하기에 조금 더 유연해 집니다.





참고
---
<https://ko.javascript.info/bubbling-and-capturing>