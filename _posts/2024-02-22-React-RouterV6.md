---
title: "[React] Router 6 버전 달라진점"
date: 2024-02-22 12:22:30 +0900
categories: [JavaScript, React]
tags: [JavaScript, React]
---

> React v16.8

React Router v6는 React Hook 을 많이 사용하므로 v5 → v6로 업그레이드 하기 전에 React 16.8 이상 이어야 한다.

## 1. Routes 의 사용법 변경

### 1. Switch 컴포넌트 대신 Routes 컴포넌트 사용

- Switch 의 네이밍이 Routes 로 변경
- exact 옵션 삭제
  - exact 는 더이상 사용하지 않고 여러 라우팅을 매칭하고 싶은 경우 URL 뒤에 \* 을 사용
- component 방식 변경 ( component={ } & render{( ) ⇒ component} 삭제 )
  - component props 대신 elemet props 로 component를 전달
- path 를 상대경로로 지정

```javascript
import React from "react";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import { Main, Page1, Page2, NotFound } from "../pages";
import { Header } from ".";

const Router = () => {
  return (
    <BrowserRouter>
      <Header />
      <Routes>
        <Route path="/" element={<Main />} />
        <Route path="/page1/*" element={<Page1 />} />
        <Route path="/page2/*" element={<Page2 />} />
        <Route path="/*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
};

export default Router;
```

### 2. 중첩 라우팅

- Router,js 에서 자식 태그로 중첩하는 라우터를 기재하고, Web.js 에서 Outlet 라이브러리를 통해 가져온다.
- exact 를 쓰지 않는 대신 “ /\* ” 가 필수 이다.

```javascript
// Router.js
import React from "react";
import { BrowserRouter, Route, Routes } from "react-router-dom";
import Web from "../Pages/Web";
import WebPost from "../Pages/WebPost";

const Router = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="web/*" element={<Web />}>
          <Route path=":id" element={<WebPost />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
};

export default Router;

--------------------------------------------------------------------------------
// Web.js
import React from "react";
import { Link, Routes, Route, Outlet } from "react-router-dom";
import WebPost from "./WebPost";

const Web = () => {
  return (
    <div>
      <h1>This is Web</h1>
      <ul>
        <li>
          <Link to="1">Post #1</Link>
        </li>
        <li>
          <Link to="2">Post #2</Link>
        </li>
        <li>
          <Link to="3">Post #3</Link>
        </li>
        <li>
          <Link to="4">Post #4</Link>
        </li>
      </ul>

      <Outlet />
    </div>
  );
};

export default Web;

--------------------------------------------------------------------------------
// WebPost.js
import React from "react";

const WebPost = () => {
  return <div>This is 포스트</div>;
};

export default WebPost;
```

### 3. New Naviagtion API

> useHistory hook 과 Redirect Component는 이제 새로운 useNavigation hook과 Navigate Component로 대체된다.

#### 3-1. useNavigation hook

- v5에서 imperative하게 이동하기 위해서는 useHistory hook으로 가져온 history 객체의 메서드를 호출하였다.

```javascript
// react-router v5
import { useHistory } from "react-router-dom";

const App = () => {
  const history = useHistory();
  const handleClick = () => history.push("/home");

  return (
    <div>
      <button onClick={handleClick}>Go to Home</button>
    </div>
  );
};
```

- 하지만 v6에서는 useNavigate hook 으로 가져온 navigate 함수를 직접적으로 호출하게 된다.

```javascript
// react-router v6
import { useNavigate } from "react-router-dom";

const App = () => {
  const navigate = useNavigate();
  const handleClick = () => navigate("/home");

  return (
    <div>
      <button onClick={handleClick}>Go to Home</button>
    </div>
  );
};
```

- 기존의 history 객체의 go( ), goForward( ), goBack( ) 메서드의 경우 아래와 같이 navigate 함수에 정수 인자를 전달하는 방식으로 대체된다.

```javascript
navigate(N); // history.go(N)
navigate(1); // history.goForward()
navigate(-1); // history.goBack()
navigate(2); // go 2 pages forward
navigate(-2); // go 2 pages backward
```

- navigate 함수에 path 값을 넣을 때, 주소 이동 간 보존해야 할 state가 있다면 두 번째 인자(Optional)에 state 필드로 객체를 전달한다.

```javascript
navigate("/home", { state });
```

- naviage 함수는 기본적으로 hitory push 방식으로 동작한다. 만약 history replace 방식으로 사용하고 싶다면, 두 번째 인자를 넣을 때 replace: true 필드를 같이 전달한다.

```javascript
navigate("/home", { state, replace: true });
```

#### 3-2. Navigate Component

- v6에서는 Navigate Component를 사용한다.

```javascript
// react-router v6
import { Navigate } from "react-router-dom";

const LoginPage = () => {
  const user = useLoginContext();

  return (
    <>
      {user && <Navigate to="/home" replace />}
      <div>...</div>
    </>
  );
};
```

- v6의 Navigate Component 는 useNavigate hook 을 래핑한 컴포넌트이기 때문에 받아오는 props 가 useNavigate 의 인자와 동일하기 때문에 Navigate Component도 기본값으로 history push 를 사용하여 이동하므로, history replace 로 이동하기 위해서는 replace props를 넘겨줘야 한다.

```javascript
<Navigate to="/home" state={state} replace />
```

### 4. Relative routes & Nested routes

- v5에서는 useRouteMatch 를 통해 현재 경로를 가져온 뒤 path props에 라우트할 경로를 덧붙여 넣어주는 방식으로 경로를 지정해주어야 했다.

```javascript
// react-router v5
const App = () => {
  return (
    <Switch>
      <Route path={'/welcome'}>
        <WelcomePage />
      </Route>
      {/* OR */}
      {/* <Route path={'/welcome'} component={WelcomePage} /> */}
    </Switch>
  )
}

const Welcome = () => {
  const match = useRouteMatch()

  return (
    <>
      <p>Welcome Page</p>
      <Link to={`${match.path}/new-user`}>New User</Link>
      <Switch>
        <Route path={`${match.path}/new-user`} component={() => <p>Hi New User!</p>} />
      </Switch>
    <>
  )
}
```

- 하지만 v6부터는 Route 컴포넌트와 Link 컴포넌트가 부모의 라우트 경로를 보고 이로부터 자신의 경로를 빌드하므로, 더 이상 처음부터 경로를 만들 필요가 없다.

```javascript
// react-router v6
const App = () => {
  return (
    <Routes>
      <Route path={'/welcome/*'} element={<WelcomePage />} />
      {/*
        path={'/welcome'} won't work here, as the new router uses "exact" route matching.
        concatenate "/*" to indicate there are more paths to be matched in "WelcomePage".
      */}
    </Routes>
  )
}

const Welcome = () => {
  return (
    <>
      <p>Welcome Page</p>
      <Link to="/new-user">New User</Link>
      {/* automatically builds path "/welcome/new-user" */}
      <Routes>
        <Route path="/new-user" element={<p>Hi New User!</p>} />
      </Routes>
    <>
  )
}
```

- v6에서는 Route 의 자식 컴포넌트로 Route 컴포넌트로 둘 수 있다. 들여쓰기를 통해 가독성을 확보할 수 있고, `<Outlet />` 을 통해 하위 라우트 컴포넌트를 그려줄 곳을 명시할 수 있다. Outlet 컴포넌트가 위치한 곳에 render가 된다.
- Route Component의 children props는 nested route 기능을 위해 예약이 되어 있기 때문에 element props로 component를 내려준다.
- 이제 단독으로 Route Component를 사용할 수 없으며, Route Compoennt는 무조건 Routes Component로 감싸져 있어야 한다.

```javascript
// react-router v6
const App = () => {
  return (
    <Routes>
      <Route path={'/welcome'} element={<WelcomePage />}>
	      // notice the absence of trailing "/*": the router can explicitly see there is a nested route
        <Route path={'/new-user'} element={<p>Hi New User!</p>} />
      </Route>
    </Routes>
  )
}

const Welcome = () => {
  return (
    <>
      <p>Welcome Page</p>
      <Link to={'/new-user'}>New User</Link>
      <Outlet /> <- indicates where the element props of the nested route should render
    <>
  )
}
```

### 5. useMatch replaces useRouteMatch

- useMatch 또한 새로 도입된 매치 알고리즘을 사용하기 때문에 무조건 패턴 경로를 인자로 넣어줘야 한다.

```javascript
import { useMatch } from 'react-router-dom'

const match = useMatch('/home')

if (match) {
  // current path is "/home"
  ...
} else {
  // current path is NOT "/home"
  ...
}
```

- 리턴하는 Match 타입의 객체가 변경되어 PathMatch 타입을 리턴한다.

```typescript
// react-router v5
interface match<ParamKey> {
  isExact: boolean;
  params: Params<ParamKey>;
  path: string;
  url: string;
}

// react-router v6
interface PathMatch<ParamKey> {
  params: Params<ParamKey>;
  pathname: string;
  pattern: PathPattern;
}

interface PathPattern {
  path: string;
  caseSensitive?: boolean;
  end?: boolean;
}
```

### 6. No Optional params (nested route)

- 더 이상 경로 Parameter 를 Optional 하게 선택하지 못하게 되었다.
- 이 경우 /products/12345 경로는 상위 ProductDetail 라우트에 매치되어 productId:”12345” 파라미터를 들고 ProductDetail Component를 렌더할 것이고, /products 경로는 하위 ProductDetail 라우트에 매치되어 productId: null 파라미터를 들고 ProductDetail Component를 렌더할 것이다.

```javascript
<Route path="/products">
  <Route path=":productId" element={<ProductDetail />} />
  <Route path="" element={<ProductDetail />} />
</Route>
```
