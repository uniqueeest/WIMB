# 리액트 컨텍스트를 이용한 컴포넌트 상태 공유

## Context

- props를 대신해서 컴포넌트 간에 데이터를 전달하는 것이 가능
- 컨텍스트를 컴포넌트 상태와 결합하면 전역 상태를 제공할 수 있음
- 햐지만 전역 상태를 위해 설계된 것이 아님. 상태가 갱신될 대 모든 컨텍스트 소비자(consumer)가 리렌더링되므로 불필요한 렌더링이 발생할 수 있음
- 일반적으로 전역 상ㄴ태를 여러 조각으로 나누어 사용하는 것이 권장됨

## useContext

- 리액트 컨텍스트는 props를 제거하는데 유용함
- 컨텍스트를 사용하면 props를 사용하지 않고도 부모 컴포넌트에서 트리 아래에 있는 자식 컴포넌트로 값을 전달하는 것이 가능

```tsx
const ColorContext = createContext('black');

const Component = () => {
  const color = useContext(ColorContext);
  return <div style={{ color }}>Hello {color}</div>;
};

const App = () => {
  return (
    <>
      <Component />
      <ColorContext.Provider value="red">
        <Component />
      </ColorContext.Provider>
      <ColorContext.Provider value="blue">
        <Component />
      </ColorContext.Provider>
    </>
  );
};
```

- 공급자로 둘러싸여 있지 않을 경우 기본값인 black이 정의되고, 나머지는 value 값에 따라 정의됨

## 컨텍스트 이해하기

### 컨텍스트 전파의 작동 방식

- 컨텍스트 공급자를 사용할 경우 컨텍스트 값을 갱신할 수 있음
- 컨텍스트 공급자가 새로운 컨텍스트 값을 받으면 모든 컨텍스트 소비자 컴포넌트가 리렌더링 됨
- 자식 컴포넌트가 리렌더링되는 이유 -> 부모 컴포넌트 or 컨텍스트
- 컨텍스트의 값이 변경되지 않았는데 리렌더링이 발생하는 문제를 방지하기 위해 '내용 끌어올리기' 또는 memo를 사용하는 방법이 있음

\*\* memo가 내부 컨텍스트 소비자가 리렌더링 되는 것을 막지 못함. why? 컴포넌트가 일관되지 않은 컨텍스트 값을 가질 수 있기 때문

### 컨텍스트에 객체를 사용할 때의 한계점

- 객체에는 여러가지 값을 포함할 수 있으며, 컨텍스트 소비자는 모든 값을 사용하지 않을 수 있음

```jsx
const CountContext = createContext({ count1: 0, count2: 0 });
```

- A라는 컴포넌트에서 count1, B라는 컴포넌트에서 count2를 사용한다고 할 때, 각각의 값을 사용하게 되면 사용하지 않는 컴포넌트에서 불필요한 리렌더링이 발생하게 됨

\*\* 추가적인 리렌더링은 기술적으로 피해야 하는 순수하게 불필요한 연산이지만, 보통 사용자가 추가 리렌더링을 알아채지 못하는 경우가 많기 때문에 성능에 큰 문제가 없으면 대체적으로 괜찮음.
몇 번의 추가 리렌더링을 피하기 위해 오버엔지리어닝을 하는 것이 더 불필요 할 수 있음

## 전역 상태를 위한 컨텍스트 만들기

### 작은 상태 조각 만들기

- 전역 상태를 여러 조각으로 나누는 것
- 합쳐진 큰 객체를 사용하는 대신 각 조각에 대한 컨텍스트와 전역 상태를 만들 수 있음

```tsx
type CountContextType = [nubmer, Dispatch<SetStateAction<number>>];

const Count1Context = createContext < CountContextType > [0, () => {}];
const Count2Context = createContext < CountContextType > [0, () => {}];

const Counter1 = () => {
  const [count1, setCount2] = useContext(Count1Context);
  return (
    <div>
      {count1}
      <button onClick={() => setCount1((c) => c + 1)}>+1</button>
    </div>
  );
};

// ...Counter2도 동일하게 적용 (Count2Context 사용)

const Parent = () => {
  return (
    <div>
      <Counter1 />
      <Counter2 />
    </div>
  );
};

const Count1Provider = ({ children }: { children: React.ReactNode }) => {
  const [count1, setCount1] = useState(0);

  return (
    <Count1Context.Provider value={[count1, setCount1]}>
      {children}
    </Count1Context.Provider>
  );
};

// ...Count2Provider도 동일하게 적용 (Count2Context 사용)

const App = () => {
  return (
    <Count2Provider>
      <Count1Provider>
        <Parent />
      </Count1Provider>
    </Count2Provider>
  );
};
```

- 공급자 컴포넌트가 많을수록 중첩은 더 깊어짐
- Counter1과 Counter2는 각각 count1과 count2가 변경될 때만 리렌더링이 됨

### useReducer로 하나의 상태를 만들고 여러 개의 컨텍스트로 전파하기

- 단일 상태를 만들고 여러 컨텍스트를 사용해 상태 조각을 배포하는 방법
- 갱신하는 함수를 배포하는 것은 별도의 컨텍스트로 해야함

```tsx
type Action = { type: 'INC1' } | { type: 'INC2' };

const Count1Context = createContext < number > 0;
const Count2Context = createContext < number > 0;
const DispatchContext = createContext<Dispatch<Action>>(() => {});

const Counter1 = () => {
  const count1 = useContext(Count1Context);
  const dispatch = useContext(DispatchContext);

  return (
    <div>
      {count1}
      <button onClick={() => dispatch({ type: 'INC1' })}>+1</button>
    </div>
  );
};

// ...Counter2도 동일하게 구현 (dispatch({ type: 'INC2' })) 사용

const Parent = () => {
  return (
    <div>
      <Counter1 />
      <Counter2 />
    </div>
  );
};

const Provider = ({ children }: { children: React.ReactNode }) => {
  const [state, dispatch] = useReducer(
    (prev: { count1: number; count2: number }, action: Action) => {
      if (action.type === 'INC1') {
        return { ...prev, count1: prev.count1 + 1 };
      }
      if (action.type === 'INC2') {
        return { ...prev, count2: prev.count2 + 1 };
      }
      throw new Error('no matching action');
    },
    {
      count1: 0,
      count2: 0,
    }
  );

  return (
    <DispatchContext.Provider value={dispatch}>
      <Count1Context.Provider value={state.count1}>
        <Count2Context.Provider value={state.count2}>
          {children}
        </Count2Context.Provider>
      </Count1Context.Provider>
    </DispatchContext.Provider>
  );
};

const App = () => {
  return (
    <Provider>
      <Parent />
    </Provider>
  );
};
```

## 컨텍스트 사용을 위한 모범 사례

### 사용자 정의 훅과 공급자 컴포넌트 만들기

```tsx
type CountContextType = [number, Dispatch<SetStateAction<number>>];

const Count1Context = createContext<CountContextType | null>(null);

export const Count1Provider = ({ children }: { children: ReactNode }) => (
  <Count1Context.Provider value={useState(0)}>
    {children}
  </Count1Context.Provider>
);

export const useCount1 = () => {
  const value = useContext(Count1Context);
  if (value === null) throw new Error('Provider missing');
  return value;
};
```

### 사용자 정의 훅이 있는 팩토리 패턴

```tsx
const createStateContext = <Value, State>(
  useValue: (init?: Value) => State
) => {
  const StateContext = createContext<State | null>(null);
  const StateProvider = ({
    initialValue,
    children,
  }: {
    initialValue?: Value;
    children?: ReactNode;
  }) => (
    <StateContext.Provider value={useValue(initialValue)}>
      {children}
    </StateContext.Provider>
  );
  const useContextState = () => {
    const value = useContext(StateContext);
    if (value === null) throw new Error('Provider missing');
    return value;
  };
  return [StateProvider, useContextState] as const;
};
```

### reduceRight를 이용한 공급자 중첩 방지

```tsx
const App = () => {
  const providers = [
    [Count1Provider, { initialValue: 10 }],
    [Count2Provider, { initialValue: 20 }],
    [Count3Provider, { initialValue: 30 }],
    [Count4Provider, { initialValue: 40 }],
    [Count5Provider, { initialValue: 50 }],
  ] as const;
  return providers.reduceRight(
    (children, [Component, props]) => createElement(Component, props, children),
    <Parent />
  );
};
```
