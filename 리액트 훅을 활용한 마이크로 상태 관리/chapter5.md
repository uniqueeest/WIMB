# 리액트 컨텍스트와 구독을 이용한 컴포넌트 상태 공유

## 모듈 상태의 한계

- 모듈 상태는 리액트 컴포넌트 외부에 존재하는 전역으로 정의된 싱글턴이기 때문에 컴포넌트 트리나 하위 트리마다 다른 상태를 가질 수 없다는 한계가 있음

## 컨텍스트와 구독 패턴 사용하기

```tsx
type state = { count: number; text?: string };

const StoreContext = createContext<Store<State>>(
  createStore<State>({ count: 0, text: 'hello' })
);
```

```tsx
const StoreProvider = ({
  initialState,
  children,
}: {
  initialState: State;
  children: ReactNode;
}) => {
  // useRef는 스토어 객체가 첫 번째 렌더링에서 한 번만 초기화되게 만드는데 사용
  const storeRef = useRef<Store<State>>();
  if (!storeRef.current) {
    storeRef.current = createStore(initialState);
  }

  return (
    <StoreContext.Provider value={storeRef.current}>
      {children}
    </StoreContext.Provider>
  );
};
```

- useSelector는 인수로 스토어 객체를 받지 않고, StoreContext에서 store 객체를 가져옴

```tsx
const useSelector = <S extends unknown>(selector: (state: State) => S) => {
  const store = useContext(StoreContext);

  return useSubscription(
    useMemo(
      () => ({
        getCurrentvalue: () => selector(store.getState()),
        subscribe: store.subscribe,
      }),
      [store, selector]
    )
  );
};
```

- 모듈 상태와 다르게 컨텍스트를 사용해서 상태를 갱신하는 방법을 제공해야 함

```tsx
const useSetState = () => {
  const store = useContext(StoreContext);
  return store.setState;
};
```

```tsx
const selectCount = (state: State) => state.count;

const Component = () => {
  const count = useSelector(selectCount);
  const setState = useSetState();

  const inc = () => {
    setState((prev) => ({
      ...prev,
      count: prev.count + 1,
    }));

    return (
      <div>
        count: {count} <button onClick={inc}>+1</button>
      </div>
    );
  };
};
```

- Component 컴포넌트는 특정 스토어 객체에 연결돼 있지 않기 때문에, 다른 스토어에서 사용할 수 있음

```tsx
const App = () => {
  // 공급자는 중첩될 수 있고, 가장 안쪽에 위치한 공급자의 값을 사용함
  return (
    <>
      <Component />
      <StoreProvider initialState={{ count: 10 }}>
        <Component />
        <Component />
        <StoreProvider initialState={{ count: 20 }}>
          <Component />
        </StoreProvider>
      </StoreProvider>
    </>
  );
};
```

- 동일한 store 객체를 사용하는 Component 컴포넌트는 동일한 count 값을 보여줌. 다른 컴포넌트 트리의 컴포넌트는 다른 store를 사용하므로 다른 count 값을 보여줌
- 이러한 방식을 통해 하위 트리에 전역 상태를 제공할 수 있고, 컨텍스트 공급자를 중첩하는 것이 가능함
- 리액트 컴포넌트 생명 주기 내에서 useState와 같은 훅으로 전역 상태를 제어할 수 있음
- 구독을 사용하면 단일 컨텍스트로는 불가능한 리렌더링 문제를 해결할 수 있음
