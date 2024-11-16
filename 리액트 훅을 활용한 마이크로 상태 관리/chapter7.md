# 사용 사례 시나리오 1:Zustand

## 모듈 상태와 불변 상태 이해하기

- Zustand는 상태를 유지하는 store를 만드는데 사용되는 라이브러리임
- 주로 모듈 상태를 위해 설계됐으므로 모듈에서 store를 정의하고 내보내는 것을 할 수 있음
- 상태 객체 속성을 갱신할 수 없는 불변 상태 모델을 기반으로 함
- 상태를 변경하기 위해서는 새 객체를 생성해서 대체해야 하며, 수정하지 않은 객체는 재사용 해야함
- 불변 상태 모델의 장점은 상태 객체의 참조에 대한 동등성만 확인하면 변경 여부를 알 수 있으므로 객체의 값 전체를 확인할 필요가 없음

```tsx
const store = create(() => ({ count: 0 }));

store.setState((prev) => ({ count: prev.count + 1 }));

const store = create(() => ({
  count: 0,
  text: 'hello',
}));
console.log(store.getState()); // {count: 0, text: "hello"};

store.setState({
  count: 1,
});

console.log(store.getState()); // {count: 1, text: "hello"}
```

- count만 변경하기 때문에 text의 속성은 변경되지 않음. 이것은 내부적으로 Object.assign()으로 구현됨

```tsx
Object.assign({}, oldState, newState);

store.subscribe(() => {
  console.log('store state is changed');
});
```

- subscribe 함수를 사용하면 store의 상태가 변경될 때마다 호출되는 콜백 함수를 등록할 수 있음

## 리액트 훅을 이용한 리렌더링 최적화

- 리액트에서 store를 사용하려면 사용자 정의 훅이 필요함.
- Zustand의 create 함수는 훅으로 사용할 수 있는 store를 생성

```tsx
const useStore = create(() => ({
  count: 0,
  text: 'hello',
}));

const Component = () => {
  const { count, text } = useStore();

  return <div>count : {count}</div>;
};
```

- count 값을 보여주면서 store의 상태가 변경될 때마다 리렌더링 됨
- text 값만 변경되고 count 값이 변경되지 않으면 컴포넌트는 동일한 JSX 요소를 출력하기에 사용자는 변경사항을 볼 수 없지만, 리렌더링은 일어남
- 선택자 함수를 통해 리렌더링을 피할 수 있음

```tsx
const Component = () => {
  const count = useState((state) => state.count);

  return <div>count : {count}</div>;
};
```

- 선택자 기반의 리렌더링 제어를 수동 렌더링 최적화라고 하는데, 리렌더링을 피하기 위해 선택자는 선택자 함수가 반환하는 결과를 비교하는 방식으로 작동함
- 선택자 기반 리렌더링 최적화의 장점
  - 선택자 함수를 명시적으로 작성하기 때문에 동작을 정확히 예측할 수 있음
- 선택자 기반 리렌더링 최적화의 단점
  - 객체 참조에 대한 이해가 필요함

# 라이브러리의 장단점

- 읽기 상태: 리렌더링을 최적화하기 위해 선택자 함수를 사용
- 쓰기 상태: 불변 상태 모델을 기반으로 함
- 리액트는 불변성에 기반한 객체 참조 동등성으로 리렌더링을 최적화 함
- 객체 참조가 동일하다면 내부 값은 변경되어도 객체 자체는 변경되지 않기 때문에 리액트는 변경되지 않았다고 추측

```tsx
const obj = { a: 1, b: 2 };
const Component = () => {
  const [state, setState] = useState(obj);

  const handleClick = () => {
    obj.a += 1;
    setState(obj);
  };

  return (
    <div>
      <button onClick={handleClick}>클릭</button>
    </div>
  );
};
```

- Zustand의 한계는 선택자를 이용한 수동 렌더링 최적화임
- 객체 참조 동등성을 이해해야 하며, 선택자 코드를 위해 보일러플레이트 코드를 많이 작성해야 함
