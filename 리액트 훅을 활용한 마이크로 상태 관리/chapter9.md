# 사용 사례 시나리오 3:Valtio

- Valtio는 주로 모듈 상태에 사용되는 라이브러리라는 점에서 Zustand와 동일
- 프락시는 자바스크립트의 특수한 객체로서 객체 연산을 감지하기 위한 핸들러를 만드는 데 활용할 수 있음

```tsx
const proxyObject = new Proxy(
  {
    count: 0,
    text: 'hello',
  },
  {
    set: (target, prop, value) => {
      console.log('start setting', prop);
      target[prop] = value;
      console.log('end setting', prop);
    },
  }
);

++propxyObject.count;
// start setting count
// end setting count
// 1
```

## 프락시를 활용한 변경 감지 및 불변 상태 생성하기

- Valtio는 프락시를 사용해 변경 가능한 객체에서 변경 불가능한 객체를 생성함. 이걸 스냅숏(snapshot)이라 함

```tsx
import { proxy, snapshot } from 'valtio';
const state = proxy({ count: 0 });
const snap1 = snapshot(state);
```

- state가 {count:0}이고 snap1도 {count:0}이지만 state와 snap1은 서로 다른 참조를 가짐
- state는 프락시로 감싼 변경 가능한 객체인 반면, snap1은 Object.freeze로 동결되어 변경 불가능한 객체임

```tsx
++stae.count;

const snap2 = snapshot(state);
```

- state는 {count:1}이며 이전과 동일한 참조를 가짐. snap2는 {count:1}이며 새로운 참조를 가짐

```tsx
const state = proxy({ obj1: { c: 0 }, obj2: { c: 0 } });

const snap21 = snapshot(state);

++state2.obj1.c;

const snap22 = snapshot(state);

// snap21 !== snap22 => 서로 다른 참조를 가지기 때문
// snap21.obj2 === snap22.obj2 => 내부 c 속성이 변경되지 않았기 때문
```

- snap21.obj2와 snap22.obj2의 참조가 동일하다는 것은 메모리를 공유한다는 의미
- 필요한 경우에만 스냅숏을 생성해서 메모리 사용량을 최적화
- \* Valtio의 최적화는 이전 스냅숏에 대한 캐싱을 기반으로 함. 즉, 캐시 공간이 하나라는 의미. 따라서 ++state.count로 카운트를 늘린 다음, --state.count로 줄이면 새로운 스냅숏이 생성됨

## 프락시를 활용한 리렌더링 최적화

```tsx
import { proxy, useSnapshot } from 'valtio';

const state = proxy({
  count1: 0,
  count2: 0,
});

const Counter1 = () => {
  const snap = useSnapshot(state);
  const inc = () => ++state.count1; // state를 변경하는 함수
  return (
    <>
      {snap.count1}
      <button onClick={inc}>+1</button>
    </>
  );
};
```

- useSnapshot에서 반환하는 값의 변수명을 snap으로 설정하는 것이 관례임
- snap 객체는 Object.freeze로 동결되며 기술적으로 변경할 수 없음
- 접근은 useSnapshot 훅에 의해 추적 정보로 감지되며, 추적 정보를 기반으로 useSnapshot 훅은 필요한 경우에만 리렌더링 감지

```tsx
const contrivedState = proxy({
  num: 123,
  str: 'hello',
  arr: [1, 2, 3],
  nestedObject: { foo: 'bar' },
  objectArray: [{ a: 1 }, { b: 2 }],
});
```

- Valtio는 중첩된 객체와 배열을 지원함

## Valtio 접근 방식의 장단점

### 멘탈 모델

- Valtio에는 두 가지 상태 업데이트 모델이 있음. 하나는 불변 갱신이고 하나는 변경 가능한 갱신임
- 자바스크립트 자체는 변경 가능한 갱신을 허용하지만 리액트는 불변 상태를 중심으로 만들어짐
- 따라서 두 모델을 같이 사용하는 경우 혼동하지 않도록 주의해야함

```ts
// 1. 요소를 제거할 떄
// 변경 가능
array.splice(index, 1);

// 불변
[...array.slice(0, index), ...array.slice(index + 1)];

// 2. 중첩된 객체에서 값을 변경하는 경우
// 변경 가능
state.a.b.c.text = 'hello';

// 불변
{
  ...state,
  a: {
    ...state.a,
    b: {
      ...state.a.b,
      c: {
        ...state.a.b.c,
        text: "hello",
      }
    }
  }
}
```

### 장점

- 변경 가능한 갱신으로 애플리케이션 코드를 줄이는데 도움이 됨
- 프락시 기반 렌더링 최적화를 사용해 애플리케이션 코드를 줄이는데 도움이 됨

### 단점

- 프락시 기반 렌더링 최적화의 단점은 예측 가능성이 떨어짐
- 렌더링 최적화를 내부적으로 처리하기 때문에 동작을 디버깅하기 어려울 수 있음
