# 지역 상태와 전역 상태 사용하기

## 리액트 컴포넌트의 지역 상태

- 리액트 컴포넌트는 트리 구조를 구성
- 트리 구조 내에서 하위 트리 내에 상태를 만들기 위해서는 상위 컴포넌트에서 지역 상태를 만들고, 컴포넌트와 자식 컴포넌트에 props로 해당 상태를 사용하면 됨
- 이 방법은 지역성과 재사용성 측면에서 좋기 때문에 보통 이러한 전략을 따르는 것이 좋음

## 리액트 컴포넌트의 전역 상태

- 특정 상황에서는 트리 내 서로 멀리 떨어져 있는 둘 이상의 컴포넌트에 공통적인 상태가 필요한 경우가 있음
- 이러한 경우에 전역 상태를 사용하는데, 전역 상태는 지역 상태와는 다르게 개념적으로 특정 컴포넌트에 속해 있지 않으므로 전역 상태를 저장하는 위치 고려가 필요함

## 함수의 동작과 리액트 컴포넌트

- 자바스크립트 함수는 크게 순수 함수와 비순수 함수로 나눌 수 있음
- 순수 함수는 오직 인수에만 의존하며 동일한 인수를 받은 경우 동일한 값을 반환함
- 상태는 인수 외부의 값을 말하며 상태에 의존하는 함수는 순수하지 않게 됨
- 리액트 컴포넌트도 자바스크립트 함수이기 때문에 순수할 수 있지만 컴포넌트 내에서 상태를 사용할 경우 순수하지 않게됨
- 하지만 상태가 컴포넌트 내에서만 사용된다면 다른 컴포넌트에 영향을 미치지 않음. 억제됨(contained) 이라고도 함

### 리액트 컴포넌트와 props

- 리액트는 개념적으로 상태를 사용자 인터페이스(UI)로 변환하는 함수임
- 리액트 컴포넌트는 말 그대로 자바스크립트 함수이며 그것의 인수를 props라고 함

```jsx
const AddOne = ({ number }) => {
  return <div>{number + 1}</div>;
};
```

- number를 받아 number + 1을 반환함. 동일한 인수를 받은 경우 동일한 값을 반환하는 순수 함수

### 지역 상태에 대한 useState

```jsx
const AddBase = ({ number }) => {
  const [base, setBase] = useState(1);
  return <div>{number + base}</div>;
};
```

- 이 함수는 인수에 포함되지 않은 base에 의존하기 때문에 순수하지 않음
- 하지만 변경되지 않는한 base를 반환하므로 멱등성을 지님
- 함수 밖에서 base를 변경하는 것은 불가능하므로 억제되어있음

### 지역 상태의 한계

- 지역성을 제공하고 싶지 않을 때는 지역 상태를 사용하는 것이 적절하지 않음
- 함수 컴포넌트 외부에서 상태를 변경해야 한다면 전역 상태가 필요함

## 지역 상태를 사용하는 패턴

### 상태 끌어올리기(Lifting State Up)

```jsx
const Component1 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>click</button>
    </div>
  );
};

const Component2 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>click</button>
    </div>
  );
};

const Parent = () => {
  const [count, setCount] = useState(0);

  return (
    <>
      <Component1 state={state} setState={setState} />
      <Component2 state={state} setState={setState} />
    </>
  );
};
```

- count 상태는 Parent에 단 한 번만 정의되기 때문에 상태는 Component1과 Component2에 공유됨
- count는 여전히 컴포넌트 내에 지역상태로 존재하며, 자식 컴포넌트는 부모 컴포넌트의 상태를 이용할 수 있음
  <br />
- 상태를 상위 컴포넌트로 전달할 경우 Parent는 모든 자식 컴포넌트를 포함해 하위 트리 전체를 리렌더링함. 이것은 일부 상황에서 성능 이슈를 일으킬 수 있음

### 내용 끌어올리기(Lifting content Up)

- 복잡한 컴포넌트 트리라면 상위 컴포넌트의 상태에 의존하지 않는 컴포넌트가 있을 수 있음

```jsx
const Component1 = ({ count, setCount, children }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>click</button>
      {children}
    </div>
  );
};

const Component2 = ({ count, setCount }) => {
  return (
    <div>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>click</button>
    </div>
  );
};

const Parent = ({ children }) => {
  const [count, setCount] = useState(0);

  return (
    <>
      <Component1 state={state} setState={setState}>
        {children}
      </Component1>
      <Component2 state={state} setState={setState} />
    </>
  );
};

const GrandParent = () => {
  return (
    <Parent>
      <AdditionalInfo />
    </Parent>
  );
};
```

- GrandParent컴포넌트는 자식에게 전달되는 AdditionalInfo요소를 가짐
- AdditionalInfo 컴포넌트는 count가 변경될 때 리렌더링되지 않음
- children은 jsx 형식으로 중첨된 자식 요소를 표현하는 특별한 prop 이름임

## 전역 상태 사용하기

- 개념적으로 하나의 컴포넌트에만 속하고 컴포넌트에 의해 캡슐화된 상태를 지역 상태
- 상태가 하나의 컴포넌트에만 속하지 않고 여러 컴포넌트에서 사용할 수 있다면 전역 상태

### 싱글턴으로의 전역 상태

- 특정 컨텍스트에서 상태가 하나의 값을 가지고 있다는 것을 의미
- 메모리에 하나의 값으로만 존재

### 공유 상태로의 전역 상태

- 상태 값이 다른 컴포넌트 간에 공유된다는 것을 의미하지만, 자바스크립트 메모리상에서 단일 값일 필요는 없음
- 컴포넌트 트리의 다른 부분(하위 트리)에 대해 여러 값을 가질 수 있음

## 전역 상태가 필요한 경우

### prop을 전달하는 것이 적절하지 않을 때

- 상위 컴포넌트의 상태를 깊은 뎁스의 컴포넌트로 전달할 때 중간 단계의 뎁스는 해당 상태를 직접 사용하지 않음에도 불구하고 prop으로 전달받아야함 (비효율적인 props drilling 발생)
- 비효율적인 개발 경험
- 상위 컴포넌트의 상태가 변경되면 중간 뎁스의 컴포넌트도 리렌더링 되어 성능에 영향을 미침

### 이미 리액트 외부에 상태가 있을 때

- 어떤 경우에는 외부에 전역 상태를 두는 편이 더 간단함
- 예) 애플리케이션에서 리액트 없이 획득한 사용자 인증 정보가 있을 떄

전역 상태는 컴포넌트의 동작을 예측하기 어렵게 하는 단점이 있기 때문에, 과하게 사용하지 않는 것이 좋음. 지역 상태를 기본으로 사용하고, 전역 상태를 보조 수단으로 사용하는 것을 권장
