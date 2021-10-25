# Introducing JSX

```js
const element = <h1>Hello, world!</h1>;
```

위의 태그 문법은 문자열도, HTML도 아니다. **JavaScript를 확장한 문법인 `JSX`이다.**

- UI가 어떻게 생겨야 하는지 설명하기 위해 React와 함께 사용할 것을 권장한다.
- `JSX`는 `React element`를 생성한다.


## JSX란?

React에서는 본질적으로 렌더링 로직이 UI 로직과 연결된다는 사실을 받아들인다(이벤트가 처리되는 방식, 시간에 따라 state가 변하는 방식, 화면에 표시하기 위해 데이터가 준비되는 방식 등).