---
title: "[JavaScript] Currying 이란?"
date: 2024-02-15 23:15:40 +0900
author:
  name: "Jinyeong Kim"
categories: [Coding, JavaScript]
tags: [coding, JavaScript]
---

![Currying Image](/assets/img/post_img/coding/javascript/currying.png){: width="500" class="normal"}

프로그래밍 언어가 가지고 있는 프로그래밍 패러다임 중, 각 언어만이 가지고 있는 고유한 프로그래밍 패러다임이 있다.

JavaScript 의 경우 클래스를 이용한 객체지향 뿐만 아니라 Prototype 기반의 객체지향, 나아가서는 최근에 인기를 끌고 있는 함수형 프로그래밍까지 가능한 멀티패러다임을 가진다.

**커링(Currying)** 은 이러한 JavaScript의 특징을 나타내는 방법중 하나이다.
<br />

# JavaScript 의 커링

수학과 컴퓨터 과학에서 **Currying** 이란 다중인수 or 여러 인수의 튜플 을 갖는 함수를 단일 인수를 갖는 함수들의 함수열로 바꾸는 것을 의미한다.

이러한 커링은 JavaScript 에만 존재하는 것은 아니고 다른 프로그래밍 언어에도 사실 존재하는데, 커링을 사용하는 이유는 함수형 프로그래밍 언어를 사용하는 이유와 일치한다.

바로, 부수효과를 줄이고 동일한 입력이 들어가면 동일한 출력이 나오게 하여 (가독성 / 유지보수 용이성)을 얻기 위함이다.

```javascript
// 커링 변환을 하는 curry(f) 함수 (일반함수 ver)
function curry(f) {
  return function (a) {
    return function (b) {
      return f(a, b);
    };
  };
}

// 커링 변환을 하는 curry(f) 함수 (화살표함수 ver)
const curry = (func) => (first) => (second) => f(first, second);

// f에 전달된 함수
const sum = (a, b) => a + b;

const curriedSum = curry(sum);

console.log(curriedSum(1)(2));

// 한번에 커링함수를 호출하는 형태
curry(sum, 1, 2);
```

- curry(func)의 반환 값은 function(a) 형태
- curriedSum(1) 과 같은 함수가 호출되었을 때, 1은 렉시컬 환경에 저장이 되고 function(b)가 반환
- 반환된 function(b) 함수가 2를 인수로 호출되고, 반환 값이 원래의 sum으로 넘겨져서 호출
- 최종적으로 sum(1, 2) 가 호출되어 1 + 2인 3이 반환

<br />

# lodash를 통한 커링 구현

```javascript
const todos = [
  { id: 3, content: "HTML", completed: false },
  { id: 2, content: "CSS", completed: true },
  { id: 1, content: "Javascript", completed: false }
];

const todoItems = (items, param) => items.map((item) => item[param]);

const getTodoItems = _.curry(todoItems);

console.log(getTodoItems(todos)("id")); // [3, 2, 1]
```

- 커링을 사용하는 경우 인자의 순서가 중요하다. 일반적으로는 앞에 존재하는 인자일 수록 변동가능성이 적고, 뒤에 있는 인자일수록 변동가능성이 높아 인자배치를 통해 코드의 기능을 유추할 수 있다.

<br />

# 커링의 활용

```javascript
const log = _.curry((date, importance, message) => {
  console.log(
    `[${date.getHours()}:${date.getMinutes()}] [${importance}] ${message}`
  );
});

// 1. 단계별로 커링함수를 분리
const setLogLevel = log(new Date());
const setLogMsg = setLogLevel("INFO");
setLogMsg("흐림 / 10°C"); // "[8:44] [INFO] 흐림 / 10°C"

// 2. 커링함수를 한번에 호출
log(new Date())("DEBUG")("맑음"); // "[8:44] ["DEBUG"] 맑음
```

- 위의 코드처럼 단계별로 커링함수를 나누게 된다면, 단계별로 고정하고자 하는 인자는 고정하여 원하는 값을 도출할 수 있게 되어 debugging 도 용이해지는 효과를 볼 수 있다.
