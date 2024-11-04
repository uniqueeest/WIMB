# 구독을 이용한 모듈 상태 공유

## 모듈 상태

- 모듈 상태의 엄격한 정의는 ECMAScript(ES) 모듈 스코프에 정의된 상수 또는 변수
- 단순하게 모듈 상태는 전역적이거나 파일의 스코프 내에서 정의된 변수라고 가정할 수 있음

```tsx
export const createContainer = (initialState) => {
  let state = initialStat;
  const getState = () => stat;
  const setState = (nextState) => {
    state = typeof nextState === 'function' ? nextState(state) : nextStat;
  };

  return { getState, setState };
};
```

```tsx
import { createContainer } from '...';

const { getState, setState } = createContainer({
  count: 0,
});
```

## 리액트에서 전역 상태를 다루기 위한 모듈 상태 사용법

- 전체 트리에서 전역 상태가 필요하다면 모듈 상태가 더 적합할 수 있음
- 리액트 컴포넌트에서 모듈 상태를 사용하려면 리렌더링 최적화를 직접 처리해야 함
- 리액트에서 리렌더링을 일으키는 훅은 useState와 useReducer 두 개만 있음

## 기초적인 구독 추가하기

- 구독은 갱신에 대한 알림을 받기 위한 방법임

```tsx
const unsubscribe = store.subscribe(() => {
  console.log('store is updated');
});
```

```tsx
// 구독으로 모듈 상태 구현
type Store<T> = {
  getState: () => T;
  setState: (action: T | ((prev: T) => T)) => void;
  subscribe: (callback: () => void) => () => void;
};

const createStore = <T extends unknown>(initialState: T): Store<T> => {
  let state = initialState;

  const callbacks = new Set<() => void>();
  const getState = () => state;
  const setState = (nextState: T | ((prev: T) => T)) => {
    state =
      typeof nextState === 'function'
        ? (nextState as (prev: T) => T)(state)
        : nextState;
    callbacks.forEach((callback) => callback());
  };

  const subscribe = (callback: () => void) => {
    callbacks.add(callback);
    return () => {
      callbacks.delete(callback);
    };
  };

  return { getState, setState, subscribe };
};
```

```tsx
const useStore = (store) => {
  const [state, setState] = useState(store.getState());

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(store.getState());
    });
    setState(store.getState());
    return unsubscribe;
  }, [store]);

  return [state, store.setState];
};
```

- 모듈 상태는 리액트에서 갱신되기 때문에 리액트의 상태와 동일하게 모듈 상태를 불변적으로 갱신하는 것이 중요함

## 선택자와 useSubscription 사용하기

- 훅이 상태 객체 전체를 반환하면 일부분만 변경되더라도 훅에 전달되기 때문에 불필요한 리렌더링을 발생시킬 수 있음
- 필요 없는 리렌더링을 피하기 위해 컴포넌트가 필요로 하는 상태의 일부분만 반환하는 선택자(selector)를 도입할 수 있음

```tsx
const store = createStore({ count1: 0, count2: 0 });

const useStoreSelector = <T, S>(store: Store<T>, selector: (state: T) => S) => {
  const [state, setState] = useState(() => selector(store.getState()));

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(selector(store.getState()));
      setState(selector(store.getState()));
    });
    setState(selector(store.getState()));
    return unsubscribe;
  }, [store, selector]);

  return state;
};
```

- useStore와 다르게 useStoreSelector의 useState 훅은 상태의 전체 내용 대신 selector의 반환 값을 가짐

```tsx
const Component1 = () => {
  const state = useStoreSelector(
  store,
  useCallback((state) => state.count1),
  []
);
...
}
```

- 안정적으로 선택자 함수를 넘기기 위해서는 useCallback을 사용해야 함. 그렇지 않으면 useEffect의 두 번째 인수에 선택자 함수가 지정돼 있으므로 선택자 함수를 바꾸지 않더라도 Component1을 렌더링할 때마다 store를 구독 해제하고 구독하는 것을 반복하게 됨
- 혹은 useCallback을 사용하지 않고 컴포넌트 외부에 선택자 함수를 정의할 수 있음

```tsx
const selectCount2 = (state: ReturnType<typeof store.getState>) => state.count2;

const Component1 = () => {
  const state = useStoreSelector(store, selectCount2);

  return (
    <div>
      count2: {state}{' '}
      <button
        onClick={() => {
          store.setState((prev) => ({
            ...prev,
            count2: prev.count2 + 1,
          }));
        }}
      >
        +1
      </button>
    </div>
  );
};
```

- useStoreSelector 훅은 store 또는 selector가 변경될 때 주의할 점이 있는데, useEffect는 조금 늦게 실행되기 때문에 재구독될 때까지는 갱신되기 이전 상태 값을 반환함
- react에서 제공하는 useSyncExternalStore 훅을 사용하여 이를 해결할 수 있음

```tsx
const useStoreSelector = (store, selector) => {
  useSyncExternalStore(
    store.subscribe,
    () => selector(store.getState()),
    () => selector(store.getState())
  );
};
```
