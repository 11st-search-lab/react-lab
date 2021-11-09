# Handling Events
https://ko.reactjs.org/docs/handling-events.html

- React의 이벤트는 소문자 대신 캐멀 케이스(camelCase)를 사용한다.
- JSX를 사용하여 문자열이 아닌 함수로 이벤트 핸들러를 전달한다.

예를 들어, HTML은 다음과 같다.

```html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

React에서는 약간 다르다.

```js
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

또 다른 차이점으로, **React에서는 false를 반환해도 기본 동작을 방지할 수 없다.** 반드시 preventDefault를 명시적으로 호출해야 한다.

```html
<form onsubmit="console.log('You clicked submit.'); return false">
  <button type="submit">Submit</button>
</form>
```

React에서는 다음과 같이 작성할 수 있다.

```js
function Form() {
  function handleSubmit(e) {
    e.preventDefault();
    console.log('You clicked submit.');
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Submit</button>
    </form>
  );
}
```

여기서 e는 합성 이벤트이다. React 이벤트는 브라우저 고유 이벤트와 정확히 동일하게 동작하지는 않는다.

React를 사용할 때 DOM 엘리먼트가 생성된 후 리스너를 추가하기 위해 addEventListener를 호출할 필요가 없다. 대신, 엘리먼트가 처음 렌더링될 때 리스너를 제공한다.

```js
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // 콜백에서 `this`가 작동하려면 아래와 같이 바인딩 해주어야 합니다.
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

JSX 콜백 안에서 this의 의미에 대해 주의해야 한다. JavaScript에서 클래스 메서드는 기본적으로 바인딩되어 있지 않다.

bind를 호출하는 것이 불편하다면, 이를 해결할 수 있는 두 가지 방법이 있다.

1. **퍼블릭 클래스 필드 문법 사용**
2. **콜백에 화살표 함수를 사용**

실험적인 퍼블릭 클래스 필드 문법을 사용하고 있다면, 클래스 필드를 사용하여 콜백을 올바르게 바인딩할 수 있다.

```js
class LoggingButton extends React.Component {
  // 이 문법은 `this`가 handleClick 내에서 바인딩되도록 합니다.
  // 주의: 이 문법은 *실험적인* 문법입니다.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

클래스 필드 문법을 사용하고 있지 않다면, 콜백에 화살표 함수를 사용하는 방법도 있다.

```js
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // 이 문법은 `this`가 handleClick 내에서 바인딩되도록 합니다.
    return (
      <button onClick={() => this.handleClick()}>
        Click me
      </button>
    );
  }
}
```

이 문법의 문제점은 LoggingButton이 렌더링될 때마다 다른 콜백이 생성된다는 것이다. 

대부분의 경우 문제가 되지 않으나, 콜백이 하위 컴포넌트에 props로서 전달된다면 그 컴포넌트들은 추가로 다시 렌더링을 수행할 수도 있다.

**이러한 종류의 성능 문제를 피하고자, 생성자 안에서 바인딩하거나 클래스 필드 문법을 사용하는 것을 권장한다.**

## 이벤트 핸들러에 인자 전달하기

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

위 두 줄은 동등하며 각각 화살표 함수와 Function.prototype.bind를 사용한다.

화살표 함수를 사용하면 명시적으로 인자를 전달해야 하지만 bind를 사용할 경우 추가 인자가 자동으로 전달된다.
