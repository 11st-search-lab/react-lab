# JSX In Depth

https://ko.reactjs.org/docs/jsx-in-depth.html

> 근본적으로, JSX는 `React.createElement(component, props, ...children)` 함수에 대한 문법적 설탕을 제공할 뿐이다.

```js
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

위의 코드는 아래와 같이 컴파일된다.

```js
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

자식 컴포넌트가 없다면 아래와 같이 자기 자신을 닫는 형태의 태그를 쓸 수 있다.

```js
<div className="sidebar" />
```

```js
React.createElement(
  'div',
  {className: 'sidebar'}
)
```

특정 JSX가 어떻게 JavaScript로 변환되는지 시험해보고 싶다면 온라인 [babel 컴파일러](https://babeljs.io/repl/#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=react&prettier=false&targets=&version=7.16.1&externalPlugins=&assumptions=%7B%7D)를 사용해보자.

## React Element의 타입 지정하기

JSX 태그의 첫 부분은 React element의 타입을 결정한다.

대문자로 시작하는 JSX 태그는 React 컴포넌트를 지정한다. 이 태그들은 같은 이름을 가진 변수들을 직접 참조한다.

### React가 스코프 내에 존재해야한다.

JSX는 React.createElement를 호출하는 코드로 컴파일 되기 때문에 React 라이브러리 역시 JSX 코드와 같은 스코프 내에 존재해야만 한다.

```js
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null);
  return <CustomButton color="red" />;
}
```

### JSX 타입을 위한 점 표기법 사용

JSX 내에서도 점 표기법을 사용하여 React 컴포넌트를 참조할 수 있다.

```js
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```

### 사용자 정의 컴포넌트는 반드시 대문자로 시작해야한다.

```js
import React from 'react';

// 잘못된 사용법입니다! 아래는 컴포넌트이므로 대문자화 해야 합니다.
function hello(props) {
  // 올바른 사용법입니다! 아래의 <div> 사용법은 유효한 HTML 태그이기 때문에 유효합니다.
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // 잘못된 사용법입니다! React는 <hello />가 대문자가 아니기 때문에 HTML 태그로 인식하게 됩니다.
  return <hello toWhat="World" />;
}
```

### 실행 중에 타입 선택하기

```js
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 올바른 사용법입니다! 대문자로 시작하는 변수는 JSX 타입으로 사용할 수 있습니다.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

## JSX 안에서의 prop 사용

JavaScript 표현식은 JSX의 `{}` 안에서 사용할 수 있다(자바스크립트에서 **표현식이란 값을 반환하는 식 또는 코드**이다).

```js
<MyComponent foo={1 + 2 + 3 + 4} />
```

### 문자열 리터럴

아래의 두 JSX 표현은 동일한 표현이다.

```js
<MyComponent message="hello world" />
<MyComponent message={'hello world'} />
```

문자열 리터럴을 넘겨줄 때, 그 값은 HTML 이스케이프 처리가 되지 않는다. 아래의 두 JSX 표현은 동일한 표현이다.

```js
<MyComponent message="&lt;3" />
<MyComponent message={'<3'} />
```

### Props의 기본값은 “True”

Prop에 어떤 값도 넘기지 않을 경우, 기본값은 `true`이다. 아래의 두 JSX 표현은 동일한 표현이다.

```js
<MyTextBox autocomplete />
<MyTextBox autocomplete={true} />
```

### 속성 펼치기

props에 해당하는 객체를 이미 가지고 있다면,...를 “전개” 연산자로 사용해 전체 객체를 그대로 넘겨줄 수 있다. 아래의 두 컴포넌트는 동일하다.

```js
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

컴포넌트가 사용하게 될 특정 prop을 선택하고 나머지 prop은 전개 연산자를 통해 넘길 수 있다.

```js
const Button = props => {
  const { kind, ...other } = props;
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};

const App = () => {
  return (
    <div>
      <Button kind="primary" onClick={() => console.log("clicked!")}>
        Hello World!
      </Button>
    </div>
  );
};
```

## JSX에서 자식 다루기

여는 태그와 닫는 태그가 있는 JSX 표현에서 두 태그 사이의 내용은 props.children이라는 특수한 prop으로 넘겨진다.

### 문자열 리터럴

```js
<MyComponent>Hello world!</MyComponent>
```

`MyComponent`의 `props.children`은 `"Hello world!"`이다. HTML은 이스케이프 처리가 되지 않으며, 일반적으로 아래와 같이 HTML을 쓰는 방식으로 JSX를 쓸 수 있다.

```js
<div>This is valid HTML &amp; JSX at the same time.</div>
```

JSX는 각 줄의 처음과 끝에 있는 공백을 제거한다. 빈 줄 역시 제거한다. 태그에 붙어있는 개행도 제거되며 문자열 리터럴 중간에 있는 개행은 한 개의 공백으로 대체된다. 따라서 아래의 예시들은 전부 똑같이 렌더링된다.

```js
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

### JSX를 자식으로 사용하기

```js
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```

React 컴포넌트는 element로 이루어진 배열을 반환할 수 있다.

```js
render() {
  // 리스트 아이템들을 추가적인 엘리먼트로 둘러쌀 필요 없습니다!
  return [
    // key 지정을 잊지 마세요 :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

아래와 같이 컴파일된다.

```js
function hello() {
  return [
  /*#__PURE__*/
  // key 지정을 잊지 마세요 :)
  React.createElement("li", {
    key: "A"
  }, "First item"), /*#__PURE__*/React.createElement("li", {
    key: "B"
  }, "Second item"), /*#__PURE__*/React.createElement("li", {
    key: "C"
  }, "Third item")];
  ;
}
```

### JavaScript 표현식을 자식으로 사용하기

```js
<MyComponent>foo</MyComponent>
<MyComponent>{'foo'}</MyComponent>
```

```js
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

```js
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```

### 함수를 자식으로 사용하기

보통 JSX에 삽입된 JavaScript 표현식은 문자열, React element 혹은 이들의 배열로 환산된다. 

`props.children`은 어떤 형태의 데이터도 넘겨질 수 있다.

```js
// 자식 콜백인 numTimes를 호출하여 반복되는 컴포넌트를 생성합니다.
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```

### boolean, null, undefined는 무시된다.

`false`, `null`, `undefined`와 `true`는 유효한 자식이다. 그저 렌더링 되지 않을 뿐이다. 아래의 JSX 표현식들은 동일하게 렌더링된다.

```js
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

이는 React element들을 조건부 렌더링할 때 유용하다.

```js
<div>
  {showHeader && <Header />}
  <Content />
</div>
```

한 가지 주의해야 할 점은 0과 같은 “falsy” 값들은 React가 렌더링 한다.

`props.messages`가 빈 배열일 때 예상과는 다르게 `0`을 출력한다.

```js
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>
```

&& 앞의 표현식이 언제나 진리값이 되도록 해야한다.

```js
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
```

false, true, null 또는 undefined와 같은 값들을 출력하고 싶다면 먼저 문자열로 전환 해야한다.

```js
<div>
  My JavaScript variable is {String(myVariable)}.
</div>
```