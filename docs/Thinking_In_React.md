# React로 사고하기

> https://ko.reactjs.org/docs/thinking-in-react.html



## 목업으로 시작하기

JSON API와 목업을 디자이너로부터 받았다고 가정하자.

![목업](https://ko.reactjs.org/static/1071fbcc9eed01fddc115b41e193ec11/d4770/thinking-in-react-mock.png)

JSON API는 아래와 같은 데이터를 반환한다.

```js
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```



## 1단계: UI를 컴포넌트 계층 구조로 나누기

- **설계**

  컴포넌트의 주변의 박스를 그려 구조를 설계한다. 디자이너의 Photoshop 레이어 이름이 React 컴포넌트의 이름이 될 수 있다.

- **책임**

  컴포넌트는 [단일 책임 원칙](https://ko.wikipedia.org/wiki/단일_책임_원칙)(하나의 컴포넌트는 한 가지 일을 하는게 이상적이라는 원칙)을 따르는 것이 좋다.  하나의 컴포넌트가 커지게 된다면 이는 보다 작은 하위 컴포넌트로 분리되어야 한다.

- **데이터 모델의 한 조각**

  :각 컴포넌트가 데이터 모델의 한 조각을 나타내도록 분리하자. 주로 JSON 데이터를 유저에게 보여주기 때문에, 데이터 모델이 적절하게 만들어졌다면, UI(컴포넌트 구조)가 잘 연결될 것이다. 이는 UI와 데이터 모델이 같은 *인포메이션 아키텍처(information architecture)*를 가지는 경향이 있기 때문이다.

![Diagram showing nesting of components](https://ko.reactjs.org/static/9381f09e609723a8bb6e4ba1a7713b46/90cbd/thinking-in-react-components.png)

> 1. **`FilterableProductTable`(노란색)**: 예시 전체를 포괄합니다.
> 2. **`SearchBar`(파란색)**: 모든 *유저의 입력(user input)* 을 받습니다.
> 3. **`ProductTable`(연두색)**: *유저의 입력(user input)*을 기반으로 *데이터 콜렉션(data collection)*을 필터링 해서 보여줍니다.
> 4. **`ProductCategoryRow`(하늘색)**: 각 *카테고리(category)*의 헤더를 보여줍니다.
> 5. **`ProductRow`(빨강색)**: 각각의 *제품(product)*에 해당하는 행을 보여줍니다.



## 2단계: React로 정적인 버전 만들기

### props로 데이터 전달하기

props는 부모가 자식에게 데이터를 넘겨줄 때 사용할 수 있는 방법입니다. 정적 버전을 만들기 위해 **state를 사용하지 말자**. state는 오직 상호작용을 위해, 즉 시간이 지남에 따라 데이터가 바뀌는 것에 사용해야 한다.



### 상향식 vs 하향식 

앱을 만들 때 하향식(top-down)이나 상향식(bottom-up)으로 만들 수 있다. 

- 상향식: 상층부에 있는 컴포넌트 (즉 `FilterableProductTable`부터 시작하는 것)부터 만들거나 
- 하향식: 하층부에 있는 컴포넌트 (`ProductRow`) 부터 만들 수도 있습니다. 

간단한 예시에서는 보통 하향식으로 만드는 게 쉽지만 프로젝트가 커지면 상향식으로 만들고 테스트를 작성하면서 개발하기가 더 쉽다.



### 단방향 데이터 흐름(one-way data flow)

이 단계가 끝나면 데이터 렌더링을 위해 만들어진 재사용 가능한 컴포넌트들의 라이브러리를 가지게 된다. 계층구조의 최상단 컴포넌트 (`FilterableProductTable`)는 prop으로 데이터 모델을 받는다. 데이터 모델이 변경되면 `ReactDOM.render()`를 다시 호출해서 UI가 업데이트 된다. 

UI가 어떻게 업데이트되고 어디에서 변경해야하는지 알 수 있다. React의 **단방향 데이터 흐름(one-way data flow)** (또는 *단방향 바인딩(one-way binding*))는 모든 것을 모듈화 하고 빠르게 만들어준다.



### 참고) Props vs State

[`props`](https://ko.reactjs.org/docs/components-and-props.html) (“properties”의 줄임말) 와 [`state`](https://ko.reactjs.org/docs/state-and-lifecycle.html) 는 일반 JavaScript 객체이다.

- `props`: (함수 매개변수처럼) 컴포넌트에 전달 
- `state`: (함수 내에 선언된 변수처럼) 컴포넌트 *안에서* 관리



## 3단계: UI state에 대한 최소한의 (하지만 완전한) 표현 찾아내기

UI를 상호작용하게 만들려면 기반 데이터 모델을 변경할 수 있는 방법이 있어야 한다. 이를 React는 **state**를 통해 변경한다.



### 중복 배제 원칙(Don't repeat yourself; DRY)

애플리케이션이 필요로 하는 가장 최소한의 state를 찾고 이를 통해 나머지 모든 것들이 필요에 따라 그때그때 계산되도록 만들자. 예를 들어 TODO 리스트를 만든다고 하면, TODO 아이템을 저장하는 배열만 유지하고 TODO 아이템의 개수를 표현하는 state를 별도로 만들지 않는다. TODO 갯수를 렌더링해야한다면 TODO 아이템 배열의 길이를 가져오면 된다.



### State 정하기

애플리케이션은 다음과 같은 데이터를 가지고 있다.

- 제품의 원본 목록 
- 유저가 입력한 검색어
- 체크박스의 값
- 필터링 된 제품들의 목록



state는 아래 세 가지 질문을 통해 결정할 수 있다.

> 1. 부모로부터 props를 통해 전달됩니까?  state가 아닙니다.
> 2. 시간이 지나도 변하지 않나요? state가 아닙니다.
> 3. 컴포넌트 안의 다른 state나 props를 가지고 계산 가능한가요? state가 아닙니다.
>
> 따라서 아래는 state가 아니다.
>
> - 제품의 원본 목록 (state 아님. props를 통해 전달됨)
> - 필터링 된 제품들의 목록 (state 아님. 제품의 원본 목록과 검색어, 체크박스의 값을 조합해서 계산해낼 수 있음)



결과적으로 애플리케이션은 다음과 같은 state를 가진다.

- **유저가 입력한 검색어**
- **체크박스의 값**



## 4단계: State가 어디에 있어야 할 지 찾기

React는 항상 컴포넌트 계층구조를 따라 아래로 내려가는 단방향 데이터 흐름을 따른다. 어떤 컴포넌트가 state를 변경하거나 **소유**할지 찾아야 한다.



### 단방향 데이터 흐름을 고려하여 state 위치 찾기

> <전략>
>
> - state를 기반으로 렌더링하는 모든 컴포넌트를 찾으세요.
> - 공통 소유 컴포넌트 (common owner component)를 찾으세요. (계층 구조 내에서 특정 state가 있어야 하는 모든 컴포넌트들의 상위에 있는 하나의 컴포넌트).
> - 공통 혹은 더 상위에 있는 컴포넌트가 state를 가져야 합니다.
> - state를 소유할 적절한 컴포넌트를 찾지 못하였다면, state를 소유하는 컴포넌트를 하나 만들어서 공통 오너 컴포넌트의 상위 계층에 추가하세요.



이 전략을 애플리케이션에 적용해보자.

> - `ProductTable`은 state에 의존한 상품 리스트의 필터링해야 하고 `SearchBar`는 검색어와 체크박스의 상태를 표시해주어야 합니다.
> - 공통 소유 컴포넌트는 `FilterableProductTable`입니다.
> - 의미상으로도 `FilterableProductTable`이 검색어와 체크박스의 체크 여부를 가지는 것이 타당합니다.





## 5단계: 역방향 데이터 흐름 추가하기

계층 구조의 하단에 있는 폼 컴포넌트에서 `FilterableProductTable`의 state를 업데이트하기 위해서는 지금까지의 흐름과 다른 방향의 데이터 흐름을 만들어야 한다. 

우리는 사용자가 폼을 변경할 때마다 사용자의 입력을 반영할 수 있도록 state를 업데이트 해야 한다. 컴포넌트는 그 자신의 state만 변경할 수 있기 때문에 `FilterableProductTable`는 `SearchBar`에 콜백을 넘겨서 state가 업데이트되어야 할 때마다 호출되도록 한다. `FilterableProductTable`에서 전달된 콜백은 `setState()`를 호출하고 앱이 업데이트될 것이다.

