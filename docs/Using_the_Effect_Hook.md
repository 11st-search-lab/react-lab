# Using the Effect Hook

https://ko.reactjs.org/docs/hooks-effect.html

useEffect는 함수 컴포넌트에서 side effect를 처리할 때 쓰인다. React는 useEffect가 수행되는 시점에 이미 DOM이 업데이트되었음을 보장한다. React 컴포넌트에는 일반적으로 정리(clean-up)가 필요한 것과 그렇지 않은 두 종류의 side effects가 있다. 

### side effects

1. **Clean-up 필요하지 않은 Effects**

- 데이터 가져오기

- 수동으로 React 컴포넌트의 DOM을 수정하는 것

- 로깅

  

2. **Clean-up 필요한 Effects**

- 구독(subscription) 설정하기



각 side effects를 Class 컴포넌트, 함수 컴포넌트에서 어떻게 처리하는지 알아보자. 



## 정리(Clean-up)를 이용하지 않는 Effects

### Class를 사용하는 예시

React의 class 컴포넌트에서 `render` 메서드 그 자체는 side effect를 발생시키지 않는다. effect를 수행하는 것은 React가 DOM을 업데이트하고 난 *이후* 이기 때문에 React class에서는 `componentDidMount`와 `componentDidUpdate`에서 side effect를 처리한다. 

React 클래스 컴포넌트는 side Effect를 처리하는 메서드를 따로 가지고 있지 않는다.  따라서 side Effect를 처리하는 함수를 별개의 메서드로 뽑아낸다고 해도 `componentDidMount`와 `componentDidUpdate` 두 곳에서 해당 함수를 호출해야한다.

```jsx
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```



`useEffect` Hook을 사용한다면  **class 안의 두 개의 생명주기 메서드**에 같은 코드가 중복되는 것을 개선할 수 있다.



### Hook을 사용하는 예시

React는 useEffect Hook을 기억하고 있다가 DOM 업데이트를 수행 후, 즉  컴포넌트 렌더링 후 호출한다.



### `useEffect`를 컴포넌트 안에서 호출하는 이유

`useEffect`는  컴포넌트 안에서 호출해야 한다. Hook은 `useEffect`를 컴포넌트 내부에 둠으로써 effect에서 state 변수 혹은 prop에 접근할 수 있다. React에 한정된 API를 사용하는 것이 아니라 자바스크립트가 이미 가지고 있는 방법, 즉 클로저를 이용하여 문제를 해결하는 것이다. 



### `useEffect`호출 시점 정하기

기본적으로 첫번째 렌더링과 이후의 모든 업데이트에서 수행 된다. 하지만  `useEffect`의 선택적 인수인 두 번째 인수에 배열을 넘겨 특정 값들이 리렌더링 시 변경되지 않을 경우 effect를 *건너뛰도록* 하는 방식으로 최적화를 할 수 있다. 

```jsx
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // count가 바뀔 때만 effect를 재실행
```



위의 예시에서는  `[count]`를 두 번째 인수로 넘긴다. `count`가 `5`이고 컴포넌트가 리렌더링된 이후에도 여전히 `count`는 변함없이 `5`라면 React는 이전 렌더링 시의 값 `[5]`를 그다음 렌더링 때의 `[5]`와 비교한다. 배열 내의 모든 값이 같기 때문에(`5 === 5`) React는 effect를 건너뛴다. 

`count`가 `6`으로 업데이트된 뒤에 렌더링하면 React는 이전에 렌더링된 값 `[5]`를 그다음 렌더링 시의 `[6]`와 비교한다. 이때 `5 !== 6` 이기 때문에 React는 effect를 재실행한다. 배열 내에 여러 개의 값이 있다면 그중의 단 하나만 다를지라도 React는 effect를 재실행한다.



## 정리(clean-up)를 이용하는 Effects

외부 데이터에 **구독(subscription)을 설정해야 하는 경우**에 메모리 누수가 발생하지 않도록 정리(clean-up)하는 것은 매우 중요하다.

### Class를 사용하는 예시

React class에서는 흔히 `componentDidMount`에 구독(subscription)을 설정한 뒤 `componentWillUnmount`에서 이를 정리(clean-up)한다.

```jsx
class FriendStatus extends React.Component {
 ...
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  ...
}
```

`componentDidMount`와 `componentWillUnmount` 를 살펴보면 두 개의 메서드 내에 개념상 똑같은 effect 코드가 있음에도 이를 분리해서 작성해야 한다.



### Hook을 이용하는 예시 

구독(subscription)의 추가와 제거를 위한 코드는 결합도가 높기 때문에 useEffect는 이를 함께 다루도록 고안되었다. effect가 함수를 반환하면 React는 그 함수를 정리가 필요한 때에 실행시킨다.

```jsx
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // effect 이후에 어떻게 정리(clean-up)할 것인지 표시
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

effect에서 반드시 유명함수(named function)를 반환해야 하는 것은 아닙니다. 목적을 분명히 하기 위해 `정리(clean-up)`라고 부르고 있지만 화살표 함수를 반환하거나 다른 이름으로 불러도 무방하다.



### 정리(clean-up) 시점

React는 컴포넌트가 마운트 해제되는 때에 정리(clean-up)를 실행한다. 하지만 effect는 한번이 아니라 렌더링이 실행되는 때마다 실행된다.



### 불필요한 정리(clean-up)의 문제 

하지만 모든 렌더링 이후에 effect를 정리(clean-up)하거나 적용하는 것이 때때로 성능 저하를 발생시키는 경우도 있다.



## 💡Tip

### 관심사 구분 -  Multiple Effect

useEffect는 여러 번 사용할 수 있습니다. 생명주기 메서드에 따라서가 아니라 **코드가 무엇을 하는지에 따라** 나눌 수가 있다. React는 컴포넌트에 사용된 *모든* useEffect는 지정된 순서에 맞춰 적용된다.



### Effect를 건너뛰어 성능 최적화하기

```jsx
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // count가 바뀔 때만 effect를 재실행
```

앞서 본 것처럼 의존성 배열에 값을 넣어 해당 값이 변경될 때만 effect가 작동하도록 할 수 있다.

하지만, 이 의존성 배열에 들어간 값이 너무 자주 변경되다면 어떻게 해야할까? 



아래와 같이 의존성 배열에 들어간 값을 제거할 수도 있지만, 그럴 경우 일반적으로 버그가 생긴다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1); // 이 effect는 'count' state에 따라 다름
    }, 1000);
    return () => clearInterval(id);
  }, []); // 🔴 버그: `count`가 종속성으로 지정되지 않음

  return <h1>{count}</h1>;
}
```

의존성 배열을 비워두면 useEffect는 컴포넌트가 마운트 될 때 1번만 실행되고 매번 렌더링 시에는 실행되지 않는다. 하지만 위 경우  count 값이 0으로 설정된 클로저를 생성하여, 매초 `setCount(0 + 1)`를 호출하므로 카운트가 1을 초과하지 않는다.



### effect 종속성 관리

이를 해결하기 위해 아래와 같이 setState  [함수 컴포넌트 업데이트 폼](https://ko.reactjs.org/docs/hooks-reference.html#functional-updates)을 사용할 수 있다.

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1); // ✅ 외부의 'count' 변수에 의존하지 않음
    }, 1000);
    return () => clearInterval(id);
  }, []); // ✅ 우리의 effect는 컴포넌트 범위의 변수를 사용하지 않음

  return <h1>{count}</h1>;
}
```

이제 `setInterval` 콜백이 1초에 한 번 실행되지만 `setCount`에 대한 내부 호출이 `count`에 최신 값을 사용할 수 있다. 

