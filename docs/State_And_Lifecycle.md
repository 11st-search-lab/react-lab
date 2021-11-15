# State and Lifecycle

> https://ko.reactjs.org/docs/state-and-lifecycle.html

`Clock`이 타이머를 설정하고 매초 UI를 업데이트하여 스스로 업데이트하도록 만들려고 한다.

```jsx
ReactDOM.render(
  <Clock />,  document.getElementById('root')
);
```

이를 구현하기 위해서 `Clock` 컴포넌트에 state를 추가해야 한다. State는 props와 유사하지만, **비공개이며 컴포넌트에 의해 완전히 제어**된다.



## 함수에서 클래스로 변환하기

다섯 단계로 `Clock`과 같은 함수 컴포넌트를 클래스로 변환할 수 있다.

1. `React.Component`를 확장하는 동일한 이름의 [ES6 class](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Classes)를 생성한다.

2. `render()`라고 불리는 빈 메서드를 추가한다.

3. 함수의 내용을 `render()` 메서드 안으로 옮긴다.

4. `render()` 내용 안에 있는 `props`를 `this.props`로 변경한다.

5. 남아있는 빈 함수 선언을 삭제한다.

   ```jsx
   class Clock extends React.Component {
     render() {
       return (
         <div>
           <h1>Hello, world!</h1>
           <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
         </div>
       );
     }
   }
   ```

   `render` 메서드는 업데이트가 발생할 때마다 호출되지만, 같은 DOM 노드로 `<Clock />`을 렌더링하는 경우 `Clock` 클래스의 단일 인스턴스만 사용된다. 이것은 로컬 state와 생명주기 메서드와 같은 부가적인 기능을 사용할 수 있게 해준다.



## 생명주기 메서드를 클래스에 추가하기

많은 컴포넌트가 있는 애플리케이션에서 컴포넌트가 삭제될 때 해당 컴포넌트가 사용 중이던 리소스를 확보하는 것이 중요하다.

### 마운팅(Mounting)

`Clock`이 처음 DOM에 렌더링 될 때마다 [타이머를 설정](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval)하려고 한다. 이것은 React에서 “마운팅”이라고 한다.

### 언마운팅(UnMounting)

또한 `Clock`에 의해 생성된 DOM이 삭제될 때마다 [타이머를 해제](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval)하려고 한다. 이것은 React에서 “언마운팅”이라고 한다.



![](https://sungjk.github.io/images/2016/10/29/LifeCycle.png)

컴포넌트 클래스에서 특별한 메서드를 선언하여 컴포넌트가 마운트되거나 언마운트 될 때 일부 코드를 작동할 수 있다.



```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
  }

  componentWillUnmount() {
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

이러한 메서드들은 “생명주기 메서드”라고 부른다.



### componentDidMount

- `componentDidMount()` 메서드는 컴포넌트 출력물이 DOM에 렌더링 된 후에 실행된다.  
- 이 장소가 타이머를 설정하기에 좋은 장소이다.

```jsx
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

```



### componentWillUnmount

- 컴포넌트가 DOM 상에서 제거될 때에 호출됩니다.
- 생명주기 메서드 안에 있는 타이머를 분해한다.

```jsx
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```



## State를 올바르게 사용하기

`setState()`에 대해서 알아야 할 세 가지가 있다.

- **직접 State를 수정하지 마세요.**

- **State 업데이트는 비동기적일 수도 있습니다.**

- **State 업데이트는 병합됩니다.**

  

### 직접 State를 수정하지 마세요

예를 들어, 이 코드는 컴포넌트를 다시 렌더링하지 않는다.

```jsx
// Wrong
this.state.comment = 'Hello';
```

대신에 `setState()`를 사용한다.

```jsx
// Correct
this.setState({comment: 'Hello'});
```

`this.state`를 지정할 수 있는 유일한 공간은 바로 constructor이다.

### 

### State 업데이트는 비동기적일 수도 있습니다.

React는 성능을 위해 여러 `setState()` 호출을 단일 업데이트로 한꺼번에 처리할 수 있다.

`this.props`와 `this.state`가 비동기적으로 업데이트될 수 있기 때문에 다음 state를 계산할 때 해당 값에 의존해서는 안 된다.

예를 들어, 다음 코드는 카운터 업데이트에 실패할 수 있다.

```jsx
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

이를 수정하기 위해 객체보다는 함수를 인자로 사용하는 다른 형태의 `setState()`를 사용한다. 그 함수는 이전 state를 첫 번째 인자로 받아들일 것이고, 업데이트가 적용된 시점의 props를 두 번째 인자로 받아들일 것이다.

```jsx
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

위에서는 [화살표 함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/애로우_펑션)를 사용했지만, 일반적인 함수에서도 정상적으로 작동합니다.

```jsx 
// Correct
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

### 

### State 업데이트는 병합됩니다.

`setState()`를 호출할 때 React는 제공한 객체를 현재 state로 병합한다.

예를 들어, state는 다양한 독립적인 변수를 포함할 수 있다.

```jsx
  constructor(props) {
    super(props);
    this.state = {
      posts: [],     
      comments: []    };
  }
```



별도의 `setState()` 호출로 이러한 변수를 독립적으로 업데이트할 수 있다.

```jsx
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments      });
    });
  }
```

병합은 얕게 이루어지기 때문에 `this.setState({comments})`는 `this.state.posts`에 영향을 주진 않지만 `this.state.comments`는 완전히 대체된다.



## 데이터는 아래로 흐릅니다.

부모 컴포넌트나 자식 컴포넌트 모두 특정 컴포넌트가 유상태인지 또는 무상태인지 알 수 없고, 그들이 함수나 클래스로 정의되었는지에 대해서 관심을 가질 필요가 없다.

이 때문에 state는 종종 **로컬** 또는 **캡슐화**라고 불린다. state가 소유하고 설정한 컴포넌트 이외에는 어떠한 컴포넌트에도 접근할 수 없다.

컴포넌트는 자신의 state를 자식 컴포넌트에 props로 전달할 수 있다.

```jsx
<FormattedDate date={this.state.date} />
```



`FormattedDate` 컴포넌트는 `date`를 자신의 props로 받을 것이고 이것이 `Clock`의 state로부터 왔는지, `Clock`의 props에서 왔는지, 수동으로 입력한 것인지 알지 못한다.

```jsx
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```



일반적으로 이를 `하향식(top-down)` 또는 `단방향식 데이터 흐름`이라고 합니다. 모든 state는 항상 특정한 컴포넌트가 소유하고 있으며 그 state로부터 파생된 UI 또는 데이터는 오직 트리구조에서 자신의 “아래”에 있는 컴포넌트에만 영향을 미친다.

트리구조가 props들의 폭포라고 상상하면 각 컴포넌트의 state는 임의의 점에서 만나지만 동시에 아래로 흐르는 부가적인 수원(water source)이라고 할 수 있다.

모든 컴포넌트가 완전히 독립적이라는 것을 보여주기 위해 `App` 렌더링하는 세 개의 `<Clock>`을 만들었습니다.

```jsx
function App() {
  return (
    <div>
      <Clock />      
      <Clock />     
      <Clock />    
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

각 `Clock`은 자신만의 타이머를 설정하고 독립적으로 업데이트를 한다.

React 앱에서 컴포넌트가 유상태 또는 무상태에 대한 것은 시간이 지남에 따라 변경될 수 있는 구현 세부 사항으로 간주한다. 유상태 컴포넌트 안에서 무상태 컴포넌트를 사용할 수 있으며, 그 반대 경우도 마찬가지로 사용할 수 있다.

