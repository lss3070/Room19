---
title: "[프로그래머스/JavaScript] 이벤트위임과 캡처링버블링"
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
프로그래머스 dev-matching을 준비를 하다 버블링과 캡쳐링에 대해 애매모호하게 아는듯한것 같아.....
필요할 때마다 찾아서 쓰는 수준이여서 한번 제대로 정리를 할 필요가 있을것 같아 이렇게 글을 남긴다.



Javascript의 이벤트 등록
---

```js


```

이벤트 버블링
-------

이벤트 버블링은 javascript의 element객체에서 이벤트를 발생시키고 더상위의 element객체 요소들에게 전달하는 것입니다.
예제를 한번 살펴보죠

```html
<style>
  form *,form {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<form onclick="alert('form')">FORM
  <div onclick="alert('div')">DIV
    <p onclick="alert('p')">P</p>
  </div>
</form>
```

<html>
<style>
 .bubbling {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<div class="bubbling" onclick="alert('form')">three
    <div class="bubbling" onclick="alert('div')">two
        <div class="bubbling" onclick="alert('p')">one</div>
    </div>
</div>

</html>

p태그를 클릭하게 되면 p태그의 상위요소인 div와 form태그도 같이 실행이 됩니다.어떻게 이런일이 발생하는지 정리하죠

1. p태그를 클릭 하였을때 p태그 등록이 되어있는 onclick 이벤트헨들러가 실행이 됩니다.
2. p태그의 부모요소인 div태그에도 onclick 이벤트 헨들러가 실행이됩니다.
3. div태그의 부모요소인 form 태그에도 onclick이벤트 핸들러가 실행이 됩니다.
4. 최상단의 document태그를 만날때까지 onclick 이벤트가 실행이 됩니다.

이러한 동작의 흐름을 이벤트 버블링이라고 지칭합니다. 하위 요소에서 상위요소로 이벤트가 전달이 되는것을 뜻하죠



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
divs.forEach(function(div) {
	div.addEventListener('click', function(){
		capture: true // default 값은 false입니다.
	});
});
</script>

<div class="capturing" onclick="alert('three')">three
    <div class="capturing" onclick="alert('two')">two
        <div class="capturing" onclick="alert('one')">one</div>
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
let capturing = document.getElementsByClassName('capturing');
divs.forEach(function(div) {
	div.addEventListener('click', function(){
		capture: true // default 값은 false입니다.
	});
});
</script>

<div class="capturing" onclick="alert('three')">three
    <div class="capturing" onclick="alert('two')">two
        <div class="capturing" onclick="alert('one')">one</div>
    </div>
</div>
</html>

이벤트 중단
---


이벤트 위임
---



참고
---
<https://ko.javascript.info/bubbling-and-capturing>