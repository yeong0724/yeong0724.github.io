---
title: "Immer 를 사용해 쉽게 불변성 유지하기"
date: 2024-04-12 20:16:40 +0900
categories: [Frontend, React]
tags: [JavaScript, React, npm]
---

`Immer` 는 리액트에서 상태값을 업데이트 할 때, "**불변성**"을 신경쓰지 않고 업데이트 할 수 있는 코드를 작성하기 쉽게 해주는 라이브러리이다.

## 그렇다면 불변성이란 도대체 무엇인가?

---

> 사전적으로 불변성이란 값이나 상태를 변경할 수 없는 것을 의미한다.

자바스크립트의 원시타입을 통해 불변성을 살펴보자.원시타입은 불변성을 가지고 있다.
대표적인 원시타입인 String 타입을 예로 들어보겠다.

```jsx
let string = "data1";

string = "data2";
```

`변수 string 이 data1 → data2 로 바꼈다` 라는 문장은 엄밀한 보면 반은 맞고 반은 틀렸다. 실제 메모리 영역에서는 ‘data1’, ‘data2’ 둘 다 존재하기 때문이다.
메모리 영역이 1 ~ 10 까지 있다고 가정하자면 ‘data1’, ‘data2’ 은 각 메모리 영역 1, 메모리 영역 2에 등록 됐다고 볼 수 있다.
즉, ‘data2’는 ‘data1’을 대체하는 것이 아니라 새로운 영역에 할당이 된 것이다.

이번에는 참조타입인 Array 타입을 통해 불변성을 살펴 보겠다.

```jsx
let array = [1, 2, 3, 4]; // 메모리영역 1
array.push(5); // 메모리영역 1

array = [1, 2, 3, 4, 5]; // 메모리영역 2 (새로운 참조값)
```

Array 의 push 함수는 기존 메모리 영역 1에 있는 원본 배열을 수정하여 불변성을 지키지 않고 있고, array = [1, 2, 3, 4, 5] 는 새 참조값을 가진 새로운 배열을 할당하여 불변성을 지켜주고 있다.

간단히 정리하자면, 불변성의 진정한 의미는 “**`메모리 영역의 값이 변하 지 않는다`**” 라는 의미이다.

<br />

## React 에서 왜 불변성을 지켜야 하는가?

---

유난히 리액트에서 불변성을 지켜줘야 하는 이유는 리액트가 상태 업데이트를 하는 원리 때문이다. 리액트는 상태값을 업데이트 할 때 얕은 비교를 수행한다.
즉, 배열이나 객체의 속성 하나 하나를 비교하는 게 아니라 이전 참조값과 현재 참조값만을 비교하여 상태 변화를 감지한다. 그래서 불변성을 지키지 않고 원본 값을 수정하게 되는 경우 상태 변화를 감지하지 못하고 랜더링이 일어나지 않게 된다.

불변성을 지킴으로써 얻게 되는 또 다른 이점은 바로 **Side-Effect** 를 방지 하는 것이다. 즉 외부에 존재하는 원본 데이터를 직접 수정 하지 않고, 원본 데이터의 복사본을 만들어서 값을 사용하기에 예상치 못한 오류를 사전에 방지 할 수 있다. 반대로 생각해보자면 외부의 값을 함부로 변경 할 수 있다면 위험한 일이다.

결국 리액트는 불변성을 지킴으로써 효과적인 상태 업데이트와 사이드 이펙트를 방지하는 이점을 얻고 있다.

<br />

## Immer 사용법

---

```jsx
import produce from "immer";

const state = {
  number: 1,
  dontChangeMe: 2
};

const nextState = produce(state, (draft) => {
  draft.number += 1;
});

console.log(nextState);
```

- produce 함수를 사용 할 때에는 첫 번째 파라미터에는 수정하고 싶은 상태, 두 번째 파라미터에는 어떻게 업데이트하고 싶을지 정의하는 함수를 넣어준다.

- 두 번째 파라미터에 넣는 함수에서는 불변성에 신경 쓰지 않고 그냥 상태 업데이트 처리를 해주면 Immer가 알아서 해준다.

### 1. Reducer 에서 Immer 사용하기

```jsx
import React, { useReducer, useMemo } from "react";
import UserList from "./UserList";
import CreateUser from "./CreateUser";
import produce from "immer";

function countActiveUsers(users) {
  console.log("활성 사용자 수를 세는중...");
  return users.filter((user) => user.active).length;
}

const initialState = {
  users: [
    {
      id: 1,
      username: "velopert",
      email: "public.velopert@gmail.com",
      active: true
    },
    {
      id: 2,
      username: "tester",
      email: "tester@example.com",
      active: false
    },
    {
      id: 3,
      username: "liz",
      email: "liz@example.com",
      active: false
    }
  ]
};

function reducer(state, action) {
  switch (action.type) {
    case "CREATE_USER":
      return produce(state, (draft) => {
        draft.users.push(action.user);
      });
    case "TOGGLE_USER":
      return produce(state, (draft) => {
        const user = draft.users.find((user) => user.id === action.id);
        user.active = !user.active;
      });
    case "REMOVE_USER":
      return produce(state, (draft) => {
        const index = draft.users.findIndex((user) => user.id === action.id);
        draft.users.splice(index, 1);
      });
    default:
      return state;
  }
}

// UserDispatch 라는 이름으로 내보내줍니다.
export const UserDispatch = React.createContext(null);

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);

  const { users } = state;

  const count = useMemo(() => countActiveUsers(users), [users]);
  return (
    <UserDispatch.Provider value={dispatch}>
      <CreateUser />
      <UserList users={users} />
      <div>활성사용자 수 : {count}</div>
    </UserDispatch.Provider>
  );
}

export default App;
```
