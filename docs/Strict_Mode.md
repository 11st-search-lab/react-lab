## Strict Mode

> https://ko.reactjs.org/docs/strict-mode.html

- StrictMode는 애플리케이션 내의 잠재적인 문제를 알아내기 위한 도구이다. 
- Fragment와 같이 UI를 렌더링하지 않는다.
- 자손들에 대한 부가적인 검사와 경고를 활성화한다.
- **Strict 모드는 개발 모드에서만 활성화되기 때문에, 프로덕션 빌드에는 영향을 끼치지 않는다.**

사용 예시:

```ts
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```

- Header와 Footer 컴포넌트는 Strict 모드 검사가 이루지지 않는다.
- ComponentOne과 ComponentTwo는 자식까지 검사가 이루어진다.

StrictMode는 아래와 같은 부분에서 도움이 된다.

- 안전하지 않은 생명주기를 사용하는 컴포넌트 발견
- 레거시 문자열 ref 사용에 대한 경고
- 권장되지 않는 findDOMNode 사용에 대한 경고
- 예상치 못한 부작용 검사
- 레거시 context API 검사

React의 향후 릴리즈에서 더 많은 기능이 더해질 예정이다.

## 안전하지 않은 생명주기를 사용하는 컴포넌트 발견

[블로그 글](https://ko.reactjs.org/blog/2018/03/27/update-on-async-rendering.html)에서 설명하였듯, 비동기 React 애플리케이션에서 특정 생명주기 메서드들은 안전하지 않다.

애플리케이션이 서드 파티 라이브러리를 사용한다면, 해당 생명주기 메서드가 사용되지 않는다고 장담하기 어렵다. Strict 모드는 이러한 경우에 도움이된다.

Strict 모드가 활성화되면, React는 안전하지 않은 생명주기 메서드를 사용하는 모든 클래스 컴포넌트 목록을 정리해 다음과 같이 컴포넌트에 대한 정보가 담긴 경고 로그를 출력한다.

<img src="https://ko.reactjs.org/static/e4fdbff774b356881123e69ad88eda88/1628f/strict-mode-unsafe-lifecycles-warning.png">

## 레거시 문자열 ref 사용에 대한 경고

이전의 React에서 레거시 문자열 ref API와 콜백 API라는, ref를 관리하는 두 가지 방법을 제공했다. 이제는 객체 ref가 문자열 ref를 교체하는 용도로 널리 더해졌기 때문에, Strict 모드는 문자열 ref의 사용에 대해 경고한다.

> 콜백 ref는 새로운 createRef API와 별개로 지속해서 지원될 예정이다.
> 
> 컴포넌트의 콜백 ref를 교체할 필요는 없습니다. 콜백 ref는 조금 더 유연하기 때문에, 고급 기능으로서 계속 지원할 예정이다.

## 권장되지 않는 findDOMNode 사용에 대한 경고

이전의 React에서 주어진 클래스 인스턴스를 바탕으로 트리를 탐색해 DOM 노드를 찾을 수 있는 findDOMNode를 지원했다. DOM 노드에 바로 ref를 지정할 수 있기 때문에 일반적으로 필요하지 않는다.

## 예상치 못한 부작용 검사

개념적으로 React는 두 단계로 동작한다.

- 렌더링 단계는 특정 환경(예를 들어, DOM과 같이)에 어떤 변화가 필요한 지 결정하는 단계이다. 이 과정에서 React는 `render`를 호출하여 이전 렌더와 결과값을 비교한다.
- 커밋 단계는 React가 변경 사항을 반영하는 단계이다(React DOM의 경우 React가 DOM 노드를 추가, 변경 및 제거하는 단계를 말한다). 이 단계에서 React는 `componentDidMount` 나 `componentDidUpdate` 와 같은 생명주기 메서드를 호출한다.

커밋 단계는 일반적으로 매우 빠르지만, 렌더링 단계는 느릴 수 있다.

React는 커밋하기 전에 렌더링 단계의 생명주기 메서드를 여러 번 호출하거나 아예 커밋을 하지 않을 수도(에러 혹은 우선순위에 따른 작업 중단) 있다.

렌더링 단계 생명주기 메서드는 클래스 컴포넌트의 메서드를 포함해 다음과 같다.

- `constructor`
- `componentWillMount` (or `UNSAFE_componentWillMount`)
- `componentWillReceiveProps` (or `UNSAFE_componentWillReceiveProps`)
- `componentWillUpdate` (or `UNSAFE_componentWillUpdate`)
- `getDerivedStateFromProps`
- `shouldComponentUpdate`
- `render`
- `setState` 업데이트 함수 (첫 번째 인자)

위의 메서드들은 여러 번 호출될 수 있기 때문에, 부작용을 포함하지 않는 것이 중요하다. 이 규칙을 무시할 경우, 메모리 누수 혹은 잘못된 애플리케이션 상태 등 다양한 문제를 일으킬 가능성이 있다.

보통 이러한 문제들은 예측한 대로 동작하지 않기 때문에 발견하는 것이 어려울 수 있다. 

Strict 모드가 자동으로 부작용을 찾아주는 것은 불가능하지만, 조금 더 예측할 수 있게끔 만들어서 문제가 되는 부분을 발견할 수 있게 도와줄 수 있다.

이는 아래의 함수를 의도적으로 이중으로 호출하여 찾을 수 있다.

- 클래스 컴포넌트의 `constructor`, `render` 그리고 `shouldComponentUpdate` 메서드
- 클래스 컴포넌트의 `getDerivedStateFromProps` static 메서드
- 함수 컴포넌트 바디
- State updater 함수 (`setState의` 첫 번째 인자)
- `useState`, `useMemo` 그리고 `useReducer에` 전달되는 함수

개발 모드에서만 적용된다. 생명주기 메서드들은 프로덕션 모드에서 이중으로 호출되지 않는다.

## 레거시 context API 검사

레거시 context API는 오류가 발생하기 쉬워 이후 릴리즈에서 삭제될 예정이다. 모든 16.x 버전에서 여전히 돌아가지만, Strict 모드에서는 아래와 같은 경고 메시지를 노출한다.

<img src="https://ko.reactjs.org/static/fca5c5e1fb2ef2e2d59afb100b432c12/51800/warn-legacy-context-in-strict-mode.png">

