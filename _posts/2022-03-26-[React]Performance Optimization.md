---
title: "[React] React에서의 성능개선에 관한 고찰 "
subtitle: ""
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    React
---


안녕하세요 이번 포스팅에서는 React에서의 메모이제이션에 대해서 알아보겠습니다.

Memoization이란?
===
메모이제이션(memoization)은 컴퓨터 프로그램이 동일한 계산을 반복해야 할 때, 이전에 계산한 값을 메모리에 저장함으로써 동일한 계산의 반복 수행을 제거하여 프로그램 실행 속도를 빠르게 하는 기술이다. 동적 계획법의 핵심이 되는 기술이다. 메모이제이션이라고도 한다.

출처:https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EC%9D%B4%EC%A0%9C%EC%9D%B4%EC%85%98

저는 React를 활용하면서 메모이제이션에 대한 개념을 접했습니다.
좀 더 나은 렌더링 좀 더 나은 효율성을 위해서 찾아보던 도중에 메모이제이션이란 개념이란게 있었고
React에서는 useMemo,useCallback,React.memo이 세가지 형식으로 메모이제이션의 기능이 지원이 됩니다.



React에서의 Memoization
===
먼저 useMemo,useCallback,React.Memo에 대해서 간단하게만 살펴보고 넘어가보죠

useMemo
---
useMemo은 메모이제이션된 값을 반환한다.

```ts
const Component=()=>{
    const memoizedValue = useMemo(()=>,[a,b])
    return (
        <>
        </>
    )
}
```
useCallback
---
useMemo의 함수형 버전이며 메모이제이션돈 콜백을 반환한다.


React.Memo
---

React에서 컴포넌트는 렌더링을 한 뒤에 이전에 렌더링 된 결과와 비교하여 바뀐 부분이 있는 경우 DOM을 업데이트를합니다.

//PureComponent의 기능과 같이 수행된다.

간단한 구조를 한번 살펴보져ㅛ
Parent
-Child

React.memo를 이용해 Parent의 값이 변할때 새로 변경된 Child만 렌더링하고 기존에 렌더링된
Child들은 렌더링이 되지 않도록 만들어보죠.

Parent
```ts
export const Parent=()=>{
    const [children,setChildren]=useState([
        {
            name:"Kim"
        },
        {
            name:"Lee"
        }
    ])
    const addChild=()=>{
        setUser([...children,{
            name:"Park"
        }])
    }
    return(
        <>
            <button
            value="추가"
            onCkick={addChild}
            />
            {
                children.map((child,index)=>
                <Child key={index} child={child}>
                )
            }
        </Child>
    )
}
```

Child
```ts
export const Child=React.memo({child})=>{
    <>
        <div>{name}</div>
    </>
} 
```

버튼을 눌러 Children을 변화시켜
Parent를 리렌더링 시키더라도 새로 추가된 children만 새로 렌더링되고 이미 렌더링된
children은 리렌더링 되지 않습니다.

---
React.memo가 필요한 경우
state나 props의 데이터가 변경되지 않고 컴포넌트를 자주 업데이트 하지 않아도 될때
React.memo가 불필요한 경우