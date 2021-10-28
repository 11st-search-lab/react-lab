# Context 

context는 React 컴포넌트 트리 안에서 전역적(global)이라고 볼 수 있는 데이터를 공유할 수 있도록 고안된 방법이다. 일반적인 React 애플리케이션에서 데이터는 위에서 아래로 (즉, 부모로부터 자식에게) props를 통해 전달되지만, App 안의 여러 컴포넌트들에 전해줘야 하는 props의 경우 (예를 들면 선호 로케일, UI 테마) 이 과정이 번거로울 수 있다. context를 이용하면, 트리 단계마다 명시적으로 props를 넘겨주지 않아도 많은 컴포넌트가 값을 공유하도록 할 수 있다.

## Context를 사용하는 경우

- 로그인한 유저

- 테마

- 선호하는 언어

- 데이터 캐시

  

## Context API 사용법

### `React.createContext`

 Context 객체를 만든다. Context 객체를 구독하고 있는 컴포넌트를 렌더링할 때 React는 트리 상위에서 가장 가까이 있는 짝이 맞는 `Provider`로부터 현재값을 읽는다.

```jsx
const MyContext = React.createContext(defaultValue);
```

`defaultValue` 매개변수는 트리 안에서 적절한 Provider를 **찾지 못했을 때만** 쓰이는 값이다.



### `Context.Provider`

```jsx
<MyContext.Provider value={/* 어떤 값 */}>
```

Context 오브젝트에 포함된 React 컴포넌트인 Provider는 context를 구독하는 컴포넌트들에게 context의 변화를 알린다.

Provider 컴포넌트는 `value` prop을 받아서 이 값을 하위에 있는 컴포넌트에게 전달한다. 값을 전달받을 수 있는 컴포넌트의 수에 제한은 없다. Provider 하위에 또 다른 Provider를 배치하는 것도 가능하며, 이 경우 하위 Provider의 값이 우선시된다.

Provider 하위에서 context를 구독하는 모든 컴포넌트는 Provider의 `value` prop가 바뀔 때마다 다시 렌더링 된다. Provider로부터 하위 consumer([`.contextType`](https://ko.reactjs.org/docs/context.html#classcontexttype)와 [`useContext`](https://ko.reactjs.org/docs/hooks-reference.html#usecontext)을 포함한)로의 전파는 `shouldComponentUpdate` 메서드가 적용되지 않으므로, **상위 컴포넌트가 업데이트를 건너 뛰더라도 consumer가 업데이트**된다.

> cf) context 값의 바뀌었는지 여부는 [`Object.is`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/is#설명)와 동일한 알고리즘을 사용해 이전 값과 새로운 값을 비교해 측정된다. 이와 같은 방식으로 변화를 측정하기 때문에 객체를 `value`로 보내는 경우 다소 문제가 생길 수 있다. [주의사항](https://ko.reactjs.org/docs/context.html#caveats) 참조



### useContext

```jsx
const value = useContext(MyContext);
```

useContext로 Context객체의 value를 가져올 수 있다. context 객체(`React.createContext`에서 반환된 값)을 받아 그 context의 현재 값을 반환한다. context의 현재 값은 트리 안에서 이 Hook을 호출하는 컴포넌트에 가장 가까이에 있는 `<MyContext.Provider>`의 `value` prop에 의해 결정된다.

useContext로 전달한 인자는 context 객체 그 자체이어야 한다. 

- useContext(MyContext) ⭕️

- useContext(MyContext.Consumer) ❌

- useContext(MyContext.Provider) ❌

`useContext`를 호출한 컴포넌트는 context 값이 변경되면 항상 리렌더링 될 것이다. 컴포넌트를 리렌더링 하는 것에 비용이 많이 든다면, [메모이제이션을 사용하여 최적화할 수 있다.](https://github.com/facebook/react/issues/15156#issuecomment-474590693)

`useContext(MyContext)`는 클래스에서의 `static contextType = MyContext` 또는 `<MyContext.Consumer>`와 같다고 보면 된다.



### `Context.Consumer`

```jsx
<MyContext.Consumer>
  {value => /* context 값을 이용한 렌더링 */}
</MyContext.Consumer>
```

context 변화를 구독하는 React 컴포넌트이다. 이 컴포넌트를 사용하면 [함수 컴포넌트](https://ko.reactjs.org/docs/components-and-props.html#function-and-class-components)안에서 context를 구독할 수 있다.

Context.Consumer의 자식은 [함수](https://ko.reactjs.org/docs/render-props.html#using-props-other-than-render)여야한다. 이 함수는 context의 현재값을 받고 React 노드를 반환한다. 이 함수가 받는 `value` 매개변수 값은 해당 context의 Provider 중 상위 트리에서 **가장 가까운 Provider의 `value` prop과 같다**. 상위에 Provider가 없다면 `value` 매개변수 값은 `createContext()`에 보냈던 `defaultValue`와 같다.



### `Context.displayName`

Context 객체는 `displayName` 문자열 속성을 설정할 수 있다. React 개발자 도구는 이 문자열을 사용해서 context를 어떻게 보여줄 지 결정한다.

예를 들어, 아래 컴포넌트는 개발자 도구에 MyDisplayName로 표시된다.

```jsx
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';

<MyContext.Provider> // "MyDisplayName.Provider" in DevTools
<MyContext.Consumer> // "MyDisplayName.Consumer" in DevTools
```