# 사용 사례 시나리오 2:Jotai

- useState, useReducer와 함께 상태의 작은 조각인 atom이라는 것을 모델로 삼음

## Jotai의 이점

### 구문 단순성

- Jotai는 context를 기반으로 만들어졌음. 하지만 전역 상태를 추가할 때마다 Provider를 추가해야하는 context와 달리, Jotai는 추가를 할 필요가 없음

```tsx
const countAtom = atom(0);

const Counter1 = () => {
  const [count, setCount] = useAtom(countAtom);

  return (
    <>
      {count}
      <button
        onClick={() => {
          setCount((prev) => prev + 1);
        }}
      >
        +1
      </button>
    </>
  );
};

const Counter2 = () => {
  const [count, setCount] = useAtom(countAtom);

  return (
    <>
      {count}
      <button
        onClick={() => {
          setCount((prev) => prev + 1);
        }}
      >
        +1
      </button>
    </>
  );
};

const App = () => {
  return (
    <>
      <Counter1 />
      <Counter2 />
    </>
  );
};
```

### 동적 아톰 생성

- Jotai의 스토어는 아톰 구성 객체와 아톰 값으로 구성된 WeakMap 객체
- 아톰 구성 객체(atom config object)는 atom 함수로 생성됨
- 아톰 값은 useAtom 훅이 반환하는 값임
- Jotai의 구독은 아톰 기반이므로 useAtom 훅이 store에 있는 특정 아톰을 구독한다는 것을 의미 -> 불필요 리렌더링을 피할 수 있음

## 렌더링 최적화

- Jotai는 상향식(bottom-up) 접근법을 활용해 렌더링 최적화를 함
- 작은 아톰을 만들고, 이를 결합하여 큰 아톰을 만듦

```tsx
const firstNameAtom = atom('React');
const lastNameAtom = atom('Hooks');
const ageAtom = atom(3);

const fullNameAtom = atom((get) => ({
  firstName: get(firstNameAtom),
  lastName: get(lastNameAtom),
}));

const personAtom = atom((get) => ({
  firstName: get(firstNameAtom),
  lastName: get(lastNameAtom),
  age: get(ageAtom),
}));
```

- read 함수는 다른 아톰을 참조하고 그 값을 가져올 수 있는 get이라는 인수를 받음. personAtom의 값은 속성 중 하나가 변경될 때마다 갱신됨. 이것을 의존성 추적(dependency tracking)이라고 함
- \* 의존성 추적은 동적이며 조건부 평가에서도 작동함. 예를 들어, read 함수가 (get) => get(a) ? get(b) : get(c) 라면, a가 참이면 의존성은 a,b 이고, 거짓이면 의존성은 b,c 임

## Jotai가 아톰 값을 저장하는 방식

```tsx
const countAtom = atom(0);
```

- countAtom은 값 또는 갱신 함수로 변경할 수 있는 값을 가진 원시 아톰임
- 아톰 값을 저장하는 store는 따로 있음
- store에는 키가 아톰 구성 객체이고 값이 아톰 값인 WeakMap 객체가 있음

## 배열 구조 추가하기

- Atoms-in-Atom
  - 배열 아톰은 아톰이 요소인 배열을 보관하는 데 사용됨
  - 배열에 새로운 요소를 추가하려면 새로운 아톰을 생성해서 추가해야 함
  - 아톰 구성은 문자열로 평가할 수 있으며 UID를 반환함
  - 요소를 렌더링하는 컴포넌트는 각 컴포넌트에서 아톰 요소를 사용함. 이렇게 하면 요소의 값을 쉽게 변경할 수 있고 리렌더링을 자연스럽게 피할 수 있음

## Jotai의 다양한 기능

### 아톰의 write 함수 정의

```tsx
const doubledCountAtom = atom(
  (get) => get(countAtom) * 2,
  (get, set, arg) => set(countAtom, arg / 2)
);
```

- get은 아톰의 값을 반환하는 함수
- set은 아톰의 값을 설정하는 함수
- arg는 아톰을 갱신할 때 받을 임의의 값

### 액션 아톰 사용

- 상태를 변경하는 코드를 위해 함수 또는 함수 집합을 만드는 경우가 많음
- 이러한 목적으로 아톰을 사용하는 것을 액션 아톰이라 함

```tsx
const countAtom = cout(0);
const incrementCountAtom = atom(null, (get, set, arg) =>
  set(countAtom, (c) => c + 1)
);

// 컴포넌트
const [, incrementCount] = useAtom(incrementCountAtom);
```

### 아톰의 onMount 옵션

- 아톰이 사용되기 시작할 때 특정 로직을 실행하고 싶을 수 있음 (ex. 외부 데이터 구독)

```tsx
const countAtom = atom(0);
countAtom.onMount = (setCount) => {
  console.log('count atom 사용 시작');
  const onUnmount = () => {
    console.log('count atom 사용 종료');
  };

  return onUnmount;
};
```
