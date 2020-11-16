---
title: "Chrom Extension,App 만들기"
subtitle: "가이드"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Chrome Extension
    Javascript
---

설명...
-------
이번에 크롬 브라우저에서 동작하는 확장 프로그램과 앱을 만드는 과정을 설명해보겠습니다.
우선 둘의 차이점으로는 크롬 확장프로그램은 크롬어플리케이션에 종속이 되어있으며 기능도 앱에 비해서 조금 제한적입니다. 반면 크롬앱같은 경우에는 Window App처럼 설치형프로그램처럼 사용할 수 있고 확장프로그램보다 기능을 다양하게 이용할 수 있습니다.


구성요소
------
- ![Image Alt 텍스트]({{site.url}}/img/chromeExtension/extensionlist.png)

- manifest.json
앱의 설명 및 개발시 필요한 사항등을 명시해 놓는 곳이다.

-background.js
백그라운드의 스크립트 처리영역이며 maniefest에서 명시를 해준다.

-window.js,window.html
앱의 Ui및 이벤트를 구성하는 요소로써 


manifest 설정
------------
chrome extension
```cpp
 {
    "manifest_version":2,
    "name": "Getting Started Example",
    "version": "1.0",
    "description": "Build an Extension!",
    "background":{
        "scripts":["script/background.js"],
        "persistent":false
    },
    "browser_action":{}
}
```

chrome app
```cpp
{
    "manifest_version": 2,
    "name": "ChromeApp",
    "version": "2",
    "permissions":["webview",
            "browser"],
    "description": "Test",

    "app":{
        "background":{
            "persistent":false,
            "scripts":["background.js"]
        }
    },
    "minimum_chrome_version":"40"
}
```

background.js 설정
-----------------
 ```cpp
chrome.browserAction.onClicked.addListener(function(){
    chrome.windows.create({
        url:"/html/window.html",
        type:"popup"
    });
});
```

 ```cpp
chrome.app.runtime.onLaunched.addListener(function() {
    chrome.app.window.create("/html/window.html",{
        bounds:{width:300,height:50}
    });
});
 ```


UI 및 이벤트 설정
----------
window.html
 ```cpp
 <!DOCTYPE html>
<html>
    <head>
        <style>
            BODY {width : 300px; min-height:250px;}
        </style>
        <script src="/script/window.js"></script>
    </head>
    <body>
    </body>
</html>
 ```

 window.js
 ```cpp
 function helloWorld(){
    document.body.innerHTML="Hello World";
}
window.onload=helloWorld;
 ```

크롬에 적용
-------
chrome://extensions/로 이동하여 압축해제된 확장 프로그램을 로드합니다 선택 후 
실행시켜 주면 된다.

chrome app 같은 경우 chrome://apps/에서 등록되어진 앱을 실행시켜주면 된다.

참고
---
https://developer.chrome.com/extensions/getstarted