---
title: "[JavaScript] Intersection Observer API"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:

---

Intersection Observer API는 타겟 요소와 상위 요소 또는 최상위 document 의 viewport 사이의 intersection 내의 변화를 비동기적으로 관찰하는 방법입니다.

viewport란?
---
모바일 브라우저에서는 viewport라고 알려진 가상의 화면에 페이지를 렌더링합니다.


viewport는 ~~

Intersection Observer API
---
Intersection Observer API가 주로 사용되는 떄는
1. 이미지나 컨텐츠 로딩을 효율적이게 하기 위한 lazy loading
2. 많은 데이터를 한번에 로드 하지않고 스크롤시에 조금씩 분할해서 로드하는 infinite-scroll
3. 광고나 배너가 노출되었는지 확인할 떄
4. 애니메이션 작업을 수행여부를 노출여부를 통해 확인 할 때

ee
===
IntersectionObserver의 생성자에는 callback과 options이라는 인수값을 가집니다.

```js
let io = new IntersectionObserver(callback,options) // 관찰자 초기화
const callback = (entries,observer)=>{
    entries
}

const options ={
    root:,
    rootMargin:,
    threshold:
}
io.observe(element) /
```
callback
---
callback은 target이 가시성이 threshold의 값만큼 도착하였을 때 실행되는 함수입니다.

IntersectionObserverEntry 배열인 `entries`와 자기 자신인 `observer`가 인수로 전달이 됩니다.

IntersectionObserverEntry의 속성
---

`boundingClientRect` 관찰 대상의 사각형 정보를 뜻하며


`intersectionRect` 관찰 대상의 교차한 영역 정보


`intersectionRatio` 관찰 대상의 교차한 영역의 백분율

`isIntersection` 관찰 대상의 교차 상태

`rootBounds` 지정한 root의 사각형 정보

`target` 관찰 대상 요소

`time` 변경이 발생한 시간 정보

options
---
1. root는 관측대상이 되는 element를 지칭합니다.
root의 기본값은 ViewPort입니다(null인경우)

2. rootMargin은 root요소의 margin값을 지정해줍니다.
threshold 나 교차 상태를 점검할 때 margin 값이 지정 되어 있다면, 이를 활용하여 계산합니다.
값은 문자열이며 css에서 margin값을 입력할 때랑 동일하게 입력하면 됩니다.

3. threshold는 target이 
옵저버가 실행되기 전 target의 가시성이 얼마나 필요한지 백분율로 표시한 것입니다.
target이 root와 몇 % 교차하였을때 실행할지 결정하는 값이며 Array타입의 배열값이나 Float타입의 단입값으로 입력이 가능합니다.

ex)
값이 0.2일 경우 스크롤 하는 도중 element가 20% 보였을 때 실행합니다.
값이 [0.2,0.3]의 경우 element가 20%,30% 보였을 때 실행합니다.

![eventloop]({{site.url}}/img/javascript/intersectionobserver/threshold.png)

IntersectionObserver의 메소드
---

IntersectionObserver.observe(targetElement)

IntersectionObserver.unobserve(targetElement)

IntersectionObserver.disconnect()

IntersectionObserver.takerecords()


사용예시
===

lazy loading
---

Intersection​ Observer API를 이용하면 지연 로딩을 손쉽게 구현 할 수 있다. 구현 방법은 대상 element가 뷰포트에 들어왔을 때 이미지 경로를 변경 한 뒤 unobserve시킨다.

<iframe height="500" style="width: 100%;" scrolling="no" title="poNYqKy" src="https://codepen.io/lss3070/embed/poNYqKy?height=265&theme-id=light&default-tab=js,result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/lss3070/pen/poNYqKy'>poNYqKy</a> by lss3070
  (<a href='https://codepen.io/lss3070'>@lss3070</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

Infinite Scrolling
---



사용가능 브라우저..
---




참고
---
<https://developer.mozilla.org/ko/docs/Web/API/Intersection_Observer_API>
<https://heropy.blog/2019/10/27/intersection-observer/>
<https://tech.lezhin.com/2017/07/13/intersectionobserver-overview>