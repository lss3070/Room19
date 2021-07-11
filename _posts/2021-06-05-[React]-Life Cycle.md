---
title: "[React] React Life Cycle"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:

---

라이프 사이클의 종류는 9가지가 존재한다.
크게 세가지 유형으로 나눌 수 있는데,생성될 때,업데이트할 때, 제거할 떄 여기서 마운트는 DOM이 생성되고 웹 브라우저상에서 나타나는 것을 뜻하고 반대로 언마운트는 DOM에서 제거 되는 것을 뜻한다.
![LifeCycle]({{site.url}}/img/react/LifeCycle.jpeg)


counstructor
---
생성자는 처음 컴포넌트를 만들때 처음으로 실행되는것이며 이메서드에서 초기 state를 설정 할 수 있다.
```js
class Example extends React.Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
}
```
getDerivedStateFromProps
---
이 메서드는 props로 받아온 값을 state에 동기화시키는 용도로 사용하며, 컴포넌트가 마운트 될 떄와 업데이트 될 떄 호출된다.
```js
class Example extends React.Component {
  static getDerivedStateFromProps(nextProps, prevState) {
    if (nextProps.value !== prevState.value) {
      return { value: nextProps.value }
    }
    return null
  }
}
```

sholudComponentUpdate
---
이 메서드는 props나 state를 변경했을 때, 리렌더링을 할지 말지 결정해주는 메서드이다. 이 메서드에서는 반드시 true나 false를 반환해야한다.이 메서드는 오직 성능 최적화만을 위한 것이며 렌더링 목적을 방지하는 목적으로 사용하게 된다면 버그로 이어질 수 있다.
```js
class Example extends React.Component {
  shouldComponentUpdate(nextProps) {
    return nextProps.value !== this.props.value
  }
}
```
render
---
이는 가장 기초적인 메서드이기도하고 가장 중요한 메서드이기도 하다. 컴포넌트를 렌더링할 때 필요한 메서드로 유일한 필수 메서드이기도 하다. 함수형 컴포넌트에서는 render를 안쓰고 컴포넌트를 렌더링할 수 있다.
```js
class Example extends React.Component {
  render() {
    return <div>컴포넌트</div>
  }
}
```
getSnapshotVeforeUpdate
---
이 메서드는 render에서 만들어진 결과가 브라우저에 실제로 반영되기 직전에 호출된다. 공식문서의 말을 따보자면 이 메서드에 대한 사용 예는 흔하지 않지만, 채팅 화면처럼 스크롤 위치를 따로 처리하는 작업이 필요한 UI 등을 생각해볼 수 있다고한다.

```js
class Example extends React.Component {
  getSnapshotBeforeUpdate(prevProps, prevState) {
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current
      return list.scrollHeight - list.scrollTop
    }
    return null
  }
}
```

componentDidMount
---
이 메서드는 컴포넌트를 만들고 첫 렌더링을 마친 후 실행한다. 
```js
class Example extends React.Component {
    componentDidMount() {
        ...
    }
}
```

componentDidUpdate
---
이것은 리렌더링을 완료한 후 실행한다. 업데이트가 끝난 직후이므로, DOM관련 처리를 해도 무방하다.
```js
class Example extends React.Component {
    componentDidUpdate(prevProps, prevState) {
        ...
    }
}
```
componentWillUnmount
---
이 메서드는 컴포넌트를 DOM에서 제거할 때 실행한다. componentDidMount에서 등록한 이벤트가 있다면 여기서 제거 작업을 해야한다. 
```js
class Example extends React.Component {
    coomponentWillUnmount() {
        ...
    }
}
```
componentDidCatch
---
마지막으로 맨 위의 사진에는 보이지 않지만 componentDidCatch라는 메서드가 존재한다. 이 메서드는 컴포넌트 렌더링 도중에 에러가 발생 했을 때 애플리케이션이 멈추지 않고 오류 UI를 보여줄 수 있게 해준다.
```js
class Example extends React.Component {
  componentDidCatch(error, info) {
    console.log(error)
  }
}
```

