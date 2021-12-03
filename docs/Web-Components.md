# Web Components

> https://ko.reactjs.org/docs/web-components.html

React와 웹 컴포넌트는 서로 다른 문제를 해결하기 위해 만들어졌다. 웹 컴포넌트는 재사용할 수 있는 컴포넌트에 강한 캡슐화를 제공하는 반면, React는 데이터와 DOM을 동기화하는 선언적 라이브러리를 제공한다.

## React에서 웹 컴포넌트 사용하기

```js
class HelloMessage extends React.Component {
  render() {
    return <div>Hello <x-search>{this.props.name}</x-search>!</div>;
  }
}
```

> **주의**
>
> 웹 컴포넌트는 종종 강제성을 띠는 API를 열어놓고 있다. 예를 들어, video라는 웹 컴포넌트는 play()나 pause()라는 함수를 열어놓고 있을 것이다. 이러한 웹 컴포넌트의 강제성을 띠는 API에 접근하기 위해서, DOM 노드에 직접 ref를 지정하는 것이 필요할 수 있다. 서드 파티 웹 컴포넌트를 사용 중이라면, 가장 좋은 해결방법은 웹 컴포넌트의 래퍼로서 동작하는 React 컴포넌트를 작성하는 것이다.
>
> 웹 컴포넌트에서 나온 이벤트들은 React 렌더링 트리에 올바르게 전파되지 않을 수 있다. 이를 해결하기 위해 이벤트를 다루기 위한 핸들러를 React 컴포넌트 내에 각각 만들어야다.

웹 컴포넌트는 “className”이 아닌 “class”를 사용한다.

```js
function BrickFlipbox() {
  return (
    <brick-flipbox class="demo">
      <div>front</div>
      <div>back</div>
    </brick-flipbox>
  );
}
```

## 웹 컴포넌트에서 React 사용하기

```js
class XSearch extends HTMLElement {
  connectedCallback() {
    const mountPoint = document.createElement('span');
    this.attachShadow({ mode: 'open' }).appendChild(mountPoint);

    const name = this.getAttribute('name');
    const url = 'https://www.google.com/search?q=' + encodeURIComponent(name);
    ReactDOM.render(<a href={url}>{name}</a>, mountPoint);
  }
}
customElements.define('x-search', XSearch);
```

## 참고

- https://developer.mozilla.org/ko/docs/Web/Web_Components
- https://d2.naver.com/helloworld/188655
- https://www.html5rocks.com/ko/tutorials/webcomponents/template/