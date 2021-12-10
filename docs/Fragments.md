# Fragments

> https://ko.reactjs.org/docs/fragments.html

Fragments는 DOM에 별도의 노드를 추가하지 않고 여러 자식을 그룹화할 수 있다.

```js
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

## 동기

컴포넌트가 여러 자식을 반환하는 것은 흔한 패턴이다.

```js
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}
```

렌더링 된 HTML이 유효하려면 `<Columns />`가 여러 `<td>` 엘리먼트만 반환해야 한다.

`<Columns />`의 `render()` 안에 부모 div로 자식들을 감싼다면 렌더링 된 HTML은 유효하지 않다.

```js
class Columns extends React.Component {
  render() {
    return (
      <div>
        <td>Hello</td>
        <td>World</td>
      </div>
    );
  }
}
```

`<Table />`의 출력 결과는 다음과 같다.

```js
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```

Fragments는 이 문제를 해결해준다.

## 사용법

```js
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```

올바른 `<Table />`의 출력 결과는 아래와 같다.

```js
<table>
  <tr>
    <td>Hello</td>
    <td>World</td>
  </tr>
</table>
```

## 단축 문법

Fragments를 선언하는 더 짧고 새로운 문법이 있다. 마치 빈 태그와 같다.

```js
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```

`key` 또는 `어트리뷰트`를 지원하지 않는다는 것을 빼고 다른 엘리먼트처럼 `<></>`을 사용할 수 있다.

## key가 있는 Fragments

Fragments에 key가 있다면 `<React.Fragment>` 문법으로 명시적으로 선언해야 한다.

```js
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // React는 `key`가 없으면 key warning을 발생합니다.
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

key는 Fragment에 전달할 수 있는 유일한 어트리뷰트이다. 추후 미래에 이벤트 핸들러와 같은 추가적인 어트리뷰트를 지원할 수도 있다.