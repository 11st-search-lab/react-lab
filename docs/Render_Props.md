# Render Props
https://ko.reactjs.org/docs/render-props.html

> “render prop”란, React 컴포넌트 간에 코드를 공유하기 위해 함수 props를 이용하는 간단한 테크닉이다.

render props 패턴으로 구현된 컴포넌트는 자체적으로 렌더링 로직을 구현하는 대신, react 엘리먼트 요소를 반환하고 이를 호출하는 함수를 사용한다.

```js
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

## 횡단 관심사(Cross-Cutting Concerns)를 위한 render props 사용법

컴포넌트는 React에서 코드의 재사용성을 위해 사용하는 주요 단위이다.

아래 컴포넌트는 웹 애플리케이션에서 마우스 위치를 추적하는 로직이다.
```js
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        <h1>Move the mouse around!</h1>
        <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}
```

다른 컴포넌트에서 이 행위를 재사용하려면 어떻게 해야 할까?

React에서 컴포넌트는 코드 재사용의 기본 단위이므로, 우리가 필요로 하는 마우스 커서 트래킹 행위를 `<Mouse>` 컴포넌트로 캡슐화하여 어디서든 사용할 수 있게 리팩토링 해보자.

```js
// <Mouse> 컴포넌트는 우리가 원하는 행위를 캡슐화 합니다...
class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

        {/* ...하지만 <p>가 아닌 다른것을 렌더링하려면 어떻게 해야 할까요? */}
        <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <>
        <h1>Move the mouse around!</h1>
        <Mouse />
      </>
    );
  }
}
```

첫 번째 방법으로는, 다음과 같이 `<Mouse>` 컴포넌트의 render 메서드안에 `<Cat>` 컴포넌트를 넣어 렌더링하는 방법이 있다.

```js
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class MouseWithCat extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

        {/*
          여기서 <p>를 <Cat>으로 바꿀 수 있습니다. ... 그러나 이 경우
          Mouse 컴포넌트를 사용할 때 마다 별도의 <MouseWithSomethingElse>
          컴포넌트를 만들어야 합니다, 그러므로 <MouseWithCat>는
          아직 정말로 재사용이 가능한게 아닙니다.
        */}
        <Cat mouse={this.state} />
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <MouseWithCat />
      </div>
    );
  }
}
```

여기에 render prop를 사용할 수 있다.

```js
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

        {/*
          <Mouse>가 무엇을 렌더링하는지에 대해 명확히 코드로 표기하는 대신,
          `render` prop을 사용하여 무엇을 렌더링할지 동적으로 결정할 수 있습니다.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

**render prop은 무엇을 렌더링할지 컴포넌트에 알려주는 함수이다.**

```js
// 어떤 이유 때문에 HOC를 만들기 원한다면, 쉽게 구현할 수 있습니다.
// render prop을 이용해서 일반적인 컴포넌트를 만드세요!
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}/>
      );
    }
  }
}
```

## render 이외의 Props 사용법

> 여기서 중요하게 기억해야 할 것은, “render props pattern”으로 불리는 이유로 꼭 prop name으로 render를 사용할 필요는 없다. 사실, 어떤 함수형 prop이든 render prop이 될 수 있다.

위 예시에서는 render를 사용했지만, 우리는 children prop을 더 쉽게 사용할 수 있다.

```js
<Mouse children={mouse => (
  <p>The mouse position is {mouse.x}, {mouse.y}</p>
)}/>
```

실제로 JSX element의 “어트리뷰트”목록에 하위 어트리뷰트 이름(예를들면 render)을 지정할 필요는 없다. 대신에, element 안에 직접 꽂아넣을 수 있다.

```js
<Mouse>
  {mouse => (
    <p>The mouse position is {mouse.x}, {mouse.y}</p>
  )}
</Mouse>
```

이 테크닉은 자주 사용되지 않기 때문에, API를 디자인할 때 children은 함수 타입을 가지도록 propTypes를 지정하는 것이 좋다.

```js
Mouse.propTypes = {
  children: PropTypes.func.isRequired
};
```

# 주의사항

## React.PureComponent에서 render props pattern을 사용할 땐 주의해주세요.

**render props 패턴을 사용하면 React.PureComponent를 사용할 때 발생하는 이점이 사라질 수 있다.**
얕은 prop 비교는 새로운 prop에 대해 항상 false를 반환한다. 이 경우 render마다 render prop으로 넘어온 값을 항상 새로 생성한다.

위에서 사용했던 `<Mouse>` 컴포넌트를 이용해서 예를 들어보겠습니다. mouse에 React.Component 대신에 React.PureComponent를 사용하면 다음과 같은 코드가 된다.

```js
class Mouse extends React.PureComponent {
  // 위와 같은 구현체...
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>

        {/*
          이것은 좋지 않습니다! `render` prop이 가지고 있는 값은
          각각 다른 컴포넌트를 렌더링 할 것입니다.
        */}
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

이 예시에서 `<MouseTracker>`가 render 될때마다, `<Mouse render>`의 prop으로 넘어가는 함수가 계속 새로 생성된다. 따라서 React.PureComponent를 상속받은 `<Mouse>` 컴포넌트 효과가 사라지게 된다.

이 문제를 해결하기 위해서, 다음과 같이 인스턴스 메서드를 사용해서 prop을 정의한다.

```js
class MouseTracker extends React.Component {
  // `this.renderTheCat`를 항상 생성하는 매서드를 정의합니다.
  // 이것은 render를 사용할 때 마다 *같은* 함수를 참조합니다.
  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

