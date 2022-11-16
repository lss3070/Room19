---
title: "[React] NextJS 13 변경점 (2)"
subtitle: "NextJs"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    React
    NextJs
---

이전 게시물에 이어서 NextJs 13에서의 변경점에 대해서 살펴보죠


### Server Compoent 

Server Component를 살펴보기에 앞서 react에서 렌더링되는 환경은 client와 server 이 두가지가 있습니다.

* **client**
client는 사용자들의 요청을 받아 서버로 보내고 서버의 응답을 사용자와 상호작용할수 있는 부분.
* **server**
server는 client의 요청을 수신하고 어떤 계산을하고 응답을 돌려주는 데이터센터를 말합니다. 

nextjs13에서는 위 server에서 렌더링이되는 server compoent가 기본적으로 설정이 되어있습니다.
(하지만 `app`디렉터리 한정이며 호환성 때문에 이전의 `pages`디렉토리에서는 사용할 수 없습니다.)

![component-tree]({{site.url}}/img/react/next13/component-tree.webp)

그렇다면 왜 Next 13버전에서는 왜 기본설정으로 Server Component를 사용하였을까요?? 

**Server Component**를 사용하면 아래와 같은 이점들이 존재하기 때문입니다.

```
* 서버는 마크다운을 html로 렌더링하기 위한 npm 패키지와 같은 무거운 코드 모듈들을 가볍게 사용할 수 있습니다. 클라이언트에서 작업을 하는 경우에는 Js코드 번들을 계속 받아야 되지만
서버 렌더링을 그런 번거로운 작업이 필요없습니다.
* 서버는 db와 graphql 등 다양한 엔드포인트에 직접적으로 액세스 할 수 있습니다.서버는 api의 엔드포인트를 거치지 않고 필요한 데이터를 바로 가지고 올 수 있으며 데이터 소스와 가깝게 배치가 되어있어 브라우저보다 더 빠르게 데이터를 자겨올 수 있습니다.
```

위 서버컴포넌트들의 이점들을 종합하여 살펴보면 Component Tree 어디에서나 backend로 접근이 가능하며 (기존 getServerProps가 접근할 수 있었지만 최상위에서만 사용하능) client의 state를 잃지 않고 다시 렌더링 할 수 있습니다.이로 인해 페이지 전체를 서버에서 렌더링 할 필요없이 일부만 서버에서 렌더링 한 후 페이지에 주입하는 것이 가능해졋습니다.


### 그렇다면 Server Component는 Server Side Rendering인가?
Server Component는 Server Side Rendering과 비슷해 혼동을 할 수 있지만 둘은 다른 개념입니다.이름이 비슷하기에 헷갈릴수도 있죠 

먼저 Server Side Rendering은 모두 아시다 싶이 hydration이 일어날 때 JS번들파일이 클라이언트에 전송이 됩니다. (어떻게 보면 client component죠)

둘의 차이점을 간단하게 비교해 보자면

* `Server Component`의 코드는 클라이언트로 전송이 되지 않지만 `SSR`은 hydration이 일어날 때 Js번들파일이 클라이언트에 전송이됩니다.
* `Server Component`는 페이지 레벨에 상관없이 접근이 가능하며 
업데이트 전 `SSR`을 이용하여 렌더링하는 경우 정적인 HTML을 받아서 렌더링한 후 js번들파일이 클라이언트에서 다 받아지면 hydration 작업이 일어나 js번들파일이 동적인 기능을 수행하게 하였습니다.
하지만 이러한 상호작용은 hydration이 일어난 이후에는 다시 사용할 수 없습니다.

NextJS 13에서는 컴포넌트가 기본적으로 Server Componet가 설정이 된다고 하는데 그렇다면 Client Component는 만들 수 있을까요?

### Client Component
기본적으로 **Server Componen**t가 적용이 되기 때문에 **Client Component**를 사용하려면 아래와 같이 상단 지시문에 'use client'를 선언을 해주어야 합니다.

```tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

#### 그렇다면 클라이언트 컴포넌트는 언제 사용해야 될까요?

![server-vs-client]({{site.url}}/img/react/next13/server-vs-client.png)

전반적으로 서버와 통신을 할때는 서버 컴포넌트사용 클라이언트 관련 기능이나 변경사항이 있으면 클라이언트 컴포넌트를 사용하면 될것 같습니다.


### 외부 라이브러리에서 client 컴포넌트 사용
`'use client'`는 서버 컴포넌트의 일부로 새롭게 도입된 기능입니다.서버 컴포넌트가 새롭기 때문에 
오늘날 클라이언트 전용 기능을 사용하는 npm 패키지의 많은 Component 지원이 되지 않습니다.. 
이러한 Compoent는 자체적으로 `'use client'`지시문을 이용하여 자체 클라이언트 Component 내에서 예상대로 작동하지만 서버 컴포넌트내에서는 작동하지 않습니다.

예를들어 Server Component에서 acme-carousel 컴포넌트를 사용하려고 합니다.

```tsx
import { AcmeCarousel } from 'acme-carousel';

export default function Page() {
  return (
    <div>
      <p>View pictures</p>

      {/* 🔴 Error: `useState` can not be used within Server Components */}
      <AcmeCarousel />
    </div>
  );
}
```

하지만 Next.js에서는 AcmeCarousel가 클라이언트 기능을 사용하고 있다는 것을 모르기 때문에 에러가 뜹니다.
이러한 부분을 해결하기위해 타사의 Component를 래핑해줘야합니다.

```tsx
'use client';

import { AcmeCarousel } from 'acme-carousel';

export default AcmeCarousel;
```

`<Carousel/>` 컴포넌트는 Server Component에서 직접 사용할 수 있습니다 .

```tsx
import Carousel from './carousel';

export default function Page() {
  return (
    <div>
      <p>View pictures</p>

      {/* 🟢 Works, since Carousel is a Client Component */}
      <Carousel />
    </div>
  );
}
```


fetch의 변화
---
NextJs 13버전에서는 app 디렉토리에서 데이터를 가져오고 캐싱하여 유효성 검사를 다시하는 기능이 추가 되었습니다.
이제 데이터를 가져올 때 `getStaticProps`,`getServerSideProps를` 쓰지 않아도 됩니다. 왜냐면 이제 fetch 함수에서 이 두가지 함수의 기능을 제공하기 때문이죠.

### Data Fetching
업데이트 된 NextJs에서 데이터를 가져오는 권장 방법은 서버 컴포넌트에서 사용하는 것입니다.
공식 문서에 따르면 서버 컴포넌트를 사용하면 많은 이점이 있습니다.
- 백엔드 데이터를 클라이언트단에서 실행되지 않기 때문에 직접 액세스 할 수 있음.
- 액세스 토큰 및 api 키와 같은 민감한 정보를 서버에 보관하여 어플리케이션을 쉽고 안전하게 보호한다.
- 데이터를 가져오고 동일한 환경에서 컴포넌트를 렌더링합니다. 이렇게 하면 클라이언트와 서버간의 양방향 통신과 클라이언트 측의 작업이 줄어듭니다.
- 클라이언트에서 여러 개별 요청 대신 단일 왕복으로 여러 데이터를 가져올 수 있다.
- client-server waterfall현상이 줄어듭니다.
- Server Copmpoent는 Js번들을 보내지 않기 때문에 클라이언트의 JsBundle 크기가 줄어듭니다.

<!-- 한가지 중요한 점은 데이터를 전송할때보다 데이터를 가져올 때 권장하는 것입니다.수많은 요청의 중복을 제거하고 캐시된 결과를 반환합니다.이로 인해 서버가 무거운 작업을 수행하고 클라이언트가 데이터를 요청하도록 하는 것이 효율적입니다. -->

여기서 중요한 점은 데이터를 전송할 때보다 데이터를 가져올때 서버 컴포넌트를 권장한다는 것입니다. 수많은 요청들의 중복을 제거하고 캐시된 결과를 반환합니다. 이로 인해 서버가 무거운 작업을 하지않게 도와주죠.

### Async Compoent & Data fetching

Next.js 13에서는  Async Compoent에 대한 데이터를 가져오는 새로운 접근 방식을 도입합니다. Async Compoent를 사용하면 async 및 await와 함께 Promise를 사용하여 Compoent를 렌더링 할 수 잇습니다.
Promise를 반환하는 외부서비스 또는 다른 API에서 데이터를 가져와야 하는 경우 Component를 비동기로 선언하고 결과값을 받습니다.

```ts
async function getData(){
  const res = await fetch('http://api.example.com')
  return res.json();
}
```

```tsx
export default async function AccountPage() {
  const table = await getData();

  return (
    <ul>
      {table.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}

```

### Data Fetch의 유형
**data fetch**에는 다양한 유형이 있습니다.
이는 데이터 저장 여부와 관련이 있으며 아래와 같은 옵션들이 있습니다.

```
Cache data: This is the default
Dynamic data: No cache
Revalidate: Cache for x time
```

**data fetch**의 기본 값은 항상 모든 항목을 캐싱하는 것입니다.
다음 과 같이 데이터를 전달 할 수 있습니다. SSG의 개념입니다.
```
fetch('https://...', { cache: 'force-cache' });

// 위 data fetch와 같으며 cache를 생략할 수 있습니다.
fetch('https://...');
```

다음은 SSR의 개념과 비슷하며 캐싱을 비활성화해서 요청시마다 새로운 데이터를 가져올 때 사용합니다.

```
fetch('https://...', { cache: 'no-store' });
```

그리고 마지막 옵션에서 캐싱할 시간을 정할 수 있습니다.이건 ISR의 개념이며 무언가를 캐싱하는 시간을 초단위로 설정합니다.

```
fetch('https://...', { next: { revalidate: 10 } });
```


### 병렬 및 순차 data fetching

데이터를 로드한 후 또 다른 데이터를 로드하는 경우가 있습니다.
예를 들어 두개의 데이터를 같이 로드하는 경우입니다.
이 경우 두 호출이 있기 때문에 두 호출이 완료되고 나면 렌더링을 할 수 있는 병렬 fetching을 사용해야합니다.


#### 병렬 data fetch
```jsx
async function getUser(username){
  const res = await fetch(`https://api.example.com/user/${username}`);
  return res.json();
}

async function getUserTodos(username){
  const res = await fetch(`https://api.example.com/user/${username}/todos`);
  return res.json();
}
async function Todos({ promise }) {
  // Wait for the albums...
  const albums = await promise;

  return (
    <ul>
      {albums.map((album) => (
        <li key={album.id}>{album.name}</li>
      ))}
    </ul>
  );
}

export default async function Page({ params: { username } }) {
  // Queue up the promises here...
  const _user = getUser(username);
  const _todos = getUserTodos(username);

  // Wait for the user...
  const user = await _user;

  return (
    <>
      <h1>{user.name}</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <Todos promise={_todos} />
      </Suspense>
    </>
  );
}
```

위 예시는 두개의 호출을 동시에 로드하여 compoent를 구성하는 방법을 보여줍니다.하지만
한번에 로드되는것이 아닌 사용할 수 있는경우에만 렌더링을하게 됩니다.
다음으로는 `getUser`,`getUserTodos` 두가지를 호출하지만 두결과를 모두 사용할 수 있는 경우에만 실제 구성 요소를 렌더링합니다.


다른 방법으로는 순차 fetching을 하용하여 water fall 방식으로 로드할 수 있습니다.

#### 순차 data fetch
```tsx
async function getUser(username) {
  const res = await fetch(`https://api.example.com/user/${username}`);
  return res.json();
}

async function getUserTodos(username) {
  const res = await fetch(`https://api.example.com/user/${username}/todos`);
  return res.json();
}

async function Todos({ userId }) {
  const todos = await getUserTodos(userId);

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.name}</li>
      ))}
    </ul>
  );
}

export default async function Page({ params: { username } }) {
  const user = await getUser(username);

  return (
    <>
      <h1>{user.name}</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <Todos userId={user.id} />
      </Suspense>
    </>
  );
}

```

이렇게 되면 페이지 컴포넌트에서 user를 로드한 다음 `Todos`의 컴포넌트를 렌더링하고 todo data load를 시작합니다. 그 결과 두개의 요청이 차례로 발생하게 됩니다.


Turbopack
---
turbopack을 써서 vite보다 속도가 10배가 빨라졌다고 하네요.
최근에 사이드 프로젝트로 react개발을 할때 vite를 이용했었는데
vite를 통해 빌드할때도 빌드속도가 엄청 빨랐다고 느껴졌는데 이것보다 더 빨라졌다니... 
우선 공식문서상에 올라와있는 정보부터 살펴보죠

```
Webpack보다 700배 빠른 업데이트
Vite보다 10배 빠른 업데이트
Webpack보다 4배 빠른 콜드 스타트
```
라고 규정해놓았는데요.

![터보팩 스피드 비교]({{site.url}}/img/react/next13/turbopack-speed.avif)


아니 vite의 빌드속도도 빨랐는데 이것보다 10배는 더 빠르다니 궁금해서 관련된 게시물들을 좀 찾아봤습니다.

![관련 자료]({{site.url}}/img/react/next13/vitevsturbo.png)
출처: https://github.com/yyx990803/vite-vs-next-turbo-hmr/discussions/8

여기 내용상으로는 nextjs의 turbopack이 10배 빠른 업데이트가 일어나려면 아래의 전제 조건들이 갖추어 져야 한다고 합니다.
```
1. vite가 swc변환을 사용하지 않는다.(vite는 babel기반이지만 turboback은 SWC기반입니다. 비교전제 자체가 잘못됐네요)
2. 응용프로그램에는 3000개 이상의 모듈이 포함이 되어있다.(일반적으로 3000개 이상의 모듈을 잘 사용하지 않죠)
3. 벤치마크는 핫 업데이트된 모듈이 평가되는 시간만 측정하지만 변경 사항이 실제로 적용될 때는 측정하지 않습니다.
```



마치며
---
공식문서를 좀 읽고 제 사견대로 쓴 글이라 오류가 있으시거나 피드백있으시면 댓글써주시기 바랍니다.


출처 : 
https://nextjs.org/blog/next-13
https://www.youtube.com/watch?v=_w0Ikk4JY7U
https://medium.com/nextjs/how-to-use-font-optimizing-in-nextjs-13-7a66c450a88a
https://www.plasmic.app/blog/how-react-server-components-work