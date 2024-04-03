---
title: "리액트에서 사용해보는 Code Splitting "
date: 2024-02-22 16:14:10 +0900
categories: [Frontend, React]
tags: [JavaScript, React]
---

## 1. Code Splitting 의 필요성

---

코드스플리팅은 React 뿐만이 아니라 Webpack 을 이용한 기타 다른 Application 에서도 모두 사용 가능한 언어이다.

JavaScript 로 개발을 하고 배포하는 과정에서 Build 과정을 거치게 되는데, 이 과정에서 모든 파일들이 하나로 합쳐지게 된다. 쉽게 말해 index.js 부터 containers, components 들로 나누어 놨던 소스 코드들이 하나의 거대한 소스로 합쳐진다. 작은 규모의 프로젝트라면 영향이 적겠지만 거대한 프로젝트(ex. Single Application Page) 라면, 굉장히 긴 자바스크립트 코드가 탄생하게 된다.

인터넷 환경이 좋지 못한 곳에서는 이렇게 방대한 소스를 불러 오는데 상당한 로딩 시간을 갖게 된다. 이때 코드 스플리팅(Code Splitting) 을 사용하게 되면 당장 사용하는 페이지의 코드 만을 로딩하고, 현재 필요하지 않은 페이지의 코드 부분은 분리 시켜 놓음으로써 Application 의 성능을 향상 시킬 수 있게 된다.

 <br>
 
 ## 2. Code Splitting 활용법
 ---
 ### 2-1. 코드 비동기 로딩
 이 방식은 필요한 부분에서 파일을 import 함으로써 필요한 순간에 코드를 불러온다. 쉽게 말해 import 를 함수형으로 사용하는 문법인데 dynamic import 라고 부른다.
 
```jsx
import logo from './logo.svg';
import './App.css';
import notify from './notify';

function App() {
const onClick = () => {
notify();
};

    return (
        <div className="App">
            <header className="App-header">
                <img src={logo} className="App-logo" alt="logo"/>
                <p onClick={onClick}>code-splitting</p>
            </header>
        </div>
    );

}

export default App;

````

- onClick 함수를 누르기도 전에 notify 함수 코드가 로드 되기 때문에, 아직 사용 하지도 않은 notify 함수 코드를 불러오는 상황이다.


```jsx
import logo from './logo.svg';
import './App.css';

function App() {
   const onClick = () => {
       import('./notify').then(result => result.default());
   };

   return (
       <div className="App">
           <header className="App-header">
               <img src={logo} className="App-logo" alt="logo"/>
               <p onClick={onClick}>code-splitting</p>
           </header>
       </div>
   );
}

export default App;
````

- 이처럼 Promise 를 반환하게끔 코드를 수정한다면 실제 onClick 함수가 동작할 때 notify 파일을 불러오게 된다.
- 실제로 빌드시에 static 폴더에 확인해보면 `notify` 파일에 대한 코드가 따로 생성 됐음을 확인 할 수 있다.
  ![Currying Image](/assets/img/post_img/coding/react/code_splitting.JPG){: width="500" align="center"}

<br>

### 2-2 React.Lazy & Suspense 사용

```jsx
React.lazy(() => component);
```

- React.Lazy 와 Suspense 는 React v16.6 부터 추가된 기능으로 React.Lazy 의 경우 Component 를 rendering 할 때 비동기적으로 로딩 하게 해주는 함수 이다.

<br >

```jsx
import React, { Suspense } from "react";

const Component = () => {
  return <Suspense fallback={Component} />;
};
```

- Suspense 는 코드 스플리팅 되어 로딩 되지 않은 Component 를 로딩 할 수 있게 만들어주는 Component 이다.
- 또한 옵션으로 로딩이 끝나지 않았을 때 보여줄 UI 를 따로 구성할 수 있다. (fallback 에는 로딩 하고자 하는 컴포넌트를 삽입한다)

<br >

```jsx
import logo from "./logo.svg";
import "./App.css";
import React, { useState, Suspense } from "react";

const SplitMe = React.lazy(() => import("./SplitMe"));

const App = () => {
  const [visible, setVisible] = useState(false);
  const onClick = () => {
    setVisible(true);
  };

  return (
    <div className={"App"}>
      <header className={"App-header"}>
        <img src={logo} className={"App-logo"} alt={"logo"} />
        <p onClick={onClick}>code splitting</p>
        <Suspense fallback={<div>로딩중...</div>}>
          {visible && <SplitMe />}
        </Suspense>
      </header>
    </div>
  );
};

export default App;
```

- onClick 함수가 동작할 때 “로딩중…” 이라는 문구가 보이고 나서 SplitMe 컴포넌트가 로드 되는 걸 확인할 수 있다.

### 2-3. Lodable Components 라이브러리

```jsx
import logo from "./logo.svg";
import "./App.css";
import React, { useState, Suspense } from "react";
import loadable from "@loadable/component";

const SplitMe = loadable(() => import("./SplitMe"), {
  fallback: <div>로딩중...</div>
});

const App = () => {
  const [visible, setVisible] = useState(false);
  const onClick = () => {
    setVisible(true);
  };

  const onMouseOver = () => {
    SplitMe.preload();
  };

  return (
    <div className={"App"}>
      <header className={"App-header"}>
        <img src={logo} className={"App-logo"} alt={"logo"} />
        <p onClick={onClick} onMouseOver={onMouseOver}>
          code splitting
        </p>
        {visible && <SplitMe />}
      </header>
    </div>
  );
};

export default App;
```

- 이 라이브러리는 코드 스플리팅을 편하게 할 수 있도록 도와주는 동시에 Server Side Rendering 이 가능하게 해준다.
- 사용 방법은 React.lazy - Suspense 와 유사하다. loadable 함수에 fallback 옵션을 통해 컴포넌트를 할당하면 코드를 불러오는 도중에 보여줄 컴포넌트를 할당할 수 있다.
- loadable 함수는 mouseover 기능에 대해 preload 기능을 제공해 마우스 커서가 올라가는 순간부터 코드를 불러올 수 있게 할 수 있다.
