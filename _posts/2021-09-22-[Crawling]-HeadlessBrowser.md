---
title: "[Crawling] Playwright를 이용한 크롤링(1)"
subtitle: "Crawling"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:

---

안녕하세요 이직 후에 처음으로 쓰는 포스팅이네요
이것저것 준비할 것도 많고 정신이 없어서 한동안 블로그 글을 못쓰고 있었는데 오늘을 블로그 포스팅할 거리도 생긴김에 이렇게 포스팅해 봅니다^^

포스팅을 2번 나눠서 올릴거라 이번 포스팅은 크롤링 보단 간단하게 headless브라우저를 어떻게 사용하는지만 알려드린다고 보시면 됩니다. 모두들 크롤링은 아실테고 Headless브라우저란 걸 처음 들어보셨을 겁니다. 코드를 보기에 앞서 간단하게 Headless브라우저에 대해서 한 번 알아보죠.


Headless Browser란?
===
Headless Browser는 GUI없이 실행되는 브라우저이며 CLI에서 작동되는 브라우저입니다.
일반적인 브라우저와 같이 웹페이지에 접속하여 Dom tree,css dom tree,렌더링 트리를 만들어 작동을 합니다.
CLI에서 작동되기 때문에 자동화 테스트와 크롤링에 유용하다는 장점이 있습니다.

물론 다른 크롤링 기능에 비해 브라우저를 하나하나 열기 떄문에 느린속도와 많은 양의 메모리를 차지하게 됩니다.

종류도 js기반으로 돌아가는건 [Puppeteer](https://pptr.dev/),[Playwright](https://playwright.dev/),[Cypress](https://www.cypress.io/) 있습니다.
그중에서도 다양한 브라우저에서 지원이 되는 Playwright를 선택하게 되었습니다.


Playwright의 기능
===
* SPA 화면에서의 렌더링

* 사용자가 사용하는 것처럼 키보드 및 마우스를 제어할 수 있다.

* 웹페이지를 자동화 테스트 할 수 있다.

* 웹페이지를 크롤링 할 수 있다.

* 크로스 브라우징을 지원한다.

* 네트워킹 가로채기


창띄우기와 로그인하는걸 테스트해보며 playwright가 어떻게 돌아가는지 알아보죠.

창띄우기 & 기본 개념
===
먼저 playwright를 설치해 줍니다.
```
yarn add playwright
```

src/app.ts
```ts
import { chromium } from 'playwright'


const App = async()=>{
       const browser = await chromium.launch({
        headless:false,
        args: ["--disable-dev-shm-usage"]
    });

    const context = await browser.newContext({});
    const page = await context.newPage();

    await page.goto("https://google.com");

    await page.close();
}
```
저는 chromium를 이용해서 crwaing을 구성해보겠습니다. 취향껏 firefox나 다른 브라우저를 이용해주시면 되겠습니다.

간단하게 주요 개념만 잡아보죠.

browser
---
browser 객체는 실제 브라우저의 인스턴스를 나타냅니다.
브라우저 인스턴스를 만드는 데 비용이 많이 들기 때문에 playwright에서는 단일 인스턴스가 여러 브라우저 컨텍스트를 통해 수행할 수 있는 작업을 극대화하도록 만들어졌습니다.

context
---
컨텍스트란 객체를 새로 생성하여 작업한 것을 볼 수가 있는데요. 브라우저 인스턴스 내에서 분리된 세션이라고 보시면 됩니다. 세션을 여러개 만들어서 브라우저 상태를 분리할 수도 있습니다.

page
---
컨텍스트 내에서는 단일 페이지를 나타낼 수 있습니다.URL로 이동해서 페이지의 콘텐츠와 상호작용할 때 사용하면 됩니다.

 
 위 코드를 실행하면 접속하고자하는 주소가 띄어집니다.
 이런식으로 말이죠.

![one]({{site.url}}/img/crawing/0922_1.gif)


로그인하기
===
그럼 조금 응용해서 outlook을 로그인하는 코드를 작성해보겠습니다.

```ts
const App = () =>{
     const browser = await chromium.launch({
        headless:false,
        args: ["--disable-dev-shm-usage"]
    });

    const context = await browser.newContext({});
    const page = await context.newPage();
    const naviGationPromise = page.waitForNavigation({
        waitUntil:'networkidle'
    });
    await page.setDefaultNavigationTimeout(0);
    await page.goto("https://outlook.live.com/owa/?nlp=1");

    await naviGationPromise;

    await page.waitForSelector('input[type="email"]');
    await page.type('input[type="email"]','이메일');
    await page.click('#idSIButton9');
    await page.waitForTimeout(1000);
    await page.waitForSelector('input[type="password"]');
    await page.type('input[type="password"]', "비밀번호");
    await page.waitForSelector("#idSIButton9");
    await page.click('#idSIButton9');
    await page.waitForSelector('#idSIButton9');
    await page.click('#idSIButton9')
    await page.waitForSelector('span.ms-Button-label');
    
    await naviGationPromise;
}

```

이전에 보다 명령어가 많이 추가되었습니다.막상 보면 별거 아니라서 간단하게 설명만해드리겠습니다.

**page.waitForNavigation**
현재 페이지가 다른페이지로 리다이렉트되기를 기다려주는 메서드입니다.

**page.waitForSelector**
해당되는 element가 로드될 때 까지 기다려줍니다.

**page.type**
주로 값을 넣어줄때 이용하며 사람이 입력하는 것처럼 자연스럽게 입력이 됩니다.

**page.click**
이름 그대로 해당 element의 클릭 이벤트를 발생시켜줍니다.

page.waitForTimeout
밀리초 단위로 대기합니다.

실행시키면 이러한 화면이 나옵니다.

![one]({{site.url}}/img/crawing/0922_1.gif)


마무리
===

다음포스트엔 slack이란 채팅 어플리케이션을 연동해서 채팅만으로 자동로그인을 구축을 해보겠습니다.



참고
===
https://blog.logrocket.com/playwright-vs-puppeteer/
https://ui.toast.com/weekly-pick/ko_20210818