# Ref와 DOM

Ref는 render 메서드에서 생성된 DOM 노드나 React 엘리먼트에 접근하는 방법을 제공한다.

일반적인 React의 데이터 플로우에서 [props](https://ko.reactjs.org/docs/components-and-props.html)는 부모 컴포넌트가 자식과 상호작용할 수 있는 유일한 수단이다. 자식을 수정하려면 새로운 props를 전달하여 자식을 다시 렌더링해야 한다. 그러나, 일반적인 데이터 플로우에서 벗어나 React 컴포넌트의 인스턴스, DOM 엘리먼와 같은 자식을 수정해야 하는 경우도 있다. React는 두 경우 모두를 위한 해결책을 제공한다.



### Ref를 사용해야 할 때

- 포커스, 텍스트 선택영역, 혹은 미디어의 재생을 관리할 때
- 애니메이션을 직접적으로 실행시킬 때
- 서드 파티 DOM 라이브러리를 React와 같이 사용할 때

선언적으로 해결될 수 있는 문제에서는 ref 사용을 지양해야한다. 예를 들어, `Dialog` 컴포넌트에서 `open()`과 `close()` 메서드를 두는 대신, `isOpen`이라는 prop을 넘겨주자.



### Ref를 남용하지 마세요

ref는 애플리케이션에 **어떤 일이 일어나게** 할 때 사용될 수도 있다. Ref를 남용하는 것 보다 Ref 사용 전 어느 컴포넌트 계층에서 상태를 소유해야하는 지 신중히 생각한 후 사용하자.



### Ref 생성하기

Ref는 `React.createRef()`를 통해 생성되고 `ref` 어트리뷰트를 통해 React 엘리먼트에 부착된다. 보통, 컴포넌트의 인스턴스가 생성될 때 Ref를 프로퍼티로서 추가하고, 그럼으로서 컴포넌트의 인스턴스의 어느 곳에서도 Ref에 접근할 수 있게 한다.



> App 컴포넌트가 setState에 의해 리렌더링 되면 App 컴포넌트를 초기화 하고 다시 만듭니다. 그에 따라 `createRef`도 다시 실행되는데, 이 때 `createRef`에 의해 만들어진 값은 무조건 `null`로 다시 초기화 됩니다.
>
> 그래서 함수혐 컴포넌트의 수명 내내 ref의 current 값을 유지하기 위해 `useRef` hook이 만들어진 것입니다.
>
> `useRef`로 만들어진 값은 함수가 리렌더링 되어도 ref가 null로 초기화 되지 않습니다. (ref 값이 다시 재생성되지 않음)
>
> [참고](https://kyounghwan01.github.io/blog/React/useRef-createRef/#%E1%84%92%E1%85%A1%E1%86%B7%E1%84%89%E1%85%AE%E1%84%92%E1%85%A7%E1%86%BC-%E1%84%8F%E1%85%A5%E1%86%B7%E1%84%91%E1%85%A9%E1%84%82%E1%85%A5%E1%86%AB%E1%84%90%E1%85%B3)

### Ref 에 접근하기

`render` 메서드 안에서 ref가 엘리먼트에게 전달되었을 때, 그 노드를 향한 참조는 ref의 `current` 어트리뷰트에 담기게 된다. ref의 값은 노드의 유형에 따라 다르다.

```jsx
const node = this.myRef.current
```

- `ref` 어트리뷰트가 HTML 엘리먼트에 쓰였다면, 생성자에서 `React.createRef()`로 생성된 `ref`는 자신을 전달받은 DOM 엘리먼트를 `current` 프로퍼티의 값으로서 받는다.
- `ref` 어트리뷰트가 커스텀 클래스 컴포넌트에 쓰였다면, `ref` 객체는 마운트된 컴포넌트의 인스턴스를 `current` 프로퍼티의 값으로서 받는다.
- **함수 컴포넌트는 인스턴스가 없기 때문에 함수 컴포넌트에 ref 어트리뷰트를 사용할 수 없다**.

아래의 예시들은 위에서 언급한 차이점들을 보여준다.

#### 

### 부모 컴포넌트에게 DOM ref를 공개하기

자식 컴포넌트의 DOM 노드에 접근하는 것은 컴포넌트의 캡슐화를 파괴하기 떄문에 권장되지 않으나, 부모 컴포넌트에서 자식 컴포넌트의 DOM 노드에 접근하려 하는 경우도 있다. 

React 16.3 이후 버전의 React는 같은 경우에서 [ref 전달하기(ref forwarding)](https://ko.reactjs.org/docs/forwarding-refs.html)을 사용하는 것이 권장된다. **Ref 전달하기는 컴포넌트가 자식 컴포넌트의 ref를 자신의 ref로서 외부에 노출하도록 한다**. 

1. **DOM 엘리먼트에 Ref 사용하기**

   컴포넌트가 마운트될 때 React는 `current` 프로퍼티에 DOM 엘리먼트를 대입하고, 컴포넌트의 마운트가 해제될 때 `current` 프로퍼티를 다시 `null`로 돌려 놓습니다. `ref`를 수정하는 작업은 `componentDidMount` 또는 `componentDidUpdate` 생명주기 메서드가 호출되기 전에 이루어집니다.

2. **클래스 컴포넌트에 ref 사용하기**

   아래에 있는 `CustomTextInput` 컴포넌트의 인스턴스가 마운트 된 이후에 즉시 클릭되는 걸 흉내내기 위해 `CustomTextInput` 컴포넌트를 감싸는 걸 원한다면, ref를 사용하여 `CustomTextInput` 컴포넌트의 인스턴스에 접근하고 직접 `focusTextInput` 메서드를 호출할 수 있습니다.

   ```jsx
   class AutoFocusTextInput extends React.Component {
     constructor(props) {
       super(props);
       this.textInput = React.createRef();  }
   
     componentDidMount() {
       this.textInput.current.focusTextInput();  }
   
     render() {
       return (
         <CustomTextInput ref={this.textInput} />    );
     }
   }
   ```

   위 코드는 `CustomTextInput`가 클래스 컴포넌트일 때에만 작동한다는 점을 기억하세요.

   ```jsx
   class CustomTextInput extends React.Component {  // ...
   }
   ```

3. **Ref와 함수 컴포넌트**

- 함수 컴포넌트는 인스턴스가 없기 때문에 **함수 컴포넌트에 ref 어트리뷰트를 사용할 수 없습니다**.



### 콜백 ref

React는 ref가 설정되고 해제되는 상황을 세세하게 다룰 수 있는 `콜백 ref` 을 제공한다.

- 콜백 ref를 사용할 때에는 `ref` 어트리뷰트에 `React.createRef()`를 통해 생성된 `ref`를 전달하는 대신, 함수를 전달한다. 

- 전달된 함수는 다른 곳에 저장되고 접근될 수 있는 React 컴포넌트의 인스턴스나 DOM 엘리먼트를 인자로서 받는다.

  

아래의 예시는 DOM 노드의 참조를 인스턴스의 프로퍼티에 저장하기 위해 `ref` 콜백을 사용하는 흔한 패턴이다.

```jsx
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // DOM API를 사용하여 text 타입의 input 엘리먼트를 포커스합니다.
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // 마운트 되었을 때 자동으로 text 타입의 input 엘리먼트를 포커스합니다.
    this.focusTextInput();
  }

  render() {
    // text 타입의 input 엘리먼트의 참조를 인스턴스의 프로퍼티
    // (예를 들어`this.textInput`)에 저장하기 위해 `ref` 콜백을 사용합니다.
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

1) 컴포넌트의 인스턴스가 마운트 될 때 React는 `ref` 콜백을 DOM 엘리먼트와 함께 호출한다. 
2) 그리고 컴포넌트의 인스턴스의 마운트가 해제될 때, `ref` 콜백을 `null`과 함께 호출한다. `ref` 콜백들은 `componentDidMount` 또는 `componentDidUpdate`가 호출되기 전에 호출된다.



콜백 ref 또한 `React.createRef()`를 통해 생성했었던 객체 ref와 같이 다른 컴포넌트에게 전달할 수 있다.

```jsx
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />    
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}      
        />
    );
  }
}
```

위의 예시에서 `Parent`는 자신의 콜백 ref를 `inputRef` prop으로서 `CustomTextInput`에게 전달한다. 그리고 `CustomTextInput`은 전달받은 함수를 `<input>`에게 `ref` 어트리뷰트로서 전달한다. 결과적으로 `Parent`에 있는 `this.inputElement`는 `CustomTextInput`의 `<input>` 엘리먼트에 대응하는 DOM 노드가 된다.

### 

### 레거시 API: 문자열 ref

문자열 ref는 [몇몇 문제](https://github.com/facebook/react/pull/8333#issuecomment-271648615)를 가지고 있고, 레거시로 여겨지며, **차후 배포에서 삭제될 것으로 예상**되기 때문에 권장되지 않는다.

> *주의
>
> ref에 접근하기 위해 `this.refs.textInput`를 사용하고 계신다면, [콜백 ref](https://ko.reactjs.org/docs/refs-and-the-dom.html#callback-refs)이나 [`createRef` API](https://ko.reactjs.org/docs/refs-and-the-dom.html#creating-refs)를 대신 사용하는 것을 권해 드립니다.

### 

### 콜백 ref에 관한 주의사항

`ref` 콜백이 인라인 함수로 선언되있다면 `ref` 콜백은 업데이트 과정 중에 처음에는 `null`로, 그 다음에는 DOM 엘리먼트로, 총 두 번 호출된다. 이러한 현상은 매 렌더링마다 `ref` 콜백의 새 인스턴스가 생성되므로 React가 이전에 사용된 ref를 제거하고 새 ref를 설정해야 하기 때문에 일어난다.

 이 현상은 `ref` 콜백을 클래스에 바인딩된 메서드로 선언함으로써 해결할 수 있다. 하지만 많은 경우 이러한 현상은 문제가 되지 않는다는 점을 기억하자.

