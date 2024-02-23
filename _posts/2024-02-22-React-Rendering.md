---
title: "리액트 컴포넌트 렌더링 최적화 방법"
date: 2024-02-22 13:52:00 +0900
categories: [Frontend, React]
tags: [JavaScript, React]
---

# Intro

불필요한 rerendering은 component를 reflow & repaint 시킴으로써 브라우저 성능 저하가 발생할 수 있다.
함수형 컴포넌트의 경우 리렌더링을 할 경우 내부 로직들이 재호출 되기 때문에 불필요한 컴포넌트 리렌더링을 최소화 해야 한다.

---

## Rendering 조건

1. 본인의 state가 변경될 때
2. 부모 컴포넌트로부터 받아오는 props가 변경될 때
3. 부모 컴포넌트가 리렌더링 될때

```javascript
const SpinBox = () => {
  const [value, setValue] = useState(0);

  const onClick = () => {
    setValue(value + 1);
  };

  return (
    <div>
      <Display value={value} />
      <Button onClick={onClick} />
    </div>
  );
};

const Display = ({ value }) => {
  console.log("Display Render");
  return <span>{value}</span>;
};

const Button = ({ onClick }) => {
  console.log("Button Render");
  return <button onClick={onClick}>증가</button>;
};
```

- 예제 코드에서 Button을 클릭하게 되면 `Display Render`, `Button Render` 둘다 출력하게 된다.
- `<SpinBox>`의 value가 변경 되었으니 value를 props로 받는 `<Display>`가 렌더링이 발생한다.
- 하지만, `<Button>`의 경우 아무런 변화가 있지 않음에도 렌더링이 일어나는데 그 이유는 부모 컴포넌트인 `<SpinBox>`가 렌더링이 일어나 그 자식 컴포넌트인 `<Button>`도 렌더링이 발생하였다.

---

## Mmoization Function

React 는 불필요한 렌더링을 방지하기 위해 다양한 memoization 함수를 제공한다.

### 1. React.memo

- 단순한 부모 컴포넌트의 렌더링은 memoization을 통해 자식 컴포넌트의 불필요한 리렌더링을 방지할 수 있다.
- 하지만 props의 값이 빈번하게 바뀌게 되는 부모 - 자식 컴포넌트 관계에 `React.memo`를 사용하게 되면 불필요한 memoization 처리로 서비의 성능이 저하되는 악영향을 미칠 수 있다.

```javascript
const Button = ({ onClick }) => {
  console.log("Button Render");
  return <button onClick={onClick}>증가</button>;
};

export default React.memo(Button);
```

### 2. useCallback

- 하지만 `<Button>`으로 전달되는 onClick 함수 때문에 memo만으로 렌더링을 방지할 수 없다.
- component가 rerender 될 때, 함수를 재사용 하기 위해서는 useCallback hook을 사용 하면 된다.

```javascript
const SpinBox = () => {
  const [value, setValue] = useState(0);

  // useCallback을 이용해 value 값이 바뀌지 않을 때 함수 재사용
  const onClick = useCallback(() => {
    setValue(value + 1);
  }, [value]);

  return (
    <div>
      <Display value={value} />
      <Button onClick={onClick} />
    </div>
  );
};
```

### 3. usestate 의 함수형 업데이트 방식

- 하지만 `useCallback`을 사용한다 하더라도 위의 코드는 의미가 없다.
- 왜냐하면 onClick이 동작하면 어차피 value의 변경이 감지되어 onClick 함수가 다시 그려지기 때문이다.
- 이러한 구조에서 최적화 문제를 해결하기 위해서는 state에 직접 접근하지 않도록 함수 작성을 해야 한다.
- 예제 코드와 같이 state를 가공하는 함수를 전달하는 방식으로 구현하면 state에 직접 접근하지 않고도 기존 state를 기반으로 변경을 할 수 있다.

```javascript
const SpinBox = () => {
  const [value, setValue] = useState(0);

  // useCallback을 이용해 value 값이 바뀌지 않을 때 함수 재사용
  const onClick = useCallback(() => {
    setValue((prevState) => prevState + 1);
  }, [value]);

  return (
    <div>
      <Display value={value} />
      <Button onClick={onClick} />
    </div>
  );
};
```

- `useReducer` 를 사용한다면 렌더링 최적화도 할 수 있을 뿐더러 `useState` (함수형 업데이트) 방식보다 좀더 확장성 있는 방식으로 개발 할 수 있을것이다.
