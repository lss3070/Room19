---
title: "Cypress와 MSW 충돌 해결: Intercept 동작시키기"
subtitle: "MSW,Cypress"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
---

개요
---

안녕하세요 이번 포스트에서는 msw와 cypress를 같이 사용하며
cypress의 intercept,waitfor등 api연동 작업이 안되는 문제에 관해 포스팅하겠습니다.
왜 안되는지에 대해 살펴보기에 앞서 두개의 라이브러리가 어떤식으로 동작하는지 알아보겠습니다.

### msw의 동작방식
MSW(Mock Service Worker)는 API 요청을 가상으로 처리해줄 수 있게 하는 도구입니다. 브라우저에서 실행되는 서비스 워커를 이용해 클라이언트 측의 HTTP 요청을 가로채며, 이를 통해 실제 API 서버가 없더라도 API 호출을 테스트할 수 있게 도와주는 도구입니다.

네트워크 계층에서 실제로 요청을 가로채기때문에,실제 API 서버 없이도 클라이언트 코드를 자연스럽게 테스트하거나 개발할 수 있게 도와줍니다.

```ts
import { setupWorker, rest } from 'msw'

const worker = setupWorker(
  rest.get('/api/user', (req, res, ctx) => {
    return res(ctx.json({ name: 'KimChi' }))
  }),
)

worker.start()
```
위와 같은 방식으로 MSW는 클라이언트의 API 요청을 가로채 가상 응답을 반환합니다.
![alt text](image.png)

### cypress의 api 동작 방식
Cypress는 웹 애플리케이션을 위한 e2e나 통합테스트를 하기위 한 테스트도구로써 API 요청을 모니터링하고 제어할 수 있는 다양한 기능을 제공합니다. 그 중 cy.intercept()와 같은 명령어는 네트워크 요청을 가로채고, 특정 요청이나 응답을 조작하거나 대기 상태를 감지하는 데 사용됩니다.


Cypress 기본 API 활용

Cypress에서 cy.intercept()를 사용해 네트워크 요청을 모니터링하는 방식은 다음과 같습니다.

```ts
cy.intercept('GET', '/api/user', { name: 'KimChi2' })
```

문제점 
하지만 MSW와 Cypress를 함께 사용하는 경우, Cypress의 내부 함수인 intercept, waitFor와 같은 기능들이 동작하지 않는 문제를 겪을 수 있습니다. 왜일까요?

Cypress는 네트워크 계층에서 요청을 가로채기 위해 웹소켓을 사용하는 반면 MSW는 HTTP 요청을 가로채기 때문에 두 도구가 서로 다른 레벨에서 동작하는 점에서 충돌이 발생하게 됩니다. 이로 인해 Cypress는 MSW에서 처리된 요청을 감지하지 못하고, intercept나 waitFor가 정상적으로 작동하지 않는 것이죠.

이 문제는 MSW의 GitHub 이슈에서 처음 보고된 바 있습니다.
https://github.com/mswjs/msw/issues/389

## 해결 방법

이를 해결하기 위해, Cypress에서 커스텀하게 명령어를 설정할수 있으며 이 기능을 사용하여 msw를 사용하면서 cypress의 기능들을 사용 할 수 있습니다.예를 들어 interceptRequest라는 커스텀 명령어를 만들어서, MSW에서 처리된 요청을 Cypress의 intercept처럼 처리할 수 있도록 설정할 수 있습니다.

먼저 customResponse 함수를 추가해줍니다req.passthrough()로 요청을 그대로 처리하도록 구성되어 있습니다. 이 구조 덕분에, Cypress와 MSW가 충돌 없이 함께 동작할 수 있게 도와줍니다.
```ts
const interceptHandler = (req: any, res: any, ctx: any, fn: any) => {
  function customResponse(...args: any) {
    const response = res(...args);
    return response;
  }

  if (!fn) return req.passthrough();

  return fn(req, customResponse, ctx);
};
```

그리고 interceptHandler 함수를 추가해줍니다. 이 함수는 MSW와 Cypress 사이에서 API 요청을 어떻게 처리하고 가공할지를 정의해주는 핵심 역할을 합니다.
```ts
Cypress.Commands.add(
  'interceptRequest',
  (type: UppercaseRestMethods | LowercaseRestMethods, route: string, ...args: any) => {
    const method = type?.toLowerCase() as LowercaseRestMethods;

    worker.use(
      rest[method](route, (req: any, res: any, ctx: any) => {
        try {
          return interceptHandler(req, res, ctx, args);
        } catch (err) {
          console.error('에러', err);
        }
      }),
    );
  },
```
위 코드를 통해, MSW가 요청을 처리한 후 Cypress에서 해당 요청을 추적할 수 있게 됩니다. 즉, MSW가 가로챈 요청을 Cypress가 인식하고, 필요한 경우 Cypress의 intercept 메커니즘을 활용해 처리할 수 있게 되는 것입니다.
