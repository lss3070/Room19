---
title: "[React] React Hook 종류"
subtitle: "기초지식"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:

---

React에서는 16.8버전부터 Hook이라는 기능으로써 함수형 컴포넌트에서도 상태관리를 도와주는 훅,렌더링 후 작업을
설정하는 훅등 기존 함수형 컴포넌트에서 할 수 없었던 다양한 작업들을 할 수 있게 도와주는 기능이다.


useState
---
제일 기본적인 useState부터 알아보죠
useState는 함수 컴포넌트안에서 state를 사용할 수 있게 해줍니다.
useState는 넘겨주는 인자로 state의 초기값을 설정해 줄 수 있으며
state는 클래스와 달리 객체일 필요는 없고, 숫자 타입과 문자 타입을 가질 수 있습니다
useState는 state 변수, 이 변수를 갱신할 수 있는 함수 두가지를 반환합니다.
```js
const App = () => {
   const [count, setCount] = useState(1);
   const increment = () => setCount(count + 1);
   const decrement = () => setCount(count - 1);

   return (
     <div className="App">
       <h1>Hello! {count}</h1>
       <button onClick={increment}>Increment</button>
       <button onClick={decrement}>Decrement</button>
     </div>
   );
 }
```
위와 같이 count를 증가/감소 할 수 있는 함수 컴포넌트를 만들어 봤다.


useEffect
---
useEffect는 컴포넌트가 렌더링 될때와 업데이트 될때
즉, 클래스형 컴포넌트의 componentDidMount, componentDidUpdate와 같은 역할을 한다.
useEffect의 첫번째 인자는 function, 두번째 인자는 dependency다.
dependency란, 두번째 인자로 받은 값이 변할 때 useEffect가 실행되는 것이다.

```js
// 기본적으로 아래와 같이 사용할 경우 컴포넌트 초기 렌더시에 "Hello"가 콘솔에 찍힌다.
const App = () => {
  useEffect(() => {
    console.log("Hello!");
  });
  return (
    <div className="App">
    </div>
  );
```
만약 어떤 특정 값이 변경이 될 때만 호출되게 하고 싶을 때 클래스형 컴포넌트라면 위와 같이 작성 했을 것 이다.
```js
const App = () => {
  const [count, setCount] = useState(0);
  const [number, setNumber] = useState(100);
  useEffect(() => {
    console.log("Hello");
  }, [count]);
  return (
    <div className="App">
      <button onClick={() => setCount(count + 1)}>count증가</button>
      <button onClick={() => setNumber(count + 1)}>number증가</button>
    </div>
  )
};
```
위와 같이 작성할 때 useEffect는 초기 렌더시, 배열 형태로 dependency로 넘겨준 count가 변경될 때만 실행된다.
때문에 number가 변경 될때는 useEffect가 실행되지 않는다. 만약 빈 배열을 넘겨준다면 useEffect는 초기에만 실행되고 어떠한 값을 변경 하더라도 실행되지 않을 것 이다.

useContext
---
useContext를 알기 위해서는 먼저 React의 Context기능에 대해서 알아야 합니다. 해당 기능에 대한 설명은 공식 페이지에 자세하게, 심지어 한글로 적혀있으니 모르겠다 하시는 분들은 먼저 보시는걸 추천합니다. Context, 그것은 React에서 props를 넘기는 과정을 돕기위해 만들어 졌습니다. 저-기 밑에있는 컴포넌트를 위해 자식 컴포넌트마다 props를 넘기는 건 비효율적이니까요. 하지만 그렇게 만들어진 Context조차도 완벽하지는 않았습니다. 함수형 컴포넌트에서 하나의 Context를 위해서는 하나의 Wrapper가 필요한데, 이 말은 결국 많은 Context를 위해서는 많은 Wrapper가 필요해진다는 말입니다. 결국 복잡한 React 프로젝트를 Wrapper Hell로 만들어 버리기 충분한 수준입니다. 이러한 문제를 해결하기 위해서 useContext가 탄생했습니다.

```js
const TestContext = createContext();

const UseContextExample = () => {
  const context = useContext(TestContext);
  return (
    <div>
      {context}
    </div>
  )
}

const App = () =>{
  return(
    <div className="App">
      <TestContext.Provider value='context!!'>
        <UseContextExample/>
      </TestContext.Provider>
    </div>
  )
}

```
useContext를 사용해서 만들어진 TestContext에서 값을 불러오기만 하면 됩니다.
값을 'context!!'을 줬습니다. UseContextExample컴포넌트에서 해당값을 useContext를 통해 가져왔습니다.

useReducer
---
Redux,Reducer에 대한 개념은 통합적으로 state를 관리하기 위해 나왔습니다. 컴포넌트들이 많아지면 많아질수록 컴포넌트들의 부모 자식으로 값을 전달해주는것도 한계가 있고 구현한다고 하더라도 관리가 점점 복잡해질겁니다. useReducer는 함수형 컴포넌트에서 어느정도 Redux를 대체해줍니다.

```js
const reducer = (state,action) => {
  switch(action.type){
    case 'CHANGE':
      return action.value;
    default:
      return state;
  }
}

const App =()=>{
  const [state,dispatch] = useReducer(reducer,'initial');
  
  return(
    <div>
      <h3>{state}</h3>
      <input type="text" value={state} onChange={(e)=>dispatch(value:e.target.value,type:'CHANGE')}>
    </div>
  )
}

```
위 코드는 input안의 값이 바뀌면 state의 값을 변경하는 코드입니다.
reducer함수는 Redux와 마찬가지로 action의 type값에 따라 어떻게 state를 변경할 건지 switch문으로 골라내도록 구현했습니다.

useReducer를 사용하여 state와 해당 state의 값을 변경할 dispatch를 선언하고 있습니다.
useReducer함수의 첫번쨰 인자는 처음에 만들어 놓은 reducer함수를 넘기고, 두번째 인자에는 state의 최초값을 넘기면 됩니다.


input에서의 value는 state 자체를 줘서 input안에서의 값을 유지하고 onChange에 dispatch를 사용하는 함수를 넘겨줬습니다. 이렇게 하면 input안의 값이 변경될 때마다 state 값이 변경되고 변경된 값이 다시 input에 반영되게 됩니다.
여기서 dispatch를 통해 넘겨준 type이나 value는 필수값이 아닙니다.



useMemo
---

React에서는 state의 값이 변경 될 때마다 다시 렌더링이되는 useEffect가 있습니다. 이렇게 렌더링 되는 과정에서 간혹 필요없는 값들을 불러오곤 합니다. 이렇게 되는 경우에는 자원의 낭비로 이어져 커다란 서비스가 만들어 졌을 경우 성능을 저하시키는 요인이 됩니다. 따라서 특정 상황에만 특정 동작을 하도록 유도할 필요가 있습니다.useMemo는 렌더링 될 때가 아닌 특정한 값이 변경 되었을 경우에만 동작합니다.
```js

const App = () =>{

  const [name,setName] = useState('');
  const nameLength = useMemo(()=>name.length,[name])

  const updateName = (e) =>{
    const name = e.target.value;
    setName(name);
  }

  return(
    <div className="App">
      <input onChange={updateName}/>
      <span>{nameLength}</span>
    </div>
  )
}
```
useEffect처럼 2번째 파라미터에 전달한 배열이 이전의 배열과 다르면 1번쨰 파라미터가 발동하며, useState와는 다르게 해당 State를 변경하는 함수를 제공하지 않습니다.



useCallback
---
useCallback은 useMemo와 비슷한개념의 hook입니다.useMemo같은 경우에는 특정한 값이 변경되었을 경우 리렌더링이되는 hook이며 usecallback은 함수가 바뀔경우 리렌더링이 되는 hook입니다.

```js

const App =()=>{
  const [name,setName] = useState('');
  
  const change = useCallback((e) => {
    setName(e.target.value);
  },[]);

  const result = useMemo(()=>sum(),[]);

  return(
    <div className="App">
      <input type="text" onChange={change}/>
      <button onClick={insert}>문자열 추가</button>
      {result}
    </div>
  )
}
```


useRef
---
ref는 특정 element 현상을 발생시키는 역할을 합니다. 예컨데 input의 포커스를 이동하거나 동영상을 재생하거나 하는 동작이 있습니다. useRef는 그런점에 있어서 함수형 컴포넌트에서 ref사용을 편하게 해주는 역할을 합니다.

```js

const App = () => {
  const [currentText,setCurrentText]= useState('');
  const [textList,setTextString]= useState([]);

  const inputText = useRef();

  const add =(useEffect(event)=>{
    const newList = textList.slice();
    newList.push(currentText);
    setTextString(newList);

    setCurrentText('');
    inputText.current.focus();
  },[currentText,textList]);
  
  const change =(useEffect(event)=>{
    setCurrentText(event.target.value);
  },[]);


  return(
    <div>
      <input type='text' ref={inputText} onChange={change} value={currentText}>
      <button onClick={add}>추가</button>
      {textList.length>0&&textList.forEach((item)=>{
        <div>{item}</div>
      })
      };
    </div>
  )
}
```



