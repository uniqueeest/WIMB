# 리액트 훅을 이용한 마이크로 상태관리

## 상태

- 사용자 인터페이스 (UI)를 나타내는 모든 데이터
- 시간이 지남에 따라 변할 수 있음
- 리액트는 이 상태와 함께 렌더링 할 컴포넌트를 처리함

## 상태 관리

- 기존의 상태 관리는 상태 관리를 위한 범용 프레임워크를 사용하는 중앙 집중적인 방식
- 리액트 훅 (React hook) 등장 이후 상태 관리를 위한 훅을 사용할 수 있으며, 상태 관리를 경량화할 수 있음

## 마이크로 상태 관리

- 범용적인 상태 관리를 위한 방법을 가벼워야 함
- 개발자는 요구사항에 따라 적절한 방법을 선택할 수 있어야 함

### 기본적인 상태 관리 기능

- 상태 읽기
- 상태 갱신
- 상태 기반 렌더링

### 추가적인 기능

- 리렌더링 최적화
- 다른 시스템과의 상호 작용
- 비동기 지원
- 파생 상태
- 간단한 문법 등

## 리액트 훅

- useState 훅은 지역 상태를 생성하는 기본적인 함수로, 로직을 캡슐화하고 재사용 가능하다는 리액트 훅의 특징이 있음. 그래서 useState를 기반으로 다양한 사용자 정의 훅을 만들 수 있음
- useReducer 훅도 지역 상태를 생성할 수 있으며, useState를 대체하는 용도로 자주 사용됨.
- useEffect 훅을 이용하면 리액트 렌더링 프로세스 바깥에서 로직을 실행할 수 있음. (Side Effect 관리가 가능) 특히 전역 상태를 다루기 위한 상태 관리 라이브러리를 개발할 때 중요한데, 그 이유는 리액트 컴포넌트 생명 주기와 함께 작동하는 기능을 구현할 수 있기 때문

리액트 훅은 UI 컴포넌트에서 로직을 추출할 수 있음

```jsx
const useCount = () => {
  const [count, setCount] = useState(0);
  return [count, setCount];
};

const Component = () => {
  const [count, setCount] = useCount();

  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>click</button>
    </div>
  );
};
```

더 복잡해 보일 수 있지만 다른 관점으로 접근 가능하다.

- useCount라는 이름을 통해 더욱 명확해짐 -> 프로그래밍 관점에서 매우 중요함. 코드의 가독성을 높일 수 있음
- Component와 useCount가 분리됨 -> 컴포넌트를 건드리지 않고도 기능을 추가할 수 있다. (마이크로 상태 관리 라이브러리에서도 중요한 부분)

## 리액트 훅 사용 시 주의해야 하는 부분

- 기존 state 객체나 ref 객체를 직접 변경해서는 안됨. why? 직접 변경할 경우 리렌더링 되지 않거나, 너무 많은 리렌더링이 발생하거나, 부분적인 리렌더링이 발생하는 등 에기치 않은 동작 발생이 일어날 수 있음
- 리액트 훅 함수와 컴포넌트 함수는 여러 번 호출될 수 있음. 따라서 함수가 여러 번 호출되더라도 일관되게 동작할 수 있게 '순수'해야 함

## 전역 상태

- 리액트는 컴포넌트에서 정의되고 컴포넌트 트리 내에서 사용되는 상태에 대해 useState와 같은 기본적인 훅을 제공하는데, 이를 흔히 지역 상태라고 함
- 전역 상태는 애플리케이션 내 서로 멀리 떨어져 있는 여러 컴포넌트에서 사용하는 상태
- 싱글턴일 필요는 없으며, 싱글턴이 아니라는 점을 명확히 하기 위해 전역 상태를 공유 상태(shared state)라고 부르기도 함

\*\* 싱글턴 : 프로그램 내 객체가 단 하나만 존재하게 만드는 디자인 패턴. 자바스크립트에서는 전역 변수를 만들 경우 최상위 객체인 window에 포함됨.

\*\* 컴포넌트 모델에 대해 주의할 점 : 컴포넌트는 함수처럼 재사용 가능한 하나의 단위임. 컴포넌트를 한 번 정의하면 여러 번 사용하는 것이 가능한데, 이는 컴포넌트가 독립적인 경우에만 가능. 컴포넌트가 컴포넌트 외부에 의존하는 경우 동작이 일관되지 않을 수 있으므로 재사용 불가 할 수 있음. 따라서 컴포넌트 자체는 전역 상태에 가급적 의존하지 않는 것이 좋음

## useState

### 값으로 상태 갱신하기

- useState가 반환하는 함수에 새로운 값을 전달하면 기존 상태 값이 새로운 값으로 대체됨

```jsx
const [count, setCount] = useState(0);

const handleClick = () => {
  setCount(1);
};
```

- handleClick 함수를 호출하여 count의 값이 1로 변경된 이후 재호출 하면 동일한 값이기 때문에 '베일아웃'되어 컴포넌트가 리렌더링되지 않음

\*\* 베일아웃(bailout) : 리렌더링을 발생시키지 않는 것을 의미

### 함수로 상태 갱신하기

- 갱신 함수를 사용하여 상태를 갱신할 수 있음
- 이전 값을 기반으로 갱신하는 경우에 유용함

```jsx
const [count, setCount] = useState(0);

// (c) => C + 1이 호출되면서 해당 함수를 클릭한 횟수를 센다.
const handleClick = () => {
  setCount((c) => c + 1);
};
```

- 갱신 함수가 이전 상태와 정확하게 동일한 상태를 반환하는 경우 베일아웃이 발생하고, 컴포넌트가 리렌더링되지 않음

### 지연 초기화

- useState는 첫 번째 렌더링에서만 평가되는 초기화 함수를 받을 수 있음

```jsx
const init = () => 0;

const [count, setCount] = useState(init);

const handleClick = () => {
  setCount((c) => c + 1);
};
```

- init 함수가 무거운 계산을 포함할 할 떄 지연 초기화를 사용하는 것이 좋음
- useState가 호출되기 전까지 init 함수는 평가되지 않고 느리게 평가됨. 즉, 컴포넌트가 마운트될 때 한 번만 호출됨

## useReducer

### 기본 사용

- reducer는 복잡한 상태에 유용함
- useReducer를 사용하면 미리 정의된 리듀서 함수와 초기 상태를 매개변수로 받아 리듀서 함수를 정의할 수 있음
- 훅 외부에서 리듀서 함수를 정의하면 코드를 분리할 수 있음
- 순수 함수이기 때문에 동작을 테스트하기가 더 쉬움

```jsx
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    default:
      throw new Error('ERROR!');
  }
};

const [state, dispatch] = useReducer(reducer, {
  count: 0,
});

const handleIncreaseCount = () => {
  dispatch({ type: 'INCREMENT' });
};
```

### 베일아웃

- useReducer에서 베일아웃을 발생시키기 위해서는 state 자체를 반환해야함. 객체를 반환하면 새로운 객체를 생성하기 때문에, 베일아웃이 발생하지 않음

### 원시값

- 객체가 아닌 값, 즉 숫자나 문자열 같은 원시 값에 대해 작동함

### 지연 초기화

- useReducer는 지연 초기화를 위해 init이라는 세 번째 선택적인 매개변수를 받을 수 있다.

```jsx
const init = (count) => ({ count, text: 'hi' });

const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'SET_TEXT':
      return { ...state, text: action.text };
    default:
      throw new Error('ERROR!');
  }
};

const [state, dispatch] = useReducer(reducer, 0, init);
```

## useState의 useReducer의 유사점과 차이점

- 실제로 리액트 내부에서 useState는 useReducer로 구현돼 있음
- useReducer에서만 reducer와 init을 훅이나 컴포넌트 외부에서 정의할 수 있다는 차이점이 있다.
- 인라인 리듀서 함수는 외부 변수에 의존할 수 있음. 이것은 useReducer에서만 가능함

```jsx
// 하지만 일반적으로 사용되지 않음
const useScore = (bonus) =>
  useReducer((prev, delta) => prev + delta + bonus, 0);
```
