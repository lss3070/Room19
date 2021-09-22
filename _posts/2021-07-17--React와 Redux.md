---
title: "[React] React와 Redux"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
    React
---

들어가며...
===

React프로젝트의 관계성이 깊어질수록 자식으로 넘겨주어야하는 props도 마찬가지로 점점 깊어집니다. 깊어지면 깊어질수록 state를 관리하기가 힘들어 내가 언제 어디서든 원하는 state를 사용할 수 있는 redux가 나왔다.

Redux는 React에 종속된 라이브러리가 아닙니다.Js에서 State를 관리하기 위해 등장한 라이브러리입니다.


세팅
===
```cmd
$ create-react-app redux-example
$ cd redux-example
$ npm i redux react-redux
```

src안에 내용들은 아래와 같이 바꿔줍니다.

 ![rootpath]({{site.url}}/img/react/redux/rootPath.png)

src/index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import { createStore } from 'redux';
import reducers from './reducers';
import { Provider } from 'react-redux';

const store = createStore(reducers);

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>
    ,
    document.getElementById('root')
)

```



src/components/app.js

```js
import React,{Component} from 'react';

class App extends Component{
    render(){
        return(
        )
    }
}
export default App;

```

