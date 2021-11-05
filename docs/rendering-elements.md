# 엘리먼트 렌더링

> https://ko.reactjs.org/docs/rendering-elements.html



- 엘리먼트는 **React 앱의 가장 작은 단위**이다. 

- 엘리먼트는 **화면에 표시할 내용**을 기술한다.

  

브라우저 DOM 엘리먼트와 달리 React 엘리먼트는 `일반 객체이며(plain object)` 쉽게 생성할 수 있다. React DOM은 React 엘리먼트와 일치하도록 DOM을 업데이트한다. 

더 널리 알려진 개념인 `컴포넌트`와 엘리먼트를 혼동할 수 있으나 엘리먼트는 컴포넌트의 `구성 요소`라는 것을 기억해야한다.



## DOM에 엘리먼트 렌더링하기

HTML 파일 어딘가에 아래와 같은 `<div>`가 있다고 가정해 보자.

```ts
<div id="root"></div>
```

이 안에 들어가는 모든 엘리먼트를 React DOM에서 관리하기 때문에 이것을 `루트(root) DOM 노드`라고 부른다.



React로 구현된 애플리케이션은 일반적으로 하나의 루트 DOM 노드가 있다. React를 기존 앱에 통합하려는 경우 원하는 만큼 많은 수의 독립된 루트 DOM 노드가 있을 수 있다.

React 엘리먼트를 루트 DOM 노드에 렌더링하려면 둘 다 [`ReactDOM.render()`](https://ko.reactjs.org/docs/react-dom.html#render)로 전달하면 된다.

```ts
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```



## 렌더링 된 엘리먼트 업데이트하기

React 엘리먼트는 [불변객체](https://ko.wikipedia.org/wiki/불변객체)이다. 

> 객체 지향 프로그래밍에 있어서 **불변객체**(immutable object)는 생성 후 그 상태를 바꿀 수 없는 객체를 말한다.

엘리먼트를 생성한 이후에는 해당 엘리먼트의 자식이나 속성을 변경할 수 없다. 엘리먼트는 영화에서 하나의 프레임과 같이 특정 시점의 UI를 보여준다. 지금까지 소개한 내용을 바탕으로 하면 UI를 업데이트하는 유일한 방법은 새로운 엘리먼트를 생성하고 이를 [`ReactDOM.render()`](https://ko.reactjs.org/docs/react-dom.html#render)로 전달하는 것이다.



예시) 똑딱거리는 시계

```jsx
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```



위 함수는 [`setInterval()`](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval) 콜백을 이용해 초마다 [`ReactDOM.render()`](https://ko.reactjs.org/docs/react-dom.html#render)를 호출한다.

> 실제로 대부분의 React 앱은 `ReactDOM.render()`를 한 번만 호출한다. State and Lifecycle장에서 이와 같은 코드가 [유상태 컴포넌트](https://ko.reactjs.org/docs/state-and-lifecycle.html)에 어떻게 캡슐화되는지 설명한다.



## 변경된 부분만 업데이트하기

React DOM은 해당 엘리먼트와 그 자식 엘리먼트를 이전의 엘리먼트와 비교하고 DOM을 원하는 상태로 만드는데 필요한 경우에만 DOM을 업데이트한다.

![DOM inspector showing granular updates](https://ko.reactjs.org/c158617ed7cc0eac8f58330e49e48224/granular-dom-updates.gif)



매초 전체 UI를 다시 그리도록 엘리먼트를 만들었지만 React DOM은 내용이 변경된 텍스트 노드만 업데이트한다.

특정 시점에 UI가 어떻게 보일지 고민하는 이런 접근법이 시간의 변화에 따라 UI가 어떻게 변화할지 고민하는 것보다 더 많은 수의 버그를 없앨 수 있다.