---
title: "[Crawling] Playwright를 이용한 크롤링_2"
subtitle: "Crawling"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    HeadlessBrowser
---


이번 포스팅은 slack이란 채팅앱을 이용해서 크롤링을 연동해보려고 합니다.



slack과 nodejs연동
===
먼저 코드를 살펴보기에 앞서 slack과 node js를 연결시켜보겠습니다.
https://api.slack.com/에 들어가셔 먼저 app을 만들어주시기 바랍니다.

![one]({{site.url}}/img/crawing/1004_1.png)
상단에 From scratch를 선택해줍니다.

워크스페이스와 App이름을 설정해줍니다.
![one]({{site.url}}/img/crawing/1004_2.png)



![one]({{site.url}}/img/crawing/1004_3.png)
자 자신의 워크스페이스 내에 App이 하나가 생성이 되었습니다.
그냥은 바로 연동은 안되고 api 설정창에서 몇개를 설정해줘야 됩니다.
하단부 이미지에 Permission과 Event Subscriptions을 설정해줘야합니다.
먼저 Permsiion을 들어가서 설정해주죠.

쭉 내리시다보면 Bot Token Scopes라는 탭이 있을겁니다 저희 거기서
Add an OAuthScope를 선택해주고 im:history라는 scope를 선택해줍니다.
![one]({{site.url}}/img/crawing/1004_4.png)
그러고 그냥 나가면 설정이 안되기 때문에 OAuth Tokens for Your Workspace탭에서
ㅑ
install to worksapce를 클릭해서 저장해줍니다.
![one]({{site.url}}/img/crawing/1004_5.png)

그런다음 Evnet Subscriptions탭에 들어가서
Enable Evnets를 활성화 시켜주시고
![one]({{site.url}}/img/crawing/1004_6.png)
Subscribe to bot events탭에서 message.im이란 이벤트를 추가해주시고 저장시켜줍니다.
![one]({{site.url}}/img/crawing/1004_7.png)


apptoken에 관한것..

![one]({{site.url}}/img/crawing/1004_8.png)

소켓모드 활성화..\
![one]({{site.url}}/img/crawing/1004_9.png)


api에서 설정해줘야 되는건 이제 끝났습니다.
이제 slack bot을 이용하여 연동을시켜보죠

```ts
import { App } from '@slack/bolt';

  const token =
  'Bot User OAuth Token';
const signingSecret='Signing Secret'

const appToken =
  'app Level Toekns';

const app: App = new App({
    token,
    signingSecret,
    socketMode:true,
    appToken
})

 (async () => {
    await app.start();
})();

app.message('test',async ({message,say,event,payload})=>{
    await say('입력 테스트 완료');
});

```
키값들을 알맞게 입력해주시고 앱을 실행시켜 보겠습니다.
위 기능은 특정 메시지를 입력하면 slack bot에서 특정 문자를 인식하여 메서드를 실행시키는 기능입니다.
app.message를 이용해서 메세지를 인식하고 
내부에서 say메서드를 활용해서 하고하자는 말이나 아니면 메서드를 넣어 기능을 실행시킵니다.

![one]({{site.url}}/img/crawing/1004_10.gif)

slack과 nodejs를 연동시키는 것까지 알아보았습니다.
이제부터 해볼건 nomard란 메세지를 입력하면 nomard코더 사이트에서 로그인 링크를 가져오는 기능을 수행할 것입니다.
저는 Office의 메일기능을 이용할 것이고 여러분들이 사용하시는 메일 계정에 따라 맞춰서 작업해주시면 되겠습니다.


Nomard 로그인 크롤링
===

```ts
const nomardLogin = async ()=>{
    try{
        const browser = await chromium.launch({
            headless:false,
            args: ["--disable-dev-shm-usage"]
        });
        const context = await browser.newContext({});
        const page = await context.newPage();
        const navigationPromise = page.waitForNavigation({
            waitUntil: "networkidle",
        });
        await page.setDefaultNavigationTimeout(0);
        await page.goto(
            "https://nomadcoders.co/login"
        );
    
        await navigationPromise;
        await page.waitForSelector('input[type="email"]');
        await page.type('input[type="email"]', "study@timmanage.com");
        await page.click("button[type=submit]");
        await navigationPromise;
    }catch(error){
        console.log(error);
    }
}
```



Office mail 크롤링
===
```ts
const mailGet = async ()=>{
    
    const browser = await chromium.launch({
        headless:false,
        args: ["--disable-dev-shm-usage"]
    });
    const context = await browser.newContext({});
    const page = await context.newPage();
    const navigationPromise = page.waitForNavigation({
        waitUntil: "networkidle",
    });
    await page.setDefaultNavigationTimeout(0);
    await page.goto(
        "https://outlook.live.com/owa/?nlp=1"
    );
    await navigationPromise;
    await page.waitForTimeout(5000);
    await page.waitForSelector('input[type="email"]');
    await page.type('input[type="email"]', "이메일");
    await page.click("#idSIButton9");
    await page.waitForTimeout(3000);
    await page.waitForSelector('input[type="password"]');
    await page.type('input[type="password"]', "비밀번호");
    await page.waitForSelector("#idSIButton9");
    await page.click("#idSIButton9");
    await page.waitForSelector('#idSIButton9');
    await page.click('#idSIButton9')
    await page.waitForSelector('span.ms-Button-label');
    await page.waitForTimeout(3000);
    await page.evaluate(()=>{

        const preTime = new Date();
        const span= document.getElementsByTagName('span');

        for(let item of [].slice.call(span) as any){
            if(item.textContent=="Nomad Coders Login"){
                let timeSpan = item.parentElement.parentElement.lastChild.title;
                if((timeSpan.includes("오후")||timeSpan.includes("오전"))){
                    let hour = timeSpan.includes("오후")?
                    timeSpan.slice(timeSpan.indexOf("오후")+3,timeSpan.indexOf(":"))+12:
                    timeSpan.slice(timeSpan.indexOf("오후")+3,timeSpan.indexOf(":"))
                    let min = timeSpan.slice(timeSpan.indexOf(":")+1,timeSpan.length);
                    let curTime = new Date(preTime.getFullYear(),preTime.getMonth(),preTime.getDate(),hour,min,0);
                    if(+curTime - +preTime>0){
                        item.closest('div[role="option"]').click();
                        break;
                    }
                }
            }
        }
    });
    await page.waitForSelector('div.wide-content-host');
    const link = await page.evaluate(()=>{
        const plist = document.getElementsByTagName('p');
        for(let item of plist as any){
            if(item.textContent.includes('https://')){
                return item.textContent;
            }
        }
    });
    await navigationPromise;
    return link;
}
```

```ts
interface MailGetOutput{
    state:boolean
    error?:string
    link?:string
}

app.message('test',async ({message,say,event,payload})=>{


    const result = await doing();
    result.state? await say(result.link!) : await say(result.error!);
});

(async () => {
    await app.start();
})();

const doing = async ():Promise<MailGetOutput>=>{
    try{
        await nomardLogin();
        const link = await mailGet();
        return{
            state:true,
            link
        }
    }catch(error: any){
        return {
            state:false,
            error,
        };
    }
}

```

자 이렇게 nomard로그인 기능을 구축해보았습니다.
nomard에 로그인을 하게 되면 로그인한 개인 계정 메일에 이메일에 갑니다.
그럼 그 이메일을 확인하는 크롤링 기능을 한번 살펴보죠.



자 이제 로그인하여 nomard 로그인 주소를 가져오게 됩니다.


```ts
app.message('nomard',async ({message,say,event,payload})=>{
    await say('접속중...');

    const result = await doing();
    result.state? await say(result.link!) : await say(result.error!);
});

(async () => {
    await app.start();
})();

const doing = async():Promise<MailGetOutput>=>{
    try{
        await nomardLogin();
        const link = await mailGet();
        return{
            state:true,
            link
        }
    }catch(error: any){
        return{
            state:false,
            error
        }
    }
}
```

위에 코드들을 붙여 실행시키면 아래 GIF화면처럼 실행이 됩니다.

![three]({{site.url}}/img/crawing/1004_10.gif)







