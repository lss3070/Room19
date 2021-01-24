---
title: "[React]create-react-app 오류"
subtitle: "오류"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    React
    Javascript
---

이틀전에 React프로젝트 초기 셋팅할 때 애먹었던 오류이다.
react-create-app을 이용하여 React를 셋팅하고 실행하려는 순간... 아래 문구처럼 에러가 떠 프로젝트 초기 실행을 할 수 없었다...

오류명 
----
```Shell
If you would prefer to ignore this check, add SKIP_PREFLIGHT_CHECK=true to an .env file in your project.
That will permanently disable this message but you might encounter other issues.
```

해결
---
내 경우에는 예전에 전역에 설치해놓았던 webpack,babel의 버전이 맞지않아 생기는 문제여서 전역폴더에 설치되어 있던 webpack과 babel을 삭제시키고 지역폴더에 webpack과 babel을 버전을 다시 맞춰서 설치하니 실행이 잘됐었다!!

