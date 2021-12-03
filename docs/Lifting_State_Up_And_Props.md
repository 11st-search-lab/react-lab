# State 끌어올리기

> https://ko.reactjs.org/docs/lifting-state-up.html



동일한 데이터에 대한 변경사항을 여러 컴포넌트에 반영해야 할 때는 가장 가까운 공통 조상으로 state를 끌어올리는 것이 좋다.

예를 들어 주어진 온도에서 물의 끓는 여부를 추정하는 온도 계산기를 만든다고 할 때 섭씨온도 입력값을 변경할 경우 화씨온도 입력값 역시 변환된 온도를 반영할 수 있어야 하며, 그 반대의 경우에도 가능해야한다. 즉 두 입력값이 서로의 것과 동기화된 상태로 있어야한다.

React에서는 컴포넌트 간의 가장 가까운 공통 조상으로 state를 끌어올림으로써 state를 공유할 수 있다. 이를 `state 끌어올리기`라고 한다. 

`TemperatureInput`이 개별적으로 가지고 있던 지역 state를 지우는 대신 `Calculator`로 그 값을 옮긴다.

`Calculator`가 공유될 state를 소유하고 있으면 이 컴포넌트는 두 입력 필드의 현재 온도에 대한 `source of truth`가 된다. 이를 통해 두 입력 필드가 서로 간에 일관된 값을 유지하도록 만들 수 있다. 두 `TemperatureInput` 컴포넌트의 props가 같은 부모인 `Calculator`로부터 전달되기 때문에, 두 입력 필드는 항상 동기화된 상태를 유지할 수 있다.

[props는 읽기 전용](https://ko.reactjs.org/docs/components-and-props.html#props-are-read-only)이다. `temperature`가 지역 state였을 때는 그 값을 변경하기 위해서 그저 `TemperatureInput`의 `this.setState()`를 호출하는 걸로 충분했다. 하지만, 이제 `temperature`가 부모로부터 prop로 전달되기 때문에 `TemperatureInput`은 그 값을 제어할 능력이 없다.

React에서는 보통 이 문제를 컴포넌트를 `제어` 가능하게 만드는 방식으로 해결한다. DOM `<input>`이 `value`와 `onChange` prop를 건네받는 것과 비슷한 방식으로, 사용자 정의된 `TemperatureInput` 역시 `temperature`와 `onTemperatureChange` props를 자신의 부모인 `Calculator`로부터 건네받을 수 있다. 이제 `TemperatureInput`에서 온도를 갱신하고 싶으면 `onTemperatureChange  `props를 호출하면 된다.

두 입력 필드의 값이 동일한 state로부터 계산되기 때문에 이 둘은 항상 동기화된 상태를 유지하게 된다.

```jsx
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = {temperature: '', scale: 'c'};  }

  handleCelsiusChange(temperature) {
    this.setState({scale: 'c', temperature});  }

  handleFahrenheitChange(temperature) {
    this.setState({scale: 'f', temperature});  }

  render() {
    const scale = this.state.scale;    
    const temperature = this.state.temperature;    
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;  
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;
    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}          
          onTemperatureChange={this.handleCelsiusChange} />        
        <TemperatureInput
          scale="f"
          temperature={fahrenheit}          
          onTemperatureChange={this.handleFahrenheitChange} />        
        <BoilingVerdict
          celsius={parseFloat(celsius)} />      
      </div>
    );
  }
}
```

이제 어떤 입력 필드를 수정하든 간에 `Calculator`의 `this.state.temperature`와 `this.state.scale`이 갱신된다. 입력 필드 중 하나는 있는 그대로의 값을 받으므로 사용자가 입력한 값이 보존되고, 다른 입력 필드의 값은 항상 다른 하나에 기반해 재계산된다.

<순서>

1. React는 DOM `<input>`의 `onChange`에 지정된 함수를 호출한다. 위 예시의 경우 `TemperatureInput`의 `handleChange` 메서드에 해당한다.

2. `TemperatureInput` 컴포넌트의 `handleChange` 메서드는 새로 입력된 값과 함께 `this.props.onTemperatureChange()`를 호출한다.

   - `onTemperatureChange`를 포함한 이 컴포넌트의 props는 부모 컴포넌트인 `Calculator`로부터 제공받은 것.

3. 이전 렌더링 단계에서, `Calculator`는 아래와 같이 지정해놓았다. 따라서 우리가 둘 중에 어떤 입력 필드를 수정하느냐에 따라서 `Calculator`의 두 메서드 중 하나가 호출된다.

   - 섭씨 `TemperatureInput`의 `onTemperatureChange`를 `Calculator`의 `handleCelsiusChange` 메서드로
   - 화씨 `TemperatureInput`의 `onTemperatureChange`를 `Calculator`의 `handleFahrenheitChange` 메서드로

4. 이들 메서드는 내부적으로 `Calculator` 컴포넌트가 새 입력값, 그리고 현재 수정한 입력 필드의 입력 단위와 함께 `this.setState()`를 호출하게 함으로써 React에게 자신을 다시 렌더링하도록 요청한다.

5. React는 UI가 어떻게 보여야 하는지 알아내기 위해 `Calculator` 컴포넌트의 `render` 메서드를 호출 

   - 두 입력 필드의 값은 현재 온도와 활성화된 단위를 기반으로 재계산
   - 온도 변환

6. React는 `Calculator`가 전달한 새 props와 함께 각 `TemperatureInput` 컴포넌트의 `render` 메서드를 호출하며 UI가 어떻게 보여야 할지를 파악한다.

7. React는 `BoilingVerdict` 컴포넌트에게 섭씨온도를 props로 건네면서 그 컴포넌트의 `render` 메서드를 호출한다.

8. React DOM은 물의 끓는 여부와 올바른 입력값을 일치시키는 작업과 함께 DOM을 갱신한다. 값을 변경한 입력 필드는 현재 입력값을 그대로 받고, 다른 입력 필드는 변환된 온도 값으로 갱신된다.

   

입력 필드의 값을 변경할 때마다 동일한 절차를 거치고 두 입력 필드는 동기화된 상태로 유지된다.

## 교훈

React 애플리케이션 안에서 변경이 일어나는 데이터에 대해서는 `source of truth`을 하나만 두어야 한다. 보통의 경우, state는 렌더링에 그 값을 필요로 하는 컴포넌트에 먼저 추가된다. 그러고 나서 다른 컴포넌트도 역시 그 값이 필요하게 되면 그 값을 그들의 가장 가까운 공통 조상으로 끌어올리면 된다. 다른 컴포넌트 간에 존재하는 state를 동기화시키려고 노력하는 대신 [하향식 데이터 흐름](https://ko.reactjs.org/docs/state-and-lifecycle.html#the-data-flows-down)을 유지하는 것을 추천한다.

> *하향식(top-down) 또는 단방향식 데이터 흐름
>
> 모든 state는 항상 특정한 컴포넌트가 소유하고 있으며 그 state로부터 파생된 UI 또는 데이터는 오직 트리구조에서 자신의 '아래에 있는 컴포넌트에만 영향을 미칩니다.

- state를 끌어올리는 작업은 양방향 바인딩 접근 방식보다 더 많은 '보일러 플레이트' 코드를 유발하지만, 버그를 찾고 격리하기 더 쉽게 만든다는 장점이 있다. 

- 어떤 state든 간에 특정 컴포넌트 안에서 존재하기 마련이고 그 컴포넌트가 자신의 state를 스스로 변경할 수 있으므로 버그가 존재할 수 있는 범위가 크게 줄어든다. 

- 사용자의 입력을 거부하거나 변형하는 자체 로직을 구현할 수도 있다.

  

어떤 값이 props 또는 state로부터 계산될 수 있다면, 아마도 그 값을 state에 두어서는 안 된다. 예를 들어 `celsiusValue`와 `fahrenheitValue`를 둘 다 저장하는 대신, 단지 최근에 변경된 `temperature`와 `scale`만 저장하면 된다. 다른 입력 필드의 값은 항상 그 값들에 기반해서 `render()` 메서드 안에서 계산될 수 있다. 이를 통해 사용자 입력값의 정밀도를 유지한 채 다른 필드의 입력값에 반올림을 지우거나 적용할 수 있게 된다.

UI에서 무언가 잘못된 부분이 있을 경우, [React Developer Tools](https://github.com/facebook/react/tree/main/packages/react-devtools)를 이용하여 props를 검사하고 state를 갱신할 책임이 있는 컴포넌트를 찾을 때까지 트리를 따라 탐색함으로써 소스 코드에서 버그를 추적할 수 있다.