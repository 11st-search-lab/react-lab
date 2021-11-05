# Using the State Hook
https://ko.reactjs.org/docs/hooks-state.html

> Hook은 React 16.8버전에 새로 추가되었다. Hook은 클래스 컴포넌트를 작성하지 않아도 state와 같은 특징들을 사용할 수 있다.

## Hook과 함수 컴포넌트

React의 함수 컴포넌트는 아래와 같이 선언할 수 있다..

```js
const Example = (props) => {
  // 여기서 Hook을 사용할 수 있습니다!
  return <div />;
}
```

또는 아래와 같다.

```js
function Example(props) {
  // 여기서 Hook을 사용할 수 있습니다!
  return <div />;
}
```

- 함수 컴포넌트는 **State가 없는 컴포넌트(stateless component)** 이다. 
- Hook은 React state를 함수 안에서 사용할 수 있게 해준다.
- **Hook은 클래스 안에서 동작하지 않는다**.

## Hook이란?

React의 useState Hook을 사용해보자.

```js
import React, { useState } from 'react';

function Example() {
  // ...
}
```

- **Hook이란?**
  - Hook은 특별한 함수이다. useState는 state를 함수 컴포넌트 안에서 사용할 수 있게 해준다.
- **언제 Hook을 사용할까?**
  - 이전에는 함수 컴포넌트를 사용하던 중 state를 추가하고 싶을 때, 클래스 컴포넌트로 바꿔서 코드를 작성했다. 하지만, 이제 함수 컴포넌트 안에서 Hook을 이용하여 state를 사용할 수 있다.

## state 변수 선언하기

- Class Component 

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  ...
}
```

- Hooks

```js
import React, { useState } from 'react';

function Example() {
  // 새로운 state 변수를 선언하고, 이것을 count라 부르자.
  const [count, setCount] = useState(0);
  ...
}
```

**useState는 컴포넌트가 렌더링할 때 오직 한 번만 생성된다.** React는 해당 변수를 리렌더링할 때 기억하고, 가장 최근에 갱신된 값을 제공한다. count 변수의 값을 갱신하려면 setCount를 호출하면 된다.

**setState는 클래스 컴포넌트의 this.setState와 다르게, state 변수를 갱신할 때, 병합하지 않고 대체한다.**

- **useState를 호출하는 것의 의미는?**
  - **state 변수** 를 선언 할 수 있다. 일반적으로 보통의 변수는 함수가 끝날 때 사라지지만, state 변수는 React에 의해 사라지지 않는다.
- **useState의 인자로 무엇을 넘겨주어야하나요?**
  - useState()Hook의 인자로 넘겨주는 값은 state의 초기 값이다.
- **useState는 무엇을 반환하나요?**
  - state 변수와 해당 변수를 갱신할 수 있는 함수를 반환한다.
- [**하나 또는 여러 state 변수를 사용해야 합니까?**](https://ko.reactjs.org/docs/hooks-faq.html#should-i-use-one-or-many-state-variables)
  - 함께 변경되는 값에 따라 state를 여러 state 변수로 분할하는 것을 권장한다.

## 요약

```js
 1:  import React, { useState } from 'react';
 2:
 3:  function Example() {
 4:    const [count, setCount] = useState(0);
 5:
 6:    return (
 7:      <div>
 8:        <p>You clicked {count} times</p>
 9:        <button onClick={() => setCount(count + 1)}>
10:         Click me
11:        </button>
12:      </div>
13:    );
14:  }
```

- **첫 번째 줄:** useState Hook을 React에서 가져온다.
- **네 번째 줄:** useState Hook을 이용하면, state 변수와 해당 state를 갱신할 수 있는 함수를 만들 수 있다. 또한, useState의 인자의 값으로 0을 넘겨주면 count 값을 0으로 초기화할 수 있다.
- **아홉 번째 줄:** 사용자가 버튼 클릭을 하면 setCount 함수를 호출하여 state 변수를 갱신한다. React는 새로운 count 변수를 Example 컴포넌트에 넘기며 해당 컴포넌트를 리렌더링합니다.

## 참고

- [Destructuring assignment](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
