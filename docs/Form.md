# Form

> https://ko.reactjs.org/docs/forms.html

HTML 폼 엘리먼트는 폼 엘리먼트 자체가 내부 상태를 가지기 때문에, React의 다른 DOM 엘리먼트와 다르게 동작한다.

예를 들어, 순수한 HTML에서 이 폼은 name을 입력받는다.

```html
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

이 폼은 사용자가 폼을 제출하면 새로운 페이지로 이동하는 기본 HTML 폼 동작을 수행한다.

React에서 동일한 동작을 원한다면 그대로 사용하면 되지만, 대부분의 경우, JavaScript 함수로 폼의 제출을 처리하고 사용자가 폼에 입력한 데이터에 접근하도록 하는 것이 편리하다.

이를 위한 표준 방식은 **“제어 컴포넌트 (controlled components)“**라고 불리는 기술을 이용하는 것이다.

## 제어 컴포넌트 (Controlled Component)

HTML에서 `<input>`, `<textarea>`,` <select>`와 같은 폼 엘리먼트는 일반적으로 사용자의 입력을 기반으로 자신의 state를 관리하고 업데이트한다.
React에서는 변경할 수 있는 state가 일반적으로 컴포넌트의 state 속성에 유지되며 setState()에 의해 업데이트한다.

우리는 React state를 “신뢰 가능한 단일 출처 (single source of truth)“로 만들어 두 요소를 결합할 수 있다. 그러면 폼을 렌더링하는 React 컴포넌트는 폼에 발생하는 사용자 입력값을 제어한다. 이러한 방식으로 React에 의해 값이 제어되는 입력 폼 엘리먼트를 “제어 컴포넌트 (controlled component)“라고 한다.

```js
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

제어 컴포넌트로 사용하면, input의 값은 항상 React state에 의해 결정된다.

코드를 조금 더 작성해야 한다는 의미이지만, 다른 UI 엘리먼트에 input의 값을 전달하거나 다른 이벤트 핸들러에서 값을 재설정할 수 있다.

## textarea 태그

HTML에서 `<textarea>` 엘리먼트는 텍스트를 자식으로 정의한다.

```html
<textarea>
  Hello there, this is some text in a text area
</textarea>
```

React에서 `<textarea>`는 value 어트리뷰트를 대신 사용한다. 이렇게하면 `<textarea>`를 사용하는 폼은 한 줄 입력을 사용하는 폼과 비슷하게 작성할 수 있다.

```js
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 'Please write an essay about your favorite DOM element.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('An essay was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Essay:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

## select 태그

HTML에서 <select>는 드롭 다운 목록을 만든다. 예를 들어, 아래 HTML은 과일 드롭 다운 목록을 만든다.

```html
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">Coconut</option>
  <option value="mango">Mango</option>
</select>
```

selected 옵션이 있으므로 Coconut 옵션이 초기값이 되는 점을 주의하자. React에서는 selected 어트리뷰트를 사용하는 대신 최상단 select태그에 value 어트리뷰트를 사용한다.

전반적으로 `<input type="text">`, `<textarea>` 및 `<select>` 모두 매우 비슷하게 동작한다.

### 주의

**select 태그에 multiple 옵션을 허용한다면 value 어트리뷰트에 배열을 전달할 수 있다.**

```html
<select multiple={true} value={['B', 'C']}>
```

## file input 태그

HTML에서 <input type="file">는 사용자가 하나 이상의 파일을 자신의 장치 저장소에서 서버로 업로드하거나 File API를 통해 JavaScript로 조작할 수 있다.

```html
<input type="file" />
```

값이 읽기 전용이기 때문에 React에서는 비제어 컴포넌트이다.

## 다중 입력 제어하기

여러 input 엘리먼트를 제어해야할 때, 각 엘리먼트에 name 어트리뷰트를 추가하고 event.target.name 값을 통해 핸들러가 어떤 작업을 할 지 선택할 수 있게 해준다.

```js
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

## 제어되는 Input Null 값

제어 컴포넌트에 value prop을 지정하면 의도하지 않는 한 사용자가 변경할 수 없다. value를 설정했는데 여전히 수정할 수 있다면 실수로 value를 undefined나 null로 설정했을 수 있다.

 (첫 번째 입력은 잠겨있지만 잠시 후 입력이 가능해진다.)
```js
ReactDOM.render(<input value="hi" />, mountNode);

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode);
}, 1000);
```

## 제어 컴포넌트의 대안

데이터를 변경할 수 있는 모든 방법에 대해 이벤트 핸들러를 작성하고 React 컴포넌트를 통해 모든 입력 상태를 연결해야 하기 때문에 때로는 제어 컴포넌트를 사용하는 게 지루할 수 있다. 특히 기존의 코드베이스를 React로 변경하고자 할 때나 React가 아닌 라이브러리와 React 애플리케이션을 통합하고자 할 때 짜증날 수 있다.

이러한 경우에 입력 폼을 구현하기 위한 대체 기술인 **비제어 컴포넌트**를 사용할 수 있다.

## 완전한 해결책

유효성 검사, 방문한 필드 추적 및 폼 제출 처리와 같은 완벽한 해결을 원한다면 Formik이 대중적인 선택 중 하나이다.