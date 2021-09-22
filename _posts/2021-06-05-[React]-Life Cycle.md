---
title: "[React] React Life Cycle"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    React
---


React는 UI를 효율적으로 구축하고 사용하기위한 JS기반의 컴포넌트 라이브러리입니다. 일반적인 DOM이 아닌 Vritual DOM을 위에서 작업을 합니다. 그렇다면 왜 일반 DOM에서 작업하는 것이 아닌 Vritual DOM에서 작업을 하는것일까요?? 일반적인 SPA페이지 에서는 DOM의 직접적인 조작이 많이 발생합니다.(레이아웃의 변화 및 트리변화가 리렌더링을 일으킵니다.) 그렇다는건 브라우저 내에서 연산을 많이 해야되며 프로그램이 효율적이게 돌아가지 않습니다. VirualDOM은 DOM의 변화가 일어나면 먼저 오프라인 DOM트리에 적용을시키고 연산이 끝나고 나면 최종적으로 DOM에 적용시킵니다. 마지막에 변화를 DOM에 적용을시키는거라 수시로OM을 직접 조작해서 변경하는것보단 훨씬 효율적이죠.

React 컴포넌트
===

리액트에서는 일반적인 웹페이지에서

React에서의 가장 중요한 개념은 컴포넌트입니다. 컴포넌트는 props를 인자로 받아 화면에 UI를 그려주는 함수이며 이러한 컴포넌트들을 이용해 React에서는 여러개의 컴포넌트를 독립적인 단위로 잘게 쪼개서 효율적인 UI를 완성할 수 있습니다.

컴포넌트의 UI를 구성하기 위해선 Mounting,Updating,Unmounting의 과정들을 거쳐야합니다.

 ![lifecycle]({{site.url}}/img/react/LifeCycle.jpeg)

Mounting
---
constructor React 컴포넌트의 생성자는 컴포넌트가 마운팅 되기전에 호출됩니다.
static getDerivedStateFromProps 최초 마운트 시와 갱신 시 모두에서 render가 호출되기 직전에 호출됩니다.state를 갱신하기 위한 객체를 반환하거나,null을 반환하여 아무것도 갱신하지 않을 수도 있습니다.
render 클래스 컴포넌트에서 반드시 구현되야하는 메서드입니다. render함수는 순수해야하며,state값을 변경하지 않고 호출될 때마다 동일한 결과를 반환해야합니다.
componentDidMount 컴포넌트 결과물이 DOM에 Mounting된 직후 호출됩니다. 이 메서드는 데이터 구독을 설정하기 좋다.데이터 구독이 이루어 졌다면 componentWillUnmout에서 구독해제 작업을 반드시 수행해야한다.

Updating
---
컴포넌트가 업데이트 되는 경우에는 props,state의 값변경 부모 컴포넌트가 리랜더링 될 때 호출됩니다.

getDerivedStateFromProps 마운트 과정에서 호출되는 함수입니다.
shouldComponentUpdate 컴포넌트를 리랜더딩의 여부를 결정하는 함수입니다. true를 반환하면 아래 함수들을 호출하여 업데이트에 따른 리랜더링을 진행합니다.false를 반환할 경우 리랜더링을 하지 않습니다.
render 새로운 값을 사용하여 View를 리랜더링합니다.
getSnapshotBeforeUpdate 변경된 요소에 대하여 DOM 객체애 반영하기 직전에 호출되는 함수입니다.
ComponentDidUpdate 컴포넌트 업데이트 작업이 끝난 리랜더링 후에 호출되는 함수입니다.

Unmounting
---
컴포넌트가 DOM에서 제거되는 것을 UnMount라고 하며, 호출되는 함수는 하나로 componentWillUnmount함수입니다. 해당 컴포넌트가 제거되기 직전에 호출됩니다.

componentWillUnmount 컴포넌트가 소멸된 시점에 실행되는 메소드입니다. 컴포넌트 내부에서 타이머나 비동기 API를 사용하고 있을때, 이를 제거하기에 유용합니다.

React 컴포넌트 props와 state
===
React는 데이터가 부모 컴포넌트에서 자식 컴포넌트로 데이터 흐름이 이루어지는 단방향 데이터 흐름이 특징입니다. 부모 컴포넌트는 자식 컴포넌트에게 props값을 전달해주며 props값을 받은 자식 컴포넌트는 렌더링하며 ,state라는 자체값을 포함하여 데이터를 변경해주고 다시 랜더링 해줍니다.

props
---
props는 컴포넌트간의 데이터를 전달하고 보관하는데이터입니다. 이 데이터는 수정되지 않고 보존되어야 하며 컴포넌트의 mounting,updating시점에 값이 할당될 뿐 컴포넌트 내부에서 값을 변경할 순 없습니다.

state
---
state는 컴포넌트의 상태를 저장할 수 있습니다. props와 차이점이라면, state는 컴포넌트 내부에 존재하고 있기 때문에, 상태값 변경이 가능하다는 것입니다. 컴포넌트는 두가지 속성인 props와 state를 가지고 있다. props는 컴포넌트의 mounting,updating시점에 값이 할당될 뿐 컴포넌트 내부에서 값을 변경할 수 없습니다.

 ![dataflow]({{site.url}}/img/react/dataflow.png)