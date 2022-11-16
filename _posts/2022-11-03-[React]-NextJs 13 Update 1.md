---
title: "[React] NextJS 13 변경점 (1)"
subtitle: "NextJs"
layout: post
auther: "Hux"
header-style: textx
catalog: true
tags:
    React
    NextJs
---

개요
---

안녕하세요 몇일전에 NextJS가 13버전이 발표가 되었습니다. 
아직까진 정식 버전은 아니고 베타 버전이지만 NextJS에서 몇가지 부분이 바뀌어서 한번 적어보게되엇습니다.

먼저 NexJs 공식 문서상으로는 크게 5가지로 간추려집니다.

* **app/Directory** - layouts,react server components,streaming기능 추가
* **Turbopack** - webpack과 비교해서 700배 빠른 모듈 번들러
* **New next/image** - 지연로딩을 좀 더 빠르게 만들었습니다.
* **New next/font** - layoutshift 현상이 없어진 자체 호스트 글꼴
* **Imporved next/link** - link태그 내부에 a태그 간소화


자 이제 세세하게 알아보죠!

라우팅 시스템의 변화!
---
NextJS 13에서의 가장 큰 변화인거 같습니다. 이전과 같이 파일 시스템 기반의 라우팅을 사용하지만 달라진것이있다면 기존에는 pages폴더를 기반으로 하여 라우팅이 구현 되었지만
nextjs 13에서는 app 디렉터리를 기반으로 해서 바뀐 구조와 다양하게 추가된 예약파일들입니다.

<!-- app 디렉토리는 다음과 같은 기능들을 포함하고 있습니다.

```
Layouts 이동
Server Component 이동 // pages폴더에서는 지원이 안돼지만 app폴더에서는 React Server Component 로드된다.
Streaming 이동
Support for Data Fecthing 이동
``` -->

먼저 이전의 라우팅 폴더 구조에 대해 살펴보시죠

![기존 라우팅]({{site.url}}/img/react/next13/routing-today.avif)

위 그림과 같이 이전 폴더구조에서는 page기반의 폴더 구조였습니다.

업데이트 된 13버전에서는 라우팅되는 최상위 폴더에 app디렉토리가 존재하며 하위단에 폴더별로 라우팅을 정의해서 사용되어집니다.

![바뀐 라우팅]({{site.url}}/img/react/next13/routes.png)

이 바뀐 라우팅 시스템과 더불어 많은 예약파일들이 추가되었습니다.바뀐 라우팅 시스템과 예약파일들을 어떻게 쓰이는지 살펴보죠


새롭게 추가된 예약파일과 라우팅 구성
---

라우팅 되는 폴더에 여러가지 예약파일들이 추가되었습니다.
먼저 기본적으로 구성되는 예약파일들부터 알아보죠

### next/page
**next/page**는 기존의 **next/index**의 기능과 비슷하다고 보시면 될것 같습니다
각 라우팅되는 폴더에 기본적으로 page라고 하는 예약파일을 이용하여 경로의 세그먼트를 정의할 수 있습니다.

![next/page]({{site.url}}/img/react/next13/page.webp)


### next/layout
nextjs 13버전에서는 각 컴포넌트별로 사용할 레이아웃을 설정 할 수 있습니다.
또한 중첩으로도 레이아웃을 설정하거나 폴더별로 영향을 받지않게 다양하게 레이아웃을 설정할 수 있습니다.
동일 세그먼트에 존재하면 url 경로에 영향을 주지 않기 때문에 동일한 부모 레이아웃을 사용하는 경우 리렌더링도 발생하지 않습니다.!!

우선 layout의 종류로는 2가지로 나뉩니다.
```
Root Layout
Nesting Layout
```

#### Root Layout
**RootLayout**는 기존의 _app.js이나 _document.js와 같이 최상위 수준에서 모든 경로에 적용이 되며 **Root Layout**을 사용하면
서버에서 반환된 초기 HTML을 수정할 수 있습니다.
html및 body태그를 포함해야합니다.


```ts
export default function RootLayout({ children }: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

#### Nesting Layout

루트 레이아웃을 제외한 레이아웃들을 나타냅니다.폴더 내부에 정의된 레이아웃들은 해당 폴더 세그먼트에 적용이 됩니다.
파일 계층 구조의 레이아웃은 중첩 children이 되고 즉 자식 레이아웃을 래핑할수 있습니다.
![nesting/layout]({{site.url}}/img/react/next13/nested-layout.webp)

```ts 
export default function DashboardLayout({
  children
}: {
  children: React.ReactNode,
}){
  return <section>{children}</section>;
}
```

위와같이 중첩을 시켜놓으면 (app/layout.js)이 (app/dashboard/layouts.js)를 래핑하고
app/dashboard/* 세그먼트를 내부에 래핑합니다

두 레이아웃은 아래와 같이 중첩이됩니다.
![nested-layout]({{site.url}}/img/react/next13/nested-layouts.png)


Route Groups
---
위에서 살펴보았던 것과 같이 폴더 구로에 따라 url이 매핑이됩니다. 
이렇게 되면 특정 폴더에 맞는 layout을 상속받아 사용할 수 밖에 없게 되는데 이럴 경우를 방지해서 url 구조에 영향을 주지 않는 경로를 구성할 수 있습니다.

몇가지 예시를 들어 살펴보죠

##### 1. 세그먼트에서 URL 경로에 영향을 받지않고 따로 세그먼트를 구성하는 경우.

괄호안에 폴더는 url에서 생략이 되며 (marketing) 경로가 내부에 있고 동일한 URL 구조를 공유하더라도 같은 level에 있는(shop)폴더를 추가하여 다른 레이아웃그룹을 만들 수 있습니다.
ex) (marketing) or (shop)

![route-group-organisation]({{site.url}}/img/react/next13/route-group-organisation.webp)

##### 2.레이아웃에서 특정 세그먼트 선택하는 경우
account,cart 세그먼트에 특정 레이아웃을 설정하고하 한다면 account,cart 폴더 상위에 (shop)이라는 폴더를 만들어 구분 지을 수 있습니다. 그룹외부의 경로인 checkout은 경로를 공유하지 않습니다.

![route-group-opt-in-layouts]({{site.url}}/img/react/next13/route-group-opt-in-layouts.webp)

##### 3.여러가지 root 레이아웃 만들기
여러가지 root layout을 만들려면 최상위의 layout을 제거하고 각경로 그룹안에 파일 layout을 추가하여 줍니다.

![route-group-multiple-root-layouts]({{site.url}}/img/react/next13/route-group-multiple-root-layouts.webp)



### next/template

**next/template**는 레이아웃과 해당 하위 세그먼트들을 래핑한다는 점에서 비슷하지만 여러 경로에서 사용되고 상태를 유지하는 **next/layout**과 달리
template 각 자식에대한 새 인트턴스만을 만듭니다.즉 사용자가 공유하는 세그먼트 경로로 이동을해도 **next/template**를 사용하면 다시 리렌더링이 된다는 말입니다.

상태가 유지가 되지않고 새로운 DOM요소로 다시 생성되기 때문에 아래와 같은 상황이 아니면 쓰지 않는걸 추천합니다.

```
- CSS또는 애니메이션 라이브러리를 사용하여 애니메이션 시작&종료
- useEffect 및 useState에 의존하는 기능
```

또한 **next/layout** 과 **next/template**가 아래와 같이 공존하는 경우에는 **next/layout**안에 next/template가 존재하게 됩니다.

![template]({{site.url}}/img/react/next13/template.webp)

```ts
<Layout>
  <Template>{children}</Template>
</Layout>
```

### next/head
디렉토리에서 HTML요소를 수정할 수 있습니다.

![head-file]({{site.url}}/img/react/next13/head-file.webp)

```ts
async function getPost(slug) {
  const res = await fetch('...');
  return res.json();
}

export default async function Head({ params }) {
  const post = await getPost(params.slug);

  return (
    <>
      <title>{post.title}</title>
    </>
  )
}
```

**next/head**는 특정 태그만 반환할 수 있습니다.

`<title>`
`<meta>`
`<link>`
`<script>`


### next/loading

next13버전에서는 **Instant Loading States**라고하는 새로운 규칙이 도입되었습니다.
**Instant Loading States**를 사용하면 스켈레톤 및 스피너 같은 로딩 화면을 미리 렌더링 할 수 있습니다.

폴더안에 loading.js를 추가하여 사용합니다.
![loading-folder]({{site.url}}/img/react/next13/loading-folder.webp)

app/dashboard/loading.tsx
```
export default function Loading() {
  // You can add any UI inside Loading, including a Skeleton.
  return <LoadingSkeleton />
}
```
더욱 세분화 된 로딩을 구현하고자 하면 react18에 나온 Suspense를 이용하면 될것 같습니다.

### next/error
error.js는 Next.js 13은 어플리케이션의 오류를 처리하는데 도움이 되는 새로운 파일 규칙들을 도입했습니다.
React Error Boundaries를 기반으로 하는 이 규칙을 사용하면 하위 트리 내에서 오류가 발생하는 경우 대체 화면을 표시 할 수 있습니다.

#### Error Boundaries

error.js 경로 세그먼트와 그 아래의 자식에 대한 오류 경계를 정의합니댜.특정 오류 정보와 오류 복구를 시도하는 기능을 표시하는데 사용할 수 있습니다.

폴더안에 error.js를 추가하여 사용합니다.
![error-folder]({{site.url}}/img/react/next13/error-folder.webp)

```ts
'use client';

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error);
  }, [error]);

  return (
    <div>
      <p>Something went wrong!</p>
      <button onClick={() => reset()}>Reset error boundary</button>
    </div>
  );
}
```

Error Boundaries는 Client Component여야 합니다.
동일한 폴더에서 및 error.js내부에 중첩됩니다 (있는 경우). 파일과 그 아래의 모든 자식을 오류 경계로 래핑 하지만 동일한 수준의 레이아웃이나 템플릿은 래핑하지 않습니다.`layout.js` `template.js` `page.js`
![error-diagram]({{site.url}}/img/react/next13/error-diagram.webp)


next/font와 next/image의 변화
---
둘의 공통적인 추가사항으로는 기존 로딩퍼포먼스를 저해하던 요소들을 개선하여 
**LayoutShift**가 방지가 되었다는것이다. 
**Layoutshift**란 이미지나 폰트같은 컨텐츠가 느리게 로딩이 되어 화면 레이아웃이 순간적으로 밀리게 되는 현상을 말합니다.


**next/image**같은 경우에는 이전에는 이미지 파일이 크거나 이럴경우에는 로드되는 시간이 느린데
이미지가 화면상에 그려지지않고 로드 되는순간에는 이미지가 자동으로 최적화가되어서 이미지가 줄어들었다가 화면상에
이미지가 로드되었을 때 다시 밀려내려와서 **Layoutshift**현상이 발생했다 그래서 이걸 방지하기 위해 width와 height값을 고정으로 설정을 해 두었지만 nextJS의 13버전에서는 이러한 부분들이 해결이 되었다.

**next/font**는 nextjs자체적으로 google font가 내장이 되어 이제 cdn링크가 필요없어지게 되었다.
이전에는 _document.js 같이 cdn링크를 사용하여 google font를 가져왔습니다.

```ts
<Head>
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="true" />
<link href="https://fonts.googleapis.com/css2?family=Anek+Malayalam:wght@200;300&family=Montserrat:wght@300;400&display=swap" rel="stylesheet" />
</Head>

```
하지만 아직까진 nextjs에 내장이 되어있는것이 아닌  @next/font를 다운 받아야 하며 그래야 구글 폰트가 적용이 됩니다.
```
npm install @next/font
or
yarn add @next/font
```
예를 들어 Ubuntu라고 하는 폰트를 가져오고 싶다면 아래와 같이 폰트를 가져와 적용을 시켜준면 된다.

```ts
import { Ubuntu} from '@next/font/google'
const ubuntu = Ubuntu({weight: '400'})

export default function Home() {
return (<h1 className={ubuntu.className}> Home page </h1> );
}

```


next/link
===
link태그 같은 경우에는 이전에 link태그 안에 a태그를 따로 설정을 해주어야 했지만
이번 13버전에서는 link태그 안에 바로 문자를 입력하여 a태그가 필요없게 되었습니다.
이제 기본태그에도 prop값을 전달 할 수 있게 되었습니다.

```ts
export default const Test=()=>{
  return(
    <>
      <link href='/test2'>Test이동</link>
    </>
  )
}
```

내용이 길어져서 Server Component와 fetch는 다음 게시물에 작성하도록 하겠습니다.


출처 : https://nextjs.org/blog/next-13
https://www.youtube.com/watch?v=_w0Ikk4JY7U
https://medium.com/nextjs/how-to-use-font-optimizing-in-nextjs-13-7a66c450a88a