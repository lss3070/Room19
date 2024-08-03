---
title: "[React] React 19버전 톫아보기"
subtitle: "React"
layout: post
auther: "Hux"
header-style: text
catalog: true
tags:
    Javascript
---

개요
---

안녕하세요 react 19 rc버전이 나온지도 어느덧 3달정도 흘렀네요.
처음엔 나왔을땐 회사일이 바빠 대강 쓱 훑고 넘어갔었는데 
내용 정리 겸 19버전을 직접 사용해보며 해당 기능별로 정리해보았습니다.


## Form을 처리하는 방식의 변화
React 19에서는 다양한 기능들이 추가되었는데요 먼저 폼의 사용방식의 변화와 훅내부에서 비동기함수를 사용하는 방식에 대해 살펴보겠습니다.
폼은 이제 actions이란 기능을 통해 관리할 수 있게되었습니다 이는 관리하는 state의 양이 줄어 코드의 가독성도 높아지고 state관리도 용이하게되었습니다.

먼저 actions을 이해하기 위해 이전에 폼을 관리하는 방식을 먼저 살펴보겠습니다
react18및 이전 버전에서는 `handleSubmit`이라는 버튼의 함수를 사용하여 폼을 제출하였습니다.
아래예제를 살펴보며 알아보죠

**기존방식**
```TypeScript
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
   event.preventDefault();
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    } 
    redirect("/path");
  };
  
  return (
    <form>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </form>
  );
```
 위 코드는 버튼을 클릭하였을 때 이름을 바꿔주고 `/path`주소로 이동시켜주는 코드입니다. 
 여기서 `updateName`함수를 처리할 때 `isPending`상태를 직접관리하고 에러가 발생하면 상태를 업데이트하며, 
 성공시 페이지를 리디렉션합니다. <br/>
 이러한 상태 변경점이 많은 경우 코드의 복잡성이 증가하며 가독성이 안좋아지는 단점이 존재합니다.


**useTransition을 통해 비동기 처리하는 방법**

위 코드의 단점을 보완하기 위해 React 19버전에선 useTransition과 같은 훅에서 비동기 함수가 지원이 되었으며, 그로 인해 pending,error 폼의 상태를 자동으로 처리할 수 있게 되었습니다.
이는 복잡한 비동기 로직을 단순화하고, 상태 관리의 복잡성을 줄이는 데 큰 도움을 줍니다.
 ```typescript
 const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      } 
      redirect("/path");
    })
  };

  return (
    <form>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </form>
    );
```
이 코드를 보면, useTransition 훅을 사용해 비동기 작업을 시작하는 부분을 startTransition 함수로 감쌌습니다. 
이렇게 하면, 비동기 작업이 시작될 때 isPending 상태가 자동으로 `true`로 설정되고,
 비동기 요청이 완료되면 isPending 상태가 false로 전환됩니다. 
 이를 통해, isPending에 대한 상태의 관리가 쉬워지고 부수적인 errorstate만 관리를 해주면 됩니다.

**action의 도입부분이 약함**!!!!!

자 action의 기능으로 넘어가면 action은 form에서의 onChange 핸들러를 따로 정의하지 않고 사용할 수 있다는 장점이 있습니다
```typescript
const [name, setName] = useState("");
  const [error, setError] = useState(null);

    const handleSubmit = async (formData) => {
    setIsPending(true);
    setError(null);

    const name = formData.get("name");

    try {
       const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      } 
      redirect("/path");
      }
  };

  return (
    <form action={handleSubmit}>
      <input value={name}/>
      <button type="submit" disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </form>
    );
```
form의 action파라미터에 함수를 넣고 그 함수에서 일괄적으로 처리가 가능합니다. 또한 useActionState를 사용하게 되면 state도 한번에 관리가 가능합니다.


## useActionState의 사용

`useActionState` 훅은 React에서 폼 데이터를 관리하는 보일러플레이트 코드를 크게 줄여줍니다. 
actions을 사용하기 전에는 각 입력 필드에 대한 상태를 정의하고, `onChange` 핸들러를 사용해 상태를 업데이트했습니다. 
하지만 `useActionState`를 사용하면 이러한 과정이 단일 함수 호출로 간소화되어 코드가 더 깔끔하고 유지보수하기 쉬워집니다.

```typescript
 const [user, submitAction, isPending] = useActionState(login, {
    error: null,
    data: null,
  });

  async function login(previousState, formData) {
    const username = formData.get("username");
    const password = formData.get("password");
    try {
      const response = await loginUser(username, password);
      return { error: null, data: response.data };
    } catch (error) {
      return { ...previousState, error: error.error };
    }
  }

  return (
    <form action={submitAction}>
      <input value={name}/>
      <button type="submit" disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </form>
    );
```

## useFormStatus
useFormStatus는 이름에서 알 수 있듯이 form의 state를 가져오는 훅입니다. form을 제출할 때 useFormStatus의 상태를 확인 할 수 있습니다.
이것도 actions에서의 비동기가 지원이되면서 추가가된 hook입니다.

```typescript
import React from 'react';
import { useFormStatus } from 'react';

const FormComponent = () => {
  const { isLoading, isSuccess, isError, setStatus } = useFormStatus();

  const handleSubmit = async (event) => {
    event.preventDefault();
    setStatus({ isLoading: true, isSuccess: false, isError: false });
    try {
      const response = await fetch('/api/submit', {
        method: 'POST',
        body: JSON.stringify({/* form data */}),
        headers: { 'Content-Type': 'application/json' },
      });
      if (response.ok) {
        setStatus({ isLoading: false, isSuccess: true, isError: false });
      } else {
        throw new Error('Submission failed');
      }
    } catch (error) {
      setStatus({ isLoading: false, isSuccess: false, isError: true });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" name="example" />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Submitting...' : 'Submit'}
      </button>
      {isSuccess && <p>Form submitted successfully!</p>}
      {isError && <p>There was an error submitting the form.</p>}
    </form>
  );
};

export default FormComponent;
```

## useOptimitic
useOptimitic는 낙관적 업데이트를 지원하는 hook입니다. 
낙관적 UI 업데이트는 서버 응답을 기다리지 않고 사용자에게 즉시 피드백을 제공하는 방식입니다. 
이는 특히 네트워크 지연 시간이 긴 환경에서 사용자 경험을 개선하는 데 유용합니다.

아래예시를 살펴보죠

```typescript
import React, { useState } from 'react';
import { useOptimistic } from 'react';

const LikeButton = ({ postId }) => {
  // useOptimistic 훅을 사용하여 상태를 초기화합니다.
  const [likes, setLikes] = useState(0);
  const [optimisticLikes, setOptimisticLikes] = useOptimistic(likes);

  const handleLike = async () => {
    // 사용자가 좋아요 버튼을 클릭하면, 낙관적으로 상태를 업데이트합니다.
    setOptimisticLikes((prevLikes) => prevLikes + 1);

    try {
      // 서버에 좋아요 요청을 보냅니다.
      const response = await fetch(`/api/like/${postId}`, {
        method: 'POST',
      });

      if (!response.ok) {
        throw new Error('Network response was not ok');
      }

      // 서버 응답이 성공적이면 상태를 업데이트합니다.
      const result = await response.json();
      setLikes(result.likes);
    } catch (error) {
      // 서버 요청이 실패하면, 낙관적 업데이트를 취소합니다.
      setOptimisticLikes((prevLikes) => prevLikes - 1);
      console.error('Error liking the post:', error);
    }
  };

  return (
    <div>
      <button onClick={handleLike}>
        Like ({optimisticLikes})
      </button>
    </div>
  );
};

export default LikeButton;
```
위 코드를 살펴보면 굳이 useOptimistic를 쓸필요가 없다는걸 바로 느끼실 수 있을겁니다.
useOptimistic아니더라도 useState를 통해 state를 업데이트하면되니 말이죠 이 둘의 차이점을 살펴보죠

#### useState를 사용할 때의 문제
useState를 사용할 때, 상태 업데이트는 비동기적으로 처리되기 때문에 함수 내에서 상태를 업데이트하고 다시 참조하려고 하면, 
이전 상태를 참조할 수 있습니다. 이로 인해 비동기 작업이 실패할 때 상태 롤백이 예상대로 동작하지 않을 수 있습니다.
```typescript
const handleLike = async () => {
    const originalLikes = likes;
    setLikes((prevLikes) => prevLikes + 1);

    try {
      const response = await fetch(`/api/like/${postId}`, {
        method: 'POST',
      });

      if (!response.ok) {
        throw new Error('Network response was not ok');
      }

      const result = await response.json();
      setLikes(result.likes);
    } catch (error) {
      console.error('Error liking the post:', error);
      // 원래 상태로 롤백하지만, originalLikes를 참조하기 때문에
      // 예상대로 동작하지 않을 수 있음.
      setLikes(originalLikes);
    }
  };
```
위 코드와 같이 useState를 사용하게 되면 likesState를 함수내부에서 이원화해야된다는 단점이 존재합니다.

useOptimitic는 함수내부의 stateUpdate도 즉각적으로 반영하기 때문에 이러한 state이원화가 필요없다는게 장점이죠

## use
use는 비동기 데이터 fetching과 상태 관리를 단순화해주는 React의 새로운 기능입니다. 
이는 컴포넌트 렌더링 중에 비동기 작업을 수행하고, 결과가 반환될 때까지 기다린 후, 컴포넌트를 다시 렌더링합니다. 

use는 또한 기존의 useContext를 대체할 수 있습니다.
use는 useContext와 다르게 최상위 레벨에서가 아닌 하위 레벨에서도 호출할 수 있습니다
좀 더 유연하게 사용할 수 있겠네요
```typescript
const ColorContext = createContext("");

function App() {
  return (
    <ColorContext value="blue">
      <Form />
    </ColorContext>
  );
}

function Form() {
  return (
    <div>
      <Button show={true}>True</Button>
      <Button show={false}>False</Button>
    </div>
  );
}

function Button({
  show,
  children,
}: {
  show: boolean;
  children: React.ReactNode;
}) {
  if (show) {
    const theme = use(ColorContext);
    return <button style={{ backgroundColor: theme }}>{children}</button>;
  }
  return false;
}
```
context설명부족

### fetch와 suspense함꼐 사용하기
fetch 설명부족
```typescript
import React, { Suspense } from 'react';

const fetchData = async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
  return response.json();
};

const DataComponent = () => {
  const data = use(fetchData());

  return (
    <div>
      <h1>{data.title}</h1>
      <p>{data.body}</p>
    </div>
  );
};

const App = () => (
  <Suspense fallback={<div>Loading...</div>}>
    <DataComponent />
  </Suspense>
);

export default App;
```
이전과 다르게 useEffect를 통해 마운트 됐을때 따로 api를 호출할 필요없이 바로 use Hook을 통해 호출해서 사용할 수 있습니다. 
이젠 비동기 데이터에 한해서 편하게 use로 호출할 수 있겠군요 

## ref의 접근성 향상

이제 `ref`를 props로 직접 전달할 수 있습니다. 
19버전 이전에선 하위 컴포넌트로 ref를 넘길때 forwardRef를 이용해서 넘겼습니다 
하지만 19버전 이후로는 forwardRef가 필요없이 바로 ref값을 넣어주면 됩니다.

**이전버전**
```typescript
const ParentCompoent() {
  const ref = useRef();

  return (
    <Child ref={ref}/>
  );
}

const Child=forwardRef((props,ref)=>{
	return <div ref={ref}></div>
}
```

**19버전**
```typescript
const ParentCompoent() {
  const ref = useRef();

  return (
    <Child ref={ref}/>
  );
}

const Child=((props,ref)=>{ <----forardRef없이 바로 사용 가능함
	return <div ref={ref}></div>
}
```

그리고 ref콜백에서 clean up 함수를 반환하는 기능이 추가되었습니다. 컴포넌트가 언마운트될 때 React는 `ref` 콜백이 반환한 정리 함수를 호출합니다. 이 기능은 DOM `ref`, 클래스 컴포넌트 `ref`, `useImperativeHandle` 모두에서 동작합니다. 기존에는 컴포넌트가 언마운트될 때 `ref` 함수가 `null`로 호출되었으나, 이제 정리 함수를 반환하는 경우 이 단계가 생략됩니다. TypeScript는 이제 `ref` 콜백에서 정리 함수 외의 다른 것을 반환하면 거부합니다.

아래와 같이 input 컴포넌트가 언마운트 될떄 이벤트 리스너를 제거해줘서 쓸데없는 메모리 낭비를 막아주게 됩니다
```typescript
<input
  ref={(ref) => {
    // ref가 생성될때 실행
     ref.addEventListener('focus', handleFocus);
    
    // 아래 익명함수는 요소가 DOM에서 제거될 때 실행됩니다.
    return () => {
      // ref가 제거될 때 실행되는 정리 함수
        ref.removeEventListener('focus', handleFocus);
    };
  }}
/>
```

출처:
[https://www.youtube.com/watch?v=4JsTCSYst9M](https://www.youtube.com/watch?v=4JsTCSYst9M)  
[https://www.youtube.com/watch?v=AJOGzVygGcYM](https://www.youtube.com/watch?v=AJOGzVygGcY)  