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

안녕하세요 이번 포스팅에선 React에서의 성능개선과 자주 사용되는 실수들에 대해서 정리해보는 시간을 가져보겠습니다.

# 1. Render 함수에서 인라인 함수 정의 피하기
React에서의 인라인 함수는 React가 비교하여 검사를 수행할 때 항상 prop값의 비교를 잘 하지 못합니다.
또한 화살표 함수는 jsx속성에서 사용되는 경우 각 렌더링에서 함수의 새 인스턴스를 만듭니다.
이건 가비지 컬렉터에 대해 많은 작업을 생성할 수 있습니다.

```jsx
const CommentList=()=>{
    const [comments,setComments]=useState([]);

    return(
        {
            comments.map((comment)=>
                <Comment onClick={(e)=>{
                    setComments(comment) comment={comment} key={comment.id}
                }}/>
            )
        }
    )
}

```
인라인 함수를 정의하는 대신 화살표 함수로 정의할 수 있습니다.

```ts
const CommentList=()=>{
    const [comments,setComments]=useState([]);

    const onCommentClick=(commentId)=>{
        setComments(commentId);
    }

    return(
        {
            comments.map((comment)=>
                <Comment onClick={(e)=>onCommentClick} 
                comment={comment} key={comment.id}
                )/>
        }
    )
}

```


# 2. javascript의 이벤트 동작 제한 및 디바운싱
일반적으로 마우스 클릭은 스크롤 및 마우스 오버에 비해 이벤트 트리거 속도가 낮습니다.
이벤트 트리거 비율이 높으면 어플리케이션에서 충돌할 가능성은 있지만 제어 가능합니다.

예를들어 UI업데이트를 수행하거나,많은 양의 데이터를 처리하거나,계산비용이 많이 드는 작업을 수행하는
요청이 있는 경우 이벤트 리스터를 변경하지 않고도 Throttling 및 디바운싱을 이용하여 성능을 높일 수 있습니다.

Throttling
---

Throttling(스로틀링)이란 함수가 지정된 시간동안 최대 한번만 호출되도록 하는 방법입니다.

Throttling이란 실행을 지연시키는 것입니다. 이벤트 핸들러 함수를 즉시 실행하는 대신에 
이벤트가 트리거 될 떄 몇 밀리초의 지연을 추가하게 됩니다.
예를들면 무한 스크롤을 구현할 때 사용할 수 있습니다.
사용자가 스크롤 할 때 리스트를 가져오는 대신 xhr 호출을 지연할 수 있습니다.

또하나의 예로는 xhr기반의 인스턴스 검색입니다. 키를 누를 때마다
서버는 누르는 것을 원하지 않을 수 있으므로 입력 필드가 몇 밀리초동안 휴면상태가 될때까지
조절하는 것이 좋습니다.

ex)10초마다 함수를 한번 호출

Debounce
---

Throttling과 달리 Debouncing은 이벤트 트리거가 너무 자주 실행되는 걸 방지하는 기술입니다.
lodash호출하려는 함수는 래핑할 수 있습니다.

ex)함수 호출 후 10초동안 함수를 호출하지 않았다면 10초가 지난 후 제일 마지막에 호출된 함수만 실행됨.


```ts
import debouce from 'lodash.debounce'
const SearchComment = ()=>{

    const [query,setQuery]=useState();

    const setSearchQuery = debounce((e)=>{
        setQuery(e.target.value);
    },1000) 

    return(
        <div>
            <input type="text" onChange={setSearchQuery}/>
        </div>
    )
}
```


# 3. map의 key값을 인덱스로 사용하지 않기
리스트를 렌더링할 때 map의 인덱스를 컴포넌트의 키로 사용되는 걸 종종 볼 수 있습니다.
```ts
    {
        comments.map((comment,index)=>{
            <Comment {...comment} key={index}>
        })
    }
```
Index를 key값으로 사용하게 되면 DOM요소를 식별하는 데 사용되는 앱에 잘못된 데이터가 표시 될 수 있습니다.
목록에 항목을 넣거나 제거할 때 Key가 이전과 동일하면 React에서는 DOM요소가 동일한 구성으로 되어있는걸로 알게 됩니다.

이러한 상황을 방지하기 위해 id를 지정해주는 모듈을 사용하는것이 좋습니다.

```ts
import shortid from  "shortid";
{
    comments.map((comment,index)=>{
        <Comment 
            {...comment} 
            key={shortid.generate()}/>
    })
}
```
하지만 무조건적으로 shortid module을 쓸 필요는 없습니다.
아래 조건만 충족이 된다면 굳이 shortid를 써서 id를 지정할 필요없습니다.

* 목록이 정적이다.
* 목록의 항목들은 재정렬되거나 필터링되지 않는다.
* 목록은 변경할 수 없다.


# 4. Prop를 초기값에 할당하지 않기

useState의 초기값을 설정하기 위해 컴포넌트에 props를 초기값으로 넣는 경우가 많습니다.
이런 경우 state가 update되었음에도 화면은 변하지 않는 문제가 발생할 수 있습니다.

```ts
const Component = (props)=>{
    const [applyCoupon,setApplyCoupon]=useState(props.applyCoupon);

    return(
        <div>
            {applyCoupon&&
            <span>Enter Coupon: <Input/></span>}
        </div>
    )
}
```
해결방법으로는 구성요소에 직접 props를 넣어주던가 useEffect에 setState를 넣어 값을 변경해주면 됩니다.

```ts
const Component = (props)=>{
    const [applyCoupon,setApplyCoupon]=useState();

    useEffect(()=>{
        setApplyCoupon(props.applyCoupon);
    },[props.applyCoupon])

    return(
        <div>
            {applyCoupon&&
            <span>Enter Coupon: <Input/></span>}
        </div>
    )
}
```


# 5. DOM에서 펼침연산자(Spread operator) 사용
알수 없는 html속성을 추가하기 때문에 DOM요소에 펼칩 연산자를 쓰는건 피해야합니다.
```ts
const CommentText=props=>{
    return(
        <div {...props}>
            {props.text}
        </div>
    )
}
```
펼침 연산자를 쓰는 대신 특정 속성을 설정합시다.
```ts
const CommentText = props => {
    return (
      <div specificAttr={props.specificAttr}>
        {props.text}
      </div>
    );
};

```

# 6. CPU를 많이 사용하는 작업에는 WebWorker사용하기

웹워커를 사용하면 기본 실행 스레드와 별도로 웹 어플리케이션의 백그라운드의 스레드에서 스크립트 작업을 할 수 있습니다.
별도의 스레드에서 복잡한 처리를 수행함으로써 일반적으로 UI인 메인스레드가 느려지지 않고 실행될 수 있습니다.

동일한 컨텍스트에서 js는 단일 스레드이므로 병렬 계산이 필요합니다.
두가지 방법으로 실행 할 수 있으며
첫번째 방법은 setTimeout기능을 기반으로 하는 유사 병렬처리를 사용하는 것입니다.
두번째 방법은 웹 워커를 사용하는 것입니다.

웹 워커는 백그라운드에서 별도의 스레드에서 다른 스크립트와 독립적으로 코드를 실행하기 때문에
광범위한 연산을 실행할 때 가장 잘 동작합니다.

다음은 웹워커를 사용하지 않는 코드입니다.

```ts


export const Posts=posts=>{

    const [sortedPost,setSortedPost]=useState();

    const sort=(posts)=>{
        for (let index = 0, len = posts.length - 1; index < len; index++) {
            for (let count = index+1; count < posts.length; count++) {
                if (posts[index].commentCount > posts[count].commentCount) {
                    const temp = posts[index];
                    posts[index] = users[count];
                    posts[count] = temp;
                }
            }
        }
    return posts;
    }

    const doSortingByComment=()=>{
         if(posts && posts.length){
            const sortedPosts = sort(posts);
            setSortedPost(sortedPosts)
        }
    }
    return(
        <>
            <Button onClick={doSortingByComment}>
                Sort By Comment
            </Button>
            <PostList post={posts}>
        </>
    )
}
```

다음은 웹워커를 사용하여 정렬을 처리하는 코드입니다.

```ts
import { useWorker } from "@koale/useworker";

const sort=()=>{
    self.addEventListener('message', e =>{
        if (!e) return;
        let posts = e.data;
        
        for (let index = 0, len = posts.length - 1; index < len; index++) {
            for (let count = index+1; count < posts.length; count++) {
                if (posts[index].commentCount > posts[count].commentCount) {
                    const temp = posts[index];
                    posts[index] = users[count];
                    posts[count] = temp;
                }
            }
        }
        postMessage(posts);
    });
}

export const Posts=posts=>{
    const [sortWorker]=useWorker(sort);

    const doSortingByComment = async()=>{
          if(posts && posts.length){
            const sortedPosts = await sortWorkerort(posts);
            setSortedPost(sortedPosts)
        }
    }

    return(
        <>
            <Button onClick={doSortingByComment}>
                Sort By Comments
            </Button>
            <PostList posts={posts}/>
        </>
    )
}

```

sort함수는 별도의 스레드에서 메서드를 실행하므로 기본 스레드를 차단하지 않습니다.
이미지 처리나 정렬 필터링 및 기타 CPU 확장작업 같은 경우에는 웹워커를 사용하는 것을 고려할 수 있습니다.


# 7. React에서의 Memoization

## Memoization이란?

메모이제이션(memoization)은 컴퓨터 프로그램이 동일한 계산을 반복해야 할 때, 이전에 계산한 값을 메모리에 저장함으로써 동일한 계산의 반복 수행을 제거하여 프로그램 실행 속도를 빠르게 하는 기술이다. 동적 계획법의 핵심이 되는 기술이다. 메모이제이션이라고도 한다.

먼저 리엑트에서 메모이제이션을 쓰는 이유는 크게 두가지입니다.
1. 복잡한 연산을 피하여 성능을 향상시킨다.
2. 안정된 값을 제공한다.

React에서 Memoization을 이용할 때 주로 쓰이는 기능들은 
useMemo,useCallback,React.memo가 있습니다.

## useMemo
---
useMemo은 메모이제이션된 값을 반환해줍니다.
useMemo는 유효성 배열이 변하면 useMemo안에 함수를 실행해서 재계산해서 결과값을 반환해줍니다.

```ts

const factorial = (n) => {
  if (n < 0) return -1;
  if (n === 0) return 1; 
  return n * factorial(n - 1);
};
const Component=()=>{
    const [counter,setCounter]=useState(1);
    const result = useMemo(()=> factorial(counter), [counter]);
    return (
        <>
            <div>Factorial of {counter} is: {result}</div>
            <div>
                <button onClick={() => setCounter(counter - 1)}>-</button>
                <button onClick={() => setCounter(counter + 1)}>+</button>
            </div>
        </>
    )
}
```

그렇다고해서 무조건적으로 의존성 배열이 변한다고해서 재계산을해서 결과값을 반환시켜주지 않습니다.
React 공식문서에 따르면 
`useMemo는 성능 최적화를 위해 사용할 수는 있지만 의미상으로 보장이 있다고 생각하지는 마세요. 가까운 미래에 React에서는, 이전 메모이제이션된 값들의 일부를 “잊어버리고” 다음 렌더링 시에 그것들을 재계산하는 방향을 택할지도 모르겠습니다. 예를 들면, 오프스크린 컴포넌트의 메모리를 해제하는 등이 있을 수 있습니다. useMemo를 사용하지 않고도 동작할 수 있도록 코드를 작성하고 그것을 추가하여 성능을 최적화하세요.`라고 되어있다.
데이터를 100%보장할 수 없는 구조는 아니라는건데 이걸 통해 알 수 있는건 useMemo는 
메모리 절약을 위해 데이터를 유지하지 않을 수 있습니다.

React공식문서에서도 useMemo는 데이터를 저장하는것이 아니라 성능을 최적화 하는데 사용해야한다고 합니다.

useMemo가 득보다 실이 많을 수 있는경우는 메모이제이션되는 기능이 비용이 많이들지 않는 작업을 수행하는 경우입니다.
useMemo 자체적으로 메모리가 필요하므로 모든 기능을 과도하게 최적화하려고 하면 어플리케이션 속도가 느려질 수 있습니다.

또한 useMemo함수가 boolean 또는 문자열과 같은 기본값을 반환할 때도 사용해서는 안됩니다. 기본값은
참조가 아닌 값으로 전달되기 때문에 구성요소가 다시 리렌더링 되더라도 항상 동일하게 유지가 됩니다.

아래 예제를 살펴보죠
```ts
const BigMovieComponent=(movieId)=>{

    const movieTitleAsString = getMovieTitle(movieId); 

    return(
        <ListChildComponent movieTitle={movieTitleAsString}>
    )
}
```
이 경우에는 movieTitleAsString함수는 자식컴포넌트에 전달되는 moveTitle에 들어가는 값을 반환해줍니다.
메모된 값이 같으면 자식 구성요소는 다시 렌더링이 되지 않습니다.

그러나 부모의 구성요소가 렌더링될 때 전달되는 함수인 useMemo는 계속 실행되고 비싼 연산이 수행되는 경우
성능에 영향을 많이 미칩니다.

또한 비싸지 않는 연산일 경우에는 javascript배열 및 객체 메서드를 메모이제이션할 필요가 없습니다.

메모하지 말아야할 또 다른 유형은 초기 데이터입니다.setState에 저장된 데이터가 복잡하고 많은 메모리를 사용하더라도
작동방식 때문에 구성요소가 마운트 해제되지 않으면 다시 계산되지 않습니다.

## useCallback
---
useMemo의 함수형 버전이며 메모이제이션을 콜백함수를 반환해 줍니다.

React 컴포넌트 내에서 함수가 선언되었다면 그 함수는 컴포넌트가 렌더링될 때마다 새로운 함수로 나타나질 겁니다.
하지만 useCallback을 사용하면 컴포넌트가 리렌더링이 되더라도 useCallback의 의존성 배열이 바뀌지 않는 한
기존 함수를 계속 반환해줍니다.

아래 예제를 살펴보죠
```ts
    const ListContainer =({searchQuery})=>{
        const itemClick=useCallback((event)=>{
            console.log('click')
        },[searchQuery]);
        return(
            <ListComponent
                searchQuery={searchQuery}
                onItemClick={onItemClick}
            />
        )
      }
```
onItemClick 콜백함수는 useCallback에 의해 메모됩니다.따라서 searchQuery 
prop와 동일한 useCallback은 동일한 함수를 반환합니다.

ListComponent컴포넌트가 다시 렌더링될 때 onItemClick함수 객체는 그대로 유지되며
ListComponent의 메모이제이션을 중단하지 않습니다.


잘못된 사례
```ts
const Component=()=>{
   const onHandleClick = useCallback(() => {
       console.log('click');
  }, []);
  return <ChildComponent onClick={onHandleClick}>
}
const ChildComponent =({onClick})=>{
    return <button onClick={onClick}>Click</button>
}
```
useCallback은 Component가 렌더링 될 때마다 호출이 됩니다.
useCallback이 동일 한 함수를 반환하더라도 여전히 인라인 함수는 다시 렌더링 될때마다 다시 생성이 됩니다.


## React.Memo
---
React에서 컴포넌트는 렌더링을 한 뒤에 이전에 렌더링 된 결과와 비교하여 바뀐 부분이 있는 경우 DOM을 업데이트를합니다.

아래 구조의 컴포넌트를 한번 살펴보죠
```
Parent
-Child
```

React.memo를 이용해 children의 상태값이 바뀌더라도 새로 변경된 Child만 렌더링하고 기존에 렌더링된
Child들은 렌더링이 되지 않도록 만들어봅시다.

Parent
```ts
export const Parent=()=>{
    const [children,setChildren]=useState([
        {
            name:"Kim",
            key:0
        },
        {
            name:"Lee",
            key:1
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
                children.map((child)=>
                <Child key={child.key} child={child}>
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
        <div>이름: {name}</div>
    </>
} 
```

버튼을 눌러 Children을 변화시켜
Parent를 리렌더링 시키더라도 새로 추가된 children만 새로 렌더링되고 이미 렌더링된
children은 리렌더링 되지 않습니다.


React.memo가 필요한 경우
상태값이나 데이터가 자주 변경되지 않으며 컴포넌트를 자주 업데이트를 하지 않는 경우 굳이 얕은 비교를 하면서
리렌더링을 할 필요가 없기 때문입니다.
state나 props의 데이터가 변경되지 않고 컴포넌트를 자주 업데이트 하지 않아도 될때

React.memo가 불필요한 경우
컴포넌트가 같은 prop로 자주 리렌더링이 되지 않는 경우에는 굳이 React.memo를 사용할 필요는없다.
잘못된 사용은 오히려 성능을 저하 시킬 수 있습니다.




참고:https://blog.logrocket.com/optimizing-performance-react-application/
https://www.codementor.io/blog/react-optimization-5wiwjnf9hj
https://ui.toast.com/weekly-pick/ko_20190731