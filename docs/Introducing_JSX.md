# Introducing JSX

```js
const element = <h1>Hello, world!</h1>;
```

위의 태그 문법은 문자열도, HTML도 아니다. **JavaScript를 확장한 문법인 `JSX`이다.**

- UI가 어떻게 생겨야 하는지 설명하기 위해 React와 함께 사용할 것을 권장한다.
- `JSX`는 `React element`를 생성한다.


## JSX란?

React에서는 본질적으로 렌더링 로직이 UI 로직과 연결된다는 사실을 받아들인다(이벤트가 처리되는 방식, 시간에 따라 state가 변하는 방식, 화면에 표시하기 위해 데이터가 준비되는 방식 등).

React는 별도의 파일에 마크업과 로직을 넣어 기술을 인위적으로 분리하는 대신, **둘 다 포함하는 “컴포넌트”라고 부르는 느슨하게 연결된 유닛으로 관심사를 분리한다.**

- React에서 JSX 사용은 필수가 아니다.
  - 하지만, 대부분의 사람은 JavaScript 코드 안에서 UI 관련 작업을 할 때 시각적으로 더 도움이 된다고 생각한다.
  - React가 도움이 되는 에러 및 경고 메시지를 표시할 수 있게 할 수 있다.
- [Pete Hunt: React: Rethinking best practices -- JSConf EU](https://www.youtube.com/watch?v=x7cQ3mrcKaY)

## JSX에 표현식 포함하기

```js
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

```js
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

- JSX를 가독성을 위해 여러 줄로 나눌 때, [자동 세미콜론을 피하고자](https://stackoverflow.com/questions/2846283/what-are-the-rules-for-javascripts-automatic-semicolon-insertion-asi) 괄호로 묶는 것을 권장한다.

## JSX도 표현식이다.

**컴파일이 끝나면, JSX 표현식이 정규 JavaScript 함수 호출이 되어서 JavaScript 객체로 인식된다.** 즉, JSX를 if 구문 및 for loop 안에 사용하고, 변수에 할당하고, 인자로서 받아들이고, 함수로부터 반환할 수 있다.


```js
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

## JSX 속성 정의

속성에 따옴표를 이용해 문자열 리터럴을 정의할 수 있다.

```js
const element = <div tabIndex="0"></div>;
```

중괄호를 사용하여 속성에 JavaScript 표현식을 삽입할 수도 있다.

```js
const element = <img src={user.avatarUrl}></img>;
```

- `React DOM`은 HTML 속성 이름 대신 `camelCase` 프로퍼티 명명 규칙을 사용
  - class는 className, tabindex는 tabIndex

## JSX로 자식 정의

태그가 비어있다면 `/>`로 닫아주어야한다.

```js
const element = <img src={user.avatarUrl} />;
```

JSX 태그는 자식을 포함할 수 있다.

```js
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

## JSX는 주입 공격을 방지합니다

JSX에 사용자 입력을 삽입하는 것은 안전하다.

```js
const title = response.potentiallyMaliciousInput;
// 이것은 안전합니다.
const element = <h1>{title}</h1>;
```

기본적으로 React DOM은 JSX에 삽입된 모든 값을 렌더링하기 전에 [이스케이프](https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-in-html) 하므로, 애플리케이션에서 명시적으로 작성되지 않은 내용은 주입되지 않는다. 모든 항목은 렌더링 되기 전에 문자열로 변환된다.

이런 특성으로 인해 [XSS(cross-site-scripting)](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8C%85) 공격을 방지할 수 있다.

## JSX는 객체를 표현합니다.

Babel은 JSX를 React.createElement() 호출로 컴파일한다.

다음 두 예시는 동일하다.

```js
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

```js
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

React.createElement()는 버그가 없는 코드를 작성하는 데 도움이 되도록 몇 가지 검사를 수행하고, 다음과 같은 객체를 생성한다.

```js
// 주의: 다음 구조는 단순화되었습니다
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

**이러한 객체를 `React element`라고 한다.** React는 이 객체를 읽어서, DOM을 구성하고 최신 상태로 유지하는데 사용한다.