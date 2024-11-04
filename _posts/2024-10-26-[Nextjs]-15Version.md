---
title: "Nextjs 15에서의 변경점"
subtitle: "MSW,Cypress"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags: Next.js
---

개요
---

안녕하세요 몇일전 Nextjs Conf에서 Nextjs 15가 공개되었는데요. 
마침 저도 새로운 프로젝트를 진행하게 되어, 이 기회를 빌려 Next.js 15를 도입해 보려 합니다. 
이번 글에서는 Next.js 15의 주요 변경 사항과 새롭게 도입된 기능을 소개하면서, 실무에서 어떻게 적용할 수 있을지 함께 알아보겠습니다.

## 1. React 19와의 호환성과 관련 기능 업데이트 

Next.js 15는 React 19 버전을 지원하기 시작했습니다(아직 rc버전인데 벌써...?!). 이번 Next.js 15의 주요 React와의 관련 사항은 다음과 같습니다.


### 1.1 React 19 지원 및 하위 호환성 유지

Next.js 15는 React 19을 지원하면서도, React 18 Pages Router와의 하위 호환성을 유지합니다. 
React 18과의 호환성 덕분에 기존 프로젝트의 구조를 유지한 채로 최신 업데이트를 쉽게 적용할 수 있습니다.

### 1.2 React Compiler

React에서 사용되던 메모라이징 기능을 구현하는 `useMemo`,`useCallback`의 경우 개발자가 수동으로 메모리 최적화를 적용했었지만
이번 Nextjs 15 버전에서 추가된 React Compiler 기능은 개발자가 `useMemo`나 `useCallback` 같은 훅을 명시적으로 선언하지 않아도 되는 환경을 지원한다고 합니다. 기본값으로 compiler 기능은 꺼져있구요 아래와 같이 활성화 시킬 수 있습니다.

next.config.ts
```ts
const nextConfig = {
 experimental: {
  reactCompiler: true
 }
};
```

적용 전 주의사항:
자동 메모이제이션이 적용되면 개발자가 특정 시점에 의도하지 않은 메모이제이션이 발생할 가능성이 있으며, 특히 과도한 메모이제이션으로 인해 성능 저하가 발생할 우려도 있죠. 특히 특정 라이브러리(ex tanstack)와의 호환성 문제가 보고되기도 하므로, 프로젝트에 미치는 영향을 사전에 검토하는 것이 좋을 듯 하네요.

### 1.3 Hydration 오류 개선
Next.js 14.1부터 Hydration Error 관련 오류 메시지와 힌트를 제공했는데, Next.js 15에서는 이를 더욱 개선했어. Hydration 에러가 발생하면 오류가 발생한 소스 코드와 해결 방법에 대한 제안이 화면에 표시돼, 디버깅이 훨씬 용이해졌습니다.

아래 이미지는 nextjs 14버전에서 나타나는 오류 메시지였습니다
일치하지 않는 ui 요소는 나타내지만 구체적인 에러에 대한 설명은 부족했습니다

![hydration-error-before-light]({{site.url}}/img/react/next15/hydration-error-before-light.avif)


하지만 nextjs 15에서는  서버/클라이언트 조건 분기, Date.now()나 Math.random()과 같은 변수 사용, 
잘못된 HTML 태그 중첩, 외부 데이터 차이 등을 상세히 안내하며, 불일치한 부분을 시각적으로 강조해 줍니다.

![hydration-error-after-light]({{site.url}}/img/react/next15/hydration-error-after-light.avif)


## 2. 보안 강화

Next.js 15에서는 Server Actions의 보안을 강화하여 서버 측 함수가 클라이언트에서 안전하게 호출되도록 여러 개선 사항을 도입했습니다.

*** Dead Code Elimination(사용되지 않는 코드 제거) ***
 사용되지 않는 Server Actions는 클라이언트 번들에 노출되지 않고, 빌드 시 자동 제거되어 성능과 보안을 향상합니다.

*** Secure Action IDs ***
 클라이언트가 Server Actions를 호출할 때, 추측 불가능한 비결정적 ID를 사용하여 접근이 어려워집니다. 이 ID는 빌드마다 새로 생성되어 예기치 않은 엔드포인트 노출을 방지합니다.

 ```ts
// app/actions.js
'use server';

// 사용된 Action -> 보안 ID 생성됨
export async function updateUserAction(formData) {}

// 사용되지 않은 Action -> 빌드 시 자동 제거됨
export async function deleteUserAction(formData) {}
```

적용 전 주의사항:
다만 Next.js 15에서 Server Actions의 보안이 강화되었다곤 하지만, 모든 엔드포인트에 대해 추가적인 보안 조치가 필요해보입니다.
Server Actions 자체가 클라이언트에서 직접 호출되는 특성상, 생성되는 엔드포인트가 외부에 노출될 수 있으므로, 인증, 권한 검증, 데이터 유효성 검사와 같은 보안 설정을 꼭 검토해셔야 되요!!!! 특히 mutations의 경우 Server Actions는 편리하게 사용할 수 있지만 완전한 보안을 보장하지 않기 때문에... 중요한 작업에는 API Routes와 같은 더 안전한 접근 방법을 고려하는 것이 좋습니다.

## 3.캐싱 및 성능 최적화

### 3.1 Async Request APIs

서버사이드 렌더 페이지에서의 비동기 함수 지원입니다.
api를 요청하더라도 동기식으로 진행이 되었지만 이젠 async/await를 지원하여 동기형식으로 api통신을 진행 할수 있게 되었네요

이제 `cookies` , `headers`, `draftMode`, `params`, `searchParams`와 같은 요청 관련 API들도 비동기 함수로 변경되었어요. 이는 SSR의 성능 최적화와 코드의 유연성을 크게 높여줍니다. 예를 들어, 아래와 같이 cookies를 사용할 때 await를 통해 서버가 데이터를 준비하도록 할 수 있습니다

```ts
import { cookies } from 'next/headers';
 
export async function AdminPanel() {
  const cookieStore = await cookies();
  const token = cookieStore.get('token');

}
```

15에서 추가된 이 비동기 작업은 기존 동기 작업과의 호환성을 깨기 때문에 이전버전에서 마이그레이션을 하고자 하는경우 아래와 같은 명령어를 입력하면 쉽게 처리됩니다.
```bash
npx @next/codemod@canary next-async-request-api .
```

### 3.2 캐싱 업데이트

1. GET Route Handlers 기본 캐싱 해제

GET Route Handler의 기본 캐싱이 제거되었습니다. 
이전에는 GET 메서드가 자동으로 캐싱되어 불필요하게 `force-dynamic`을 지정해야 했으나, 이제는 기본적으로 캐싱되지 않으므로 캐싱이 필요한 경우에만 `export const dynamic = 'force-static'`을 사용하면 됩니다.

또한, sitemap.ts, opengraph-image.tsx, icon.tsx와 같은 메타데이터 관련 Route Handler는 여전히 기본적으로 캐싱됩니다. 이번 개선으로 더미 코드가 줄어들고 유연한 캐싱 설정이 가능해졌습니다.

2. 클라이언트 라우터 캐시 기본 캐싱 해제
이전 Next.js 14.2.0에서는 클라이언트 라우터 캐시를 더 세밀하게 조절할 수 있도록 `staleTimes` 플래그가 도입되었는데, Next.js 15에서는 페이지 이동 시 캐시를 기본적으로 `staleTime: 0`으로 설정해, 페이지 컴포넌트가 항상 최신 데이터를 반영하도록 했습니다.

여기선 몇가지 예외 사항이 있습니다
```
* 공유 레이아웃 데이터는 서버에서 다시 가져오지 않아 부분 렌더링을 지원함.
* 뒤로/앞으로 이동 시에는 캐시를 통해 스크롤 위치 등을 복구할 수 있도록 캐시가 유지됨.
* loading.js는 기본적으로 5분간 캐시되고, staleTimes.static 값을 변경하여 이 기간을 조정할 수 있음.
```

next.config.ts
```ts
const nextConfig = {
  experimental: {
    staleTimes: {
      dynamic: 30,
    },
  },
};

```

### 3.3 외부 패키지 번들링 개선

Next.js 15에서 외부 패키지 번들링 최적화를 통해 초기 성능을 개선하는 기능이 추가되었습니다.
App Router와 Pages Router에서 외부 패키지 번들링 설정 방식이 다소 달랐는데, 
이번 업데이트로 두 라우터 간의 설정이 더 일관되게 맞춰졌습니다

App Router에서는 외부 패키지가 기본적으로 번들링됩니다. 
반대로 Pages Router의 경우 외부 패키지가 기본적으로 번들링 되지 않습니다. 다만, transpilePackages 옵션을 사용해 Pages Router에서도 번들링할 패키지를 지정할 수 있습니다. 이를 통해 외부 패키지를 직접 선택하여 번들링할 수 있습니다.

이렇게 따로 번들링 되는것들을 통합하기위해 `bundlePagesRouterDependencies` 기능이 추가되었습니다
`bundlePagesRouterDependencies`를 통해 Page Router에서도 App Router와 동일하게 외부 패키지를 자동으로 번들링하도록 설정 할 수 있습니다.

특정 외부 패키지는 번들에서 제외하고자 할 때 `serverExternalPackages` 옵션을 사용하면 됩니다. 
이 옵션은 App Router와 Pages Router 모두에서 동작하며, 번들에서 제외하고자 하는 패키지 이름을 명시해 성능 최적화와 번들 크기 조정을 세밀하게 관리할 수 있습니다.

next.config.ts
```ts
const nextConfig = {
  // Pages Router에서도 외부 패키지를 자동 번들링
  bundlePagesRouterDependencies: true,
  // 특정 패키지를 번들링에서 제외
  serverExternalPackages: ['package-name'],
  // Pages Router에서 번들링할 패키지 명시
  transpilePackages: ['package-name-to-transpile'],
};
```

## 4. 개발자 경험 개선

### 4.1 self-hosting 개선
Next.js 15에서는 셀프 호스팅 환경에서 캐시 관리와 이미지 최적화를 더 쉽게 할 수 있도록 몇 가지 개선이 이루어졌습니다.

#### expireTime 설정
ISR 페이지의 stale-while-revalidate(백그라운드 업데이트) 기간을 지정하는 expireTime 설정이 새로 도입되었습니다.
기존의 experimental.swrDelta 실험적 옵션을 공식 설정으로 대체하여, next.config.js 파일에서 캐시 만료 시간을 보다 쉽게 제어할 수 있게 되었습니다.

next.config.ts
```ts
//before
const nextConfig = {
  experimental: {
    swrDelta: 0,
  },
};
...
///after
const nextConfig = {
  eexpireTime:0
};

```
#### 기본 만료 시간 업데이트
대부분의 CDN이 stale-while-revalidate 설정을 의도한 대로 적용할 수 있도록 기본 만료 시간이 1년으로 업데이트되었습니다. 
이를 통해 더 안정적이고 예측 가능한 캐시 관리가 가능해졌습니다.

#### 사용자 정의 Cache-Control 지원 강화
Next.js는 기본적으로 페이지를 캐시할 때 기본 Cache-Control 헤더를 적용합니다. 
그러나 사용자가 직접 정의한 Cache-Control 헤더 값이 덮어쓰이지 않도록 개선되었습니다. 이를 통해 특정 페이지의 캐시 기간을 직접 설정할 수 있어요.

예를 들어, A페이지는 5분마다 업데이트되고, B페이지는 하루 동안 캐시되기를 원한다고 가정해본다면, 각 페이지에 Cache-Control 헤더를 설정하여 캐시 기간을 달리 적용할 수 있습니다. 이렇게 하면, 뉴스 페이지는 최신 상태를 유지하고, 가이드 페이지는 더 오래 캐시되므로 서버 부담을 줄일 수 있습니다.

#### 자동 sharp 통합 
 기존에는 이미지 최적화를 위해 sharp 패키지를 별도로 설치해야 했지만, Next.js 15부터는 개발자가 직접 sharp 패키지를 설치하고 구성할 필요가 없으니, 초기 설정이 간단해집니다.

### 4.2 Development and Build Improvements

#### Server Components HMR 캐싱
HMR 캐시는 서버 컴포넌트가 저장될 때마다 API 호출을 재사용합니다. 로컬 개발 중 API 호출 비용과 시간 낭비를 줄여주며 동일한 매개변수로 동일한 API를 호출할 때 캐시된 응답을 재사용하는 방식으로, 만약 요청의 파라미터가 다르다면 새로운 API 호출이 이뤄지게 됩니다. 로컬 개발 중에는 동일한 요청의 반복 호출을 최소화해주죠.

#### Static Generation(정적 생성) 최적화
정적 생성 빌드 속도가 개선되었는데, 페이지를 한 번만 렌더링해 초기 데이터와 HTML을 생성해 기존의 이중 렌더링을 줄여줬습니다.
또한, 동일한 데이터를 사용하는 다른 페이지들 간 캐시를 공유하여 중복된 API 호출을 줄이는 방식으로 빌드 속도를 높이고 네트워크 요청이 많은 페이지에서 유용합니다.

#### Static Generation(정적 생성) 제어 (실험적)
Next.js 15에서는 빌드 과정에서의 세부 설정을 지원하는 고급 옵션이 추가되었습니다. 이 설정은 특정 요구사항이 있는 프로젝트에만 사용하는 것이 좋으며, 과도하게 설정할 경우 메모리 부족이나 리소스 문제가 발생할 수 있습니다. (반복 호출을 방지 할순 있지만... 메모리사용량이 늘어날수도 있어 아직까진 안사용하는게 나을듯 합니다...)

nextConfig.config.ts
```ts
const nextConfig = {
  experimental: {
    // 빌드 실패 시 페이지 생성 재시도 횟수
    staticGenerationRetryCount: 1
    // 워커가 처리할 페이지 수
    staticGenerationMaxConcurrency: 8
    // 워커를 생성하기 위한 최소 페이지 수
    staticGenerationMinPagesPerWorker: 25
  },
}
 
export default nextConfig;
```

### 4.3 기타 부가적인 기능 추가

#### 응답 후 추가 작업을 위한 unstable_after(실험적)

응답 후 추가 작업을 위한 `unstable_after`를 도입했습니다. 
`unstable_after`를 통해 응답 처리 후에도 로그, 분석, 외부 시스템 동기화 같은 비차단 작업을 예약할 수 있으며, 
기존 서버리스 함수에서는 응답 후 바로 연산이 종료되어 후속 작업을 처리하기 어려웠지만, `unstable_after`를 사용하면 사용자가 응답을 받은 후에도 추가 작업을 실행할 수 있습니다.

사용 방법은 `next.config.ts`에 `experimental.after` 옵션을 활성화해주고 원하는 액션에 after함수를 호출하면 됩니다

next.config.ts
```ts
const nextConfig = {
  experimental: {
    after: true,
  },
};
```

unstable_after 함수를 호출하여 서버 컴포넌트, 서버 액션, 라우트 핸들러, 또는 미들웨어 내에서 후속 작업을 추가합니다.
```tsx
// 비차단 로깅 예시
import { unstable_after as after } from 'next/server';
import { logError } from '@/utils';

export default function ErrorBoundary({ children }) {
  after(() => {
    logError();
  });
  return <>{children}</>;
}
```

#### instrumentation.js 도입
instrumentation.js는 서버 라이프사이클 동안 성능 추적과 에러 모니터링을 쉽게 설정할 수 있게 합니다. 프로젝트 루트 또는 src에 instrumentation.js 파일을 추가해 register() 함수로 OpenTelemetry 같은 도구를 초기화하고, onRequestError 훅을 통해 라우터, 렌더링 소스 등의 정보를 포함한 에러를 추적할 수 있습니다.

*** 사용방식(ex.datadog) ***
dd-trace와 같은 외부 모니터링 툴은 serverComponentsExternalPackages 옵션으로 추가할 수 있습니다.

next.config.ts
```ts
// Next.js 15 이후의 설정 예시
const nextConfig = {
  experimental: {
    // instrumentationHook: true, stable버전에 도입되었기 때문에 필요없음
    serverComponentsExternalPackages: ["dd-trace"],
  },
};
```
// instrumentation.js
```js
// Datadog을 이용한 instrumentation.js 설정 예시
const { tracer } = await import("dd-trace");

export async function register() {
    tracer.init({
      logInjection: true,
      service: "nextjs-with-datadog",
    });
}

export async function onRequestError(err, request, context) {
  // 서버 에러 발생 시 자동으로 트레이싱
  tracer.trace('error', { error: err.message, context });
}
```
이 설정을 통해 instrumentation.js에서 서버 성능 추적과 에러 모니터링이 완전히 통합되며, 
onRequestError 훅을 활용해 추가적인 에러 로깅이 필요 없이 자동으로 에러 정보를 Datadog에 전송할 수 있습니다.

#### next.config.ts
이제 next.config.ts를 사용할 수 있게 되었습니다

```ts
import type { NextConfig } from 'next';
 
const nextConfig={};
 
export default nextConfig;
```

#### turbopack 지원 
터보팩이 지원된다고 하네요 로컬에서의 개발 속도가 더 빨라질 듯 합니다. 
webpack의 번들링 속성과 turbopack의 번들링 속성이 크게 차이가 없으면 사용을 고려해봐도 좋겠네요

터보팩 사용방법
```bash
next dev --turbo
```

#### @next/codemodCLI로 최신 버전 마이그레이션
codemod CLI를 통해 stable버전이나 release버전으로 업그레이드 할 수 있습니다.

```bash
npx @next/codemod@canary upgrade latest // 최신버전 업데이트
```

#### eslint 9 지원
Next.js 15는 ESLint 9를 지원하며, ESLint 8의 지원이 2024년 10월 5일에 종료됨에 따라 새 버전으로의 전환을 권장합니다.
 Next.js는 이전 버전과의 하위 호환성을 제공하므로, 현재 ESLint 8 또는 ESLint 9 중 원하는 버전을 선택하여 사용할 수 있습니다.

1. 자동 호환성 유지: ESLint 9로 업그레이드했을 때, 새로운 설정 형식을 아직 적용하지 않은 경우, Next.js가 자동으로 ESLINT_USE_FLAT_CONFIG=false를 설정하여 기존 설정을 유지할 수 있도록 지원합니다.

2. Deprecated 옵션 제거: next lint를 실행할 때 --ext와 --ignore-path와 같은 이전 옵션들이 더 이상 지원되지 않습니다. ESLint 10에서 이러한 설정이 완전히 제거될 예정이므로, 미리 새로운 설정 형식으로 전환하는 것이 좋습니다.

3. eslint-plugin-react-hooks 업데이트: ESLint 9와 함께 eslint-plugin-react-hooks가 버전 5.0.0으로 업그레이드되었습니다. 이 버전에서는 React Hooks 사용에 대한 새로운 규칙이 추가되었습니다. 새로운 규칙은 eslint-plugin-react-hooks 변경 로그에서 확인할 수 있습니다.