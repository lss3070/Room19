---
title: "[React] React 18버전에서의 바뀐점"
subtitle: "React"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    HeadlessBrowser
---

개요
---

안녕하세요 어제부로 리액트 18버전이 정식 버전으로 바뀌었습니다.👏👏👏
사실 알파 버전으로 업데이트 된지는 꽤 되었지만 업데이트 사항 복기겸 작성해 보겠습니다.

동시성
---
react 18에서의 가장 중요한 추가 사항은 `동시성`입니다. `동시성`은 그 자체로 기능이 아니며 React가 동시에 여러 버전의 UI를 준비할 수 있도록 해주는 배후 매커니즘입니다.`동시성`은 구현 세부사항으로도 생각할 수 있으며 react에서의 우선 순위 대기열 및 다중 버퍼링과
같은 내부 구현에서 정교한 기술을 사용합니다.

이전의 대부분의 개발자들이 `메모이제이션`이나 `Debounc(디바운스)`기법을 사용해서 사용자의 경험을 개선시키고자 하겠지만, 이는 주된 문제해결을 뒤로 미뤄두는 것일 뿐입니다. 렌더링은 여전히 길을 가로막는 큰 트럭과 같은 존재입니다.

예를 들어 필터링을 이용한 검색 기능을 만든다고 생각하면 `Debounc(디바운스)`이나 `Throttle(쓰로틀)`을 이용하여
특정 최대 빈도수로 업데이트를하거나 사용자가 타이핑을 멈추고 나서만 목록을 업데이트 합니다. 하지만 성능이 안좋은 곳에서는 여전히 버벅거릴 것이며 `Debounc(디바운스)`나 `Throttle(쓰로틀)`이 최적의 방식은 아닙니다.
(버벅거리는 이유는 렌더링이 시작되게 되면 중간에 중단을 할 수가 없기 때문입니다.)

Concurrent React의 주요 속성은 렌더링이 중단 가능하다는 것입니다!!이전의 React버전에서는 동일하게 업데이트가 중단되지 않는 `단일 동기 트랙잭션`입니다. 이렇게 되면 렌더링을 시작하면 사용자가 화면에서 결과를 볼 수 있을때까지 어떤 것도 업데이트를 중단 할 수 없습니다.
하지만 `동시렌더링`에서는 `동기 렌더링`처럼 항상 그렇지 않습니다.React가 업데이트 렌더링을 시작한 후 중간에 일시 중지한 다음에도 나중에 계속 할 수 있습니다. 진행 중인 렌더링을 완전히 포기할 수도 있습니다.React는 렌더링이 중단되더라도 UI가 일관되게 표시됩니다.
이를 위해 전체 트리가 평가되면 끝까지 DOM 변형을 수행하기를 기다립니다. 이 기능을 통해 React에서는 메인 스레드를 차단하지 않고 백그라운드에서 새 화면을 준비할 수 있습니다. 

`동시성`이란 결국 여러 작업들을 처리 할 수 있도록 작업들을 작은 조각들로 나누는 방법이고 react에서 이제 하려고 하는것입니다.
즉 렌더링 과정을 더 작은 작업들로 나누고 스케줄러를 통해서 각 작업들에 중요도에 따른 우선순위를 정합니다.(Time-slicing)라고 부릅니다.

##### concurrent react에 대해서 간단하게 정리해보면

**1.메인 스레드를 블록하지 않는다.**   
**2.동시에 여러 작업들을 처리하고 우선 순위에 따라 각 작업들 간에 전환할 수 있다.**   
**3.최종 결과로 확정하지 않고도 부분적으로 트리를 렌더링 할 수 있다.**



결과적으론 react를 사용하는 방식은 이전과 똑같습니다.prop및 state와 같은 개념은 근복적으로 동일하게 작동이 되며 react는 휴리스틱을 사용하여 업데이트의 급함 정도만 결정하고 몇줄의 코드를 수정해서 사용자가 모든 상호작용에 대해 원하는 사용자의 경험을 얻을 수 있도록 합니다.

이제 React 18버전에서 어떤 기능이 새로 생겼는지
추가된 기능들에 대해 알아보죠

Automatic Batching for Fewer Renders
---

### 배치란 무엇인가?

react에서의 `배치`는 여러개의 상태 업데이트를 한 번의 리렌더링으로 묶는 작업을 말합니다.
만약 하나의 클릭 함수에 동일한 state값을 업데이트 하는 함수가 여러개일 경우에는 하나하나 일일히 업데이트를 하고 리렌더링을 하는 경우에는 자원을 너무 많이 소모하게 되는 이런 비효율성 때문에 함수가 항상 끝마치고나면 리렌더링이 되었습니다.

### 이전의 배치

하지만 이과정은 일반적이지 않습니다. 예를들어 데이터를 외부 소스로 부터 가져외 아래 보이는 handleClick 함수 내부에서 state를 업데이트 하고자하면 react 업데이트를 배치하지 않고 두개의 독립적인 업데이트를 수행했다.
(여기서 일관적이지 못한 이유는 React가 브라우저 이벤트의 업데이트에만 `배치`를 해왔었고 이럴 경우에는 fetch 이벤트의 이벤트 핸들링이 완료되고 난 후 state를 업데이트하기 때문에 적용이 되지 않았다.)
```tsx
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 17과 이전 버전에서는 이 업데이트들이
      // 이벤트가 *진행되는 중*이 아닌, *완료된 후의* 콜백에서 실행되기 때문에
      // 배칭되지 않았다.
      setCount((c) => c + 1); // 리렌더링을 발생시킨다.
      setFlag((f) => !f); // 리렌더링을 발생시킨다.
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```
Demo: [React 17은 외부 이벤트 핸들러를 일괄 처리하지 않습니다 .](https://codesandbox.io/s/trusting-khayyam-cn5ct?file=/src/index.js, "google link")
(콘솔에 렌더가 두 번 찍히는 것을 보자)

**하지만 18버전에서는 모든 업데이트들이 어디서 왔는가에 상관없이 자동으로 배치가된다!!**

[링크를 들어가서 확인해보자!!](https://codesandbox.io/s/morning-sun-lgz88?file=/src/index.js, "google link")


### 배치 원하지 않을 때

대부분의 경우에는 배치가 안전한 절차이지만 몇몇 코드는 state를 변경후 즉시 DOM으로부터 값을 가져오는 것에 의존한다.
이전경우 `flushSync`함수를 사용함으로써 배치를 피해갈 수 있다.

```tsx
import { flushSync } from 'react-dom'; // Note: react-dom, not react

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React has updated the DOM by now
  flushSync(() => {
    setFlag(f => !f);
  });
  // React has updated the DOM by now
}
```

SSR Support for Suspense 
---

### Suspense 개요

SSR은 서버상의 React Component를 이용하여 HTML을 만들고 유저에게 보낼 수 있도록 해준다.
SSR은 유저로 하여금 js 번들이 로딩되고 실행되기 전에 페이지의 컨텐츠를 볼 수 있게 도와준다.

##### react에서의 ssr은 항상 아래와 같은 순서로 작동되어왔다.


**_서버에서 전체 애플리케이션에서 사용할 데이터를 가져온다.**   
**그 후, 서버에서 애플리케이션을 HTML로 렌더링한 후 응답(response)로 보낸다.**   
**그 후, 클라이언트에서 JavaScript를 불러온다.**   
**그 후, 클라이언트에서 서버에서 생성된 HTML에 JavaScript 로직을 연결시킨다.**   


여기서 핵심은 각각의 단계가 다음단계 시작 전에 전체 어플리케이션에 대한 작업을 완료 하여야한다는 점이다.
하지만 이러한 방법은 어플리케이션의 몇몇 일부분만 다른 부분보다 느릴 수 있기에 작지 않는 어플리케이션에 있어 효율적이지 못하다.

React 18버전에선 `Suspense`를 통해 어플리케이션을 작고 독립적인 단위로 쪼개어 위 단계들을 독립적으로 진행 할 수 있도록 하고
전체 어플리케이션의 SSR 프로세스를 막지 않게 해준다.결과적으로 유저들은 컨텐츠를 더 빨리 볼 수 있고 상호작용도 훨씬 더 빨리 할 수 있게 된다.

컴포넌트를 렌더링하고 정적인 html상에서 이벤트 핸들러를 붙여주는 과정을 `하이드레이션(hydration)`이라 한다.
유저들로 하여금 js가 로딩이 되기전에 컨텐츠를 볼 수 있도록 해준다.또한 네트워크 상태가 좋지않은 유저들에게 엄청나게 큰
차이를 가져오고 전반적인 성능 향상을 가져온다.

### SSR의문제점?

오늘 날 SSR의 가장 큰 문제점은 현재 제공되는 서버에서 클라이언트로 HTML을 보내기전에 서버상에서 모든 데이터를 모아놓아야한다.
이 방법은 꽤나 비효율적입니다.
예를 들어 댓글이 있는 글을 렌더링 하고 싶은데 댓글은 빨리 보여주는 것이 중요하기 떄문에 서버사이드 HTML출력을 추가해 주고싶다.
하지만 DB나 API의 속도가 느린데 이건 건들릴 수도 없는 상황이다.
이전 버전에서 해결 할 수 있는 방법으로는 API에서 받은 댓글 데이터들이 HTML에 로드 됐을때 하이드레이션을 해주는 것인데 이렇게 된다면 전체 트리를 렌더링하기 전까지 html전송을 지연시켜줘야되기 떄문에 이방법은 좋지않다.
현재까지 `hydration(하이드레이션)`은 단일 작업만 가능하기에 네비바,사이드바,본문,댓글들의 코드가 전부 불러와야지 할 수 있으며 그 전엔 할수가 없다.
물론 코드 스플리팅을 통해 따로따로 로드 할 수 있지만 이럴 경우 HTML에 있는 댓글을 뺴줘야한다.


하지만 18버전에서는 `waterfall방식`으로 진행이 되었던 부분들 작업을 쪼개 전체 어플리케이션으로 하는 것이 아닌 각각의 부분들로 단계를 수행 할 수 있게 하는 것이다.


##### react 18에서는 Suspense를 이용하여 두개의 주요 SSR기능들을 제공한다.   

**1.서버에서 HTML 스트리밍 형식으로 전달.**   
**2.클라이언트에서 선택적 하이드레이션**   


이 기능들이 어떤 역할을 하고 어떤문제를 해결하는지 아래를 살펴보자.


### HTML 스트리밍
아래 컴포넌트를 살펴보자.
`Suspense`로 Comments라는 컴포넌트를 감싸주고 Comments라는 컴포넌트가 준비되기 전까지 Spinner컴포넌트를 보여준다.
```tsx
<Layout>
  <NavBar />
  <Sidebar />
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>
```
Comment컴포넌트를 `Suspense`로 감싸줌으로써 React에서는 댓글 부분을 가리지 않고 나머지 페이지에 대해 
HTML 스트리밍을 할 수 있도록 도와준다.

![one]({{site.url}}/img/react/eighteenUpdate/suspense_1.png)

이제 초기 HTML에서는 Comments 컴포넌트를 찾을 수 없다.
```html
<main>
  <nav>
    <!--NavBar -->
    <a href="/">Home</a>
  </nav>
  <aside>
    <!-- Sidebar -->
    <a href="/profile">Profile</a>
  </aside>
  <article>
    <!-- Post -->
    <p>Hello world</p>
  </article>
  <section id="comments-spinner">
    <!-- Spinner -->
    <img width="400" src="spinner.gif" alt="Loading..." />
  </section>
</main>
```

여기서 끝나지 않으며 서버단에서 댓글의 데이터가 준비되면 React는 동일한 스트림에 추가되는 HTML과 해당 HTML을
올바른 장소에 위치시키기 위한 작은 인라인 script태그를 보내준다.

```html
<div hidden id="comments">
  <!-- Comments -->
  <p>First comment</p>
  <p>Second comment</p>
</div>
<script>
  // This implementation is slightly simplified
  document
    .getElementById("sections-spinner")
    .replaceChildren(document.getElementById("comments"));
</script>
```

이젠 react가 불러와지기도 전에 늦게 도착한 HTML이 전부 들어오게 된다.

![two]({{site.url}}/img/react/eighteenUpdate/suspense_2.png)

이러한 방법은 우리가 이제 무언가를 보여주기 위해 모든 데이터를 불러올 때까지 기다릴 필요가 없어졌습니다.
화면의 일부 HTML을 보내는 작업을 지연하면 더 이상 모든 HTML을 지연 시킬것인지 해당 파트를 HTML에서 제외할 것인지
선택할 필요가 없다.

### 선택적 hydration(하이드레이션)

문제가 또 하나 남아있습니다.댓글을 위한 js코드가 로딩되기 전에 클라이언트상에서 어플리케이션을 `hydration` 할 수 없습니다.
코드 규모가 크면 꽤나 걸릴 수 있는 작업이 됩니다.

큰 번들 사이즈를 피하기 위해 주로 코드 스플리팅이 사용됩니다.
특정 코드의 부분이 동기적으로 로드 될 필요 없다라고 명시해주면 번들러가 별도의 `<script>` 태그로 분리해준다.
React.lazy를 사용해서 댓글 부분을 코드 스플리팅하여 메인 번들에서 분리 시킬 수 있다.

```tsx
import { lazy } from "react";

const Comments = lazy(() => import("./Comments.js"));

// ...

<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>;
```

이전에 이 방법은 서버 렌더링 환경에서 동작하지 않는다.
하지만 React 18에서 `Suspense`라는 댓글컴포넌트가 불러와지기 전에 어플리케이션을 `hydration`할 수 있게 해준다.
유저의 관점에서 최초의 HTML로 스트리밍된 상호작용이 불가능한 컨텐츠를 보게 된다.

아래 그림을 살펴보자 초록색으로 색칠한 부분이 하이드레이션 된 부분이다.

정적인 부분들을 가져온다.
![one]({{site.url}}/img/react/eighteenUpdate/suspense_1.png)

외부 api에서 가져온 Comments컴포넌트의 html 구조를 가져온다.
![two]({{site.url}}/img/react/eighteenUpdate/suspense_2.png)

정적인 부분들을 다 가져왔으니 하이드레이션을 지시한다.

![three]({{site.url}}/img/react/eighteenUpdate/suspense_3.png)
Comment컴포넌트를 `Suspense`를 묶음으로써 React로 하여금 스트리밍과 하이드레이션이 지연되는 요소로 블로킹되는 것을 막아준다.
이제 두번째 문제점이 해결되었다. 하이드레이션을 하기 위해 모든 HTML이 가져와 지는것을 기다릴 필요가 없다.
![four]({{site.url}}/img/react/eighteenUpdate/suspense_4.png)

### 모든 컴포넌트가 하이드레이션 되기 전 페이지상 상호작용

댓글 부분을 `Suspnse`로 감쌌지만 드러나지 않는 개선점이 하나 더 있습니다.
하이드레이션과정 자체가 더 이상 다른 작업을 할 수 없게 브라우저를 점유하지 않는다.
예를 들어 하이드레이션이 진행되는 동안 유저가 사이드바를 클릭했다고 가정하자.
![hi_one]({{site.url}}/img/react/eighteenUpdate/multi_hydration_1.png)


이제 NavBar와 Post를 가지고 있는 최초의 HTML이 전송된 후에도 서버로부터 Sidebar와 Comments가 스트리밍 될 수 있다.
하지만 이런 경우 `hydration`에도 영향을 준다. 예를들어 두 항목의 HTML이 모두 불러와졌지만, 아직 코드가 불러와지지 않았을 경우 아래와 같이 나타난다.
![hi_two]({{site.url}}/img/react/eighteenUpdate/multi_hydration_2.png)

이제 사이드바와 댓글 코드를 가지고 있는 번들이 불러와진다. React는 둘 모두를 hydration하는데 트리상에서 먼저 발견되는
`Suspense` boundary부터 시작하게 된다.
![hi_three]({{site.url}}/img/react/eighteenUpdate/multi_hydration_3.png)

하지만 예를 들어 유저는 코드가 로드된 댓글 위쪽에 대해 먼저 클릭한다고 가정하자.

![hi_four]({{site.url}}/img/react/eighteenUpdate/multi_hydration_4.png)

React에선 이 동작을 기록하고 클릭한 것이 더 급하기 때문에 댓글 컴포넌트에 대한 `hydration`을 우선순위 부여한다.

댓글 컴포넌트가 `hydration`을 마치면 react는 기록된 이벤트를 다시 실행하고 컴포넌트로 하여금 해당 상호 작용에 반응하도록 한다.
그 후, 이제 react는 다른 작업이 없기 때문에 사이드바를 hydration할 것이다.

이과정에서 우리는 세번째 문제를 해결했습니다. 선택적 `hydration` 덕분에 아무것이나 상호작용하기 위해 모든 것을 다 `hydration`을 하지않아도 됩니다.React는 최대한 빨리 모든 것을 `hydration`할 것이고 유저의 상호작용을 기반으로 화면상에서 급한 부분에 우선순위를 부여할 것이다.


##### 좋은 예시   
**1.데이터 fetch라이브러**   
**2.의도적으로 설계된 로딩상태**   
**3.경쟁상태(race condition)을 피할 수 있도록 돕는다.**   


##### 나쁜 예시
**1.suspense는 데이터 불러오기 작업과 뷰레이어를 결합해 주지 않는다.**   
**2.UI 상에서 로딩상태를 표시 할 수 있도록 조정하는 것을 돕지만 이는 네트워크 로직을 react에 종속 시키는 것은 아니다.**   


새로운 hook 기능
---

### useTransition


`startTransitionAPI`는 업데이트를 긴급 및 비 긴급으로 분류하는데 도움이 됩니다.클릭,선택등과 같이 즉각적인
응답이 필요한 이벤트는 긴급 이벤트로 처리되어야 하며, 검색 결과 표시, 텍스트 강조 표시 등과 같이 즉각적이지 않을 것으로 예상되는
기타 업데이트는 전환 또는 긴급하지 않은 것으로 표시 될 수 있습니다.
또한 `isPending`를 제공하여 사용자가 기다리는 동안 Loading 표시할 수 있습니다.

<iframe height="300" style="width: 100%;" scrolling="no" title="Untitled" src="https://codepen.io/lss3070/embed/ZEvJpow?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/lss3070/pen/ZEvJpow">
  Untitled</a> by lss3070 (<a href="https://codepen.io/lss3070">@lss3070</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

### useDeferredValue


말그대로 어떤 변수의 지연된 값을 반환하는 hook이며 사용자들이 입력을 기반으로 즉시 렌더링하거나 데이터 조회를 기다려야 할 때 인터페이스를 반응적으로 유지하는데 사용됩니다.
`debounce`개념이며 `debounce`을 하기 위해 외부 라이브러리를 사용했어야했는데 내장 기능으로 포함이 되어서 이제 내부함수로 사용할 수 있겠네요.
여기서 `timeoutMs`은 지연되는 시간을 뜻하며 `timeoutMS`에 따라 최대 2초 동안 뒤처져서 백그라운드에서 현재 텍스트로 렌더링 할 수 있습니다.

<iframe height="300" style="width: 100%;" scrolling="no" title="Untitled" src="https://codepen.io/lss3070/embed/XWVeVrE?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/lss3070/pen/XWVeVrE">
  Untitled</a> by lss3070 (<a href="https://codepen.io/lss3070">@lss3070</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

### useId

`useId API`는 서버 렌더링 및 `hydration`중에 안정된 ID를 생성하여 불일치를 방지합니다.
server에서 생성한 react tree와 client에서 그린 react tree사이에 `hydration`이 원할하게 할 수 있도록 일관적인 id가 생성된다.
서버렌더링 콘텐츠 이외의 콘텐츠들을 글로벌카운팅됩니다.

```tsx
import React, { useId } from "react";

function App() {
  const id = useId();
  return (
    <>
      <div className="field">
        <label htmlFor={`${id}-name`} >Name</label>
        <input type="text" name="name" id={`${id}-name`} />
      </div>
      <div className="field">
        <label htmlFor={`${id}-address`} >Address</label>
        <input type="text" aria-labelledBy={`${id}-name ${id}-address`} />
      </div>
      <div className="field">
        <label htmlFor={`${id}-passport`} >Do you have passport?</label>
        <input type="checkbox" name="passport" id={`${id}-passport`} />
      </div>
      </>
  );
}
```

### useInsertionEffect


react의 컴포넌트가 그려지는 순서를 보면 `render -> useLayoutEffect-> commit -> useEffect`
이순서대로 그려진다.

일반적으로 useLayoutEffect를 사용할 때 ref에 대한 접근을 할 수가 없다.

이를 해결하기 위해 React팀은 `useInsertionEffect Hook`을 도입했습니다.
`useInsertionEffect`는 `useLayoutEffect`와 매우 유사하지만 DOM노드의 참조에 액세스 할 수 있습니다.

즉 스타일 지정 규칙만 삽입 할 수 있으며 `<style>`주요 사용 사례는 SVG와 같은 전역 DOM 노드를 삽입하는 `<defs>`입니다. 이것은 클라이언트 측 태그 생성에만 관련이 있으므로 서버에선 실행이 되지 않습니다.

**react에선 internalSource는 props,state,context같은 것이 있다.** 

```tsx
function useCSS(rule) {
  useInsertionEffect(() => {
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(getStyleForRule(rule));
    }
  });
  return rule;
}

function Component() {
  let className = useCSS(rule);
  return <div className={className} />;
}
```

### useSyncExternalStore

externalStore란 외부의 mutable한 store를 뜻하며 주로 redux store,swr store를 뜻한다.
즉 `useSyncExternalStore`는 외부의 store에서 변화를 감지하여 store의 상태를 업데이트 시켜주는 hook이다.


#### Tearing

-Tearing은 시각의 불일치를 뜻합니다.즉 UI에 동일한 상태에 대한 여러 값이 표시됩니다.
React18 버전 이전에는 이 문제가 발생하지 않았습니다.
하지만 React 18이후에는 동시 렌더링을 통해 렌더링 중 React가 일시 중지되기 때문에 이 문제가 발생 할 수 있습니다.

여기에서 구성요소들은 색상을 가져오기 위해 외부 저장소에 액세스 해야합니다.동기 렌더링을 사용하면 UI에서 렌더링되는 생상이 일관됩니다.

![externalstore_1]({{site.url}}/img/react/eighteenUpdate/useSyncExternalStore_1.png)
동시 렌더링에서는 처음 가져온 색상이 파란색입니다. react를 반응시키면 스토어가 빨간색 값을 사용해서 렌더링을 계속합니다. 이로 인해 UI에 불일치가 발생하며 이를 Tearing이라고 합니다.
![externalstore_2]({{site.url}}/img/react/eighteenUpdate/useSyncExternalStore_2.png)
이 문제를 해결하기 위해 React팀은 useMutable을 추가 했습니다.
훅을 통해 가변 외부 소스에서 안전하고 효율적으로 읽을 수 있습니다.

```tsx
import {useSyncExternalStore} from 'react';

useSyncExternalStore(
  subscribe: (callback) => Unsubscribe
  getSnapshot: () => State
) => State

const selectedField = useSyncExternalStore(store.subscribe, () => store.getSnapshot().selectedField);
```

`useSyncExternalStore hook`은 두가지 기능을 사용합니다.   

콜백함수를 등록하는 `subscribe`함수   
그리고 `getSnapshot`은 구독된 값이 마지막 시간 이후 변경되었는지 렌더링되었는지 확인되는데 사용이 됩니다.
값은 문자열이나 숫자와 같이 변경할 수 없는 값이거나 캐시/메모리화된 객체이여야 합니다.




참고   
_[https://reactjs.org/blog/2022/03/29/react-v18.html](https://reactjs.org/blog/2022/03/29/react-v18.html)_
_[https://github.com/reactwg/react-18/discussions/37](https://github.com/reactwg/react-18/discussions/37)_
_[https://github.com/reactwg/react-18/discussions/21](https://github.com/reactwg/react-18/discussions/21)_
_[https://blog.saeloun.com/2021/12/09/react-18-useid-api](https://blog.saeloun.com/2021/12/09/react-18-useid-api)_
_[https://blog.saeloun.com/2021/09/09/react-18-introduces-starttransition-api](https://blog.saeloun.com/2021/09/09/react-18-introduces-starttransition-api)_
_[https://blog.saeloun.com/2021/12/30/react-18-usesyncexternalstore-api](https://blog.saeloun.com/2021/12/30/react-18-usesyncexternalstore-api)_
_[https://blog.saeloun.com/2021/12/30/react-18-usesyncexternalstore-api](https://blog.saeloun.com/2021/12/30/react-18-usesyncexternalstore-api)_
