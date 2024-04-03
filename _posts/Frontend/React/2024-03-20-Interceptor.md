---
title: "Axios Interceptor: HTTP 요청 효율화의 핵심"
date: 2024-03-20 19:10:40 +0900
categories: [Frontend, React]
tags: [JavaScript, React]
---

## Interceptor 란?

---

Axios 의 interceptor 는 HTTP 요청과 응답을 가로채고 수정하는 기능을 제공한다. 이를 통해 요청을 서버로 보내기 전이나 서버로부터 응답을 받은 후에 다양한 작업을 수행할 수 있다.

axios 에서는 두 가지 유형의 interceptor 를 제공한다.

> **Request Interceptor (요청 인터셉터)**

먼저 요청 인터셉터는 axios.interceptors.request.use( ) 를 통해 등록할 수 있다. 이 인터셉터는 요청을 보내기 전에 실행되며, 주로 요청 헤더를 수정하거나 요청 데이터를 변형하는 작업을 수행한다.

예를 들어, 모든 요청에 특정한 헤더를 추가하거나 인증 토큰을 포함시키는 작업이 요청 인터셉터를 사용하여 수행 될 수 있다.

> **Response Interceptor (응답 인터셉터)**

그리고, 응답 인터셉터는 axios.interceptors.response.use( ) 를 통해 등록 할 수 있다. 이 인터셉터는 서버로부터 응답을 받은 후에 실행되며, 주로 받은 응답 데이터를 가공하거나 오류를 처리하는 작업을 수행한다.

예를 들어, 서버로부터 받은 데이터를 특정 형식으로 변환하거나 오류 상태 코드에 따라 적절한 처리를 수행하는 등의 작업을 응답 인터셉터를 사용하여 할 수 있다.

![interceptor.png](/assets/img/post_img/coding/react/interceptor.png){: width="600px" align="center"}

<center>출처: https://medium.com/@barisberkemalkoc/axios-interceptor-intelligent-db46653b7303</center>

<br />

## Interceptor 를 사용하는 이점

---

axios 의 interceptor 아래와 같은 특징들로 다양한 HTTP 통신 상황에서 매우 유용하게 활용 될 수 있다.

### 1. 일관된 데이터 변형

- 모든 요청이나 응답에 동일한 전처리나 후처리 작업을 적용할 수 있습니다. 이는 요청이나 응답에 대해 일관된 데이터 형식을 유지하거나 특정 데이터를 가공하여 사용할 때 유용하다.

<br />

### 2. 재사용성 및 유지보수성 향상

- Interceptor 를 사용하여 중복 코드를 제거하고 코드의 재사용성과 유지보수성을 향상시킬 수 있다.
- 예를 들어, 인증 헤더를 추가하는 작업이 모든 요청에서 반복될 경우, 인터셉터를 사용하여 이를 한 곳에서 처리할 수 있다.

<br />

### 3. 보안 및 오류 처리

- 인터셉터를 사용하여 보안 관련 작업을 수행하거나 서버로부터 오는 오류를 처리할 수 있습니다.
- 예를 들어, 모든 요청에 인증 토큰을 추가하여 보안을 강화하거나, 응답 인터셉터를 사용하여 서버에서 오는 특정 상태 코드에 대한 처리를 수행할 수 있습니다.

<br />

### 4. 로깅 및 모니터링

- 인터셉터를 사용하여 모든 요청과 응답에 대한 로깅을 구현하거나, 요청 및 응답 시간을 측정하여 성능을 모니터링할 수 있습니다. 이를 통해 애플리케이션의 동작을 추적하고 문제를 식별하는 데 도움이 됩니다.

<br />

### 5. 기능 확장

- axios의 interceptor를 사용하면 라이브러리의 기능을 확장할 수 있습니다. 예를 들어, 요청이나 응답에 대한 캐싱, 미리 처리된 데이터를 반환하는 등의 기능을 추가할 수 있습니다.

<br />

## Interceptor 구현 코드

---

```jsx
const instance = axios.create({
  baseURL: getBaseUrl(baseURL()),
  headers: {
    "Content-Type": "aplication/json",
    accToken
  },
  timeout: 180000
});

// axios inst
export const commonAxios = ({
  serviceTypeURLName = getServiceTypeURLName(),
  type,
  path,
  headers,
  params = "",
  body = {},
  ...rest
}) => {
  const locale = JSON.parse(localStorage.getItem("locale"))?.locale ?? null;

  return instance({
    method: type,
    url: `${serviceTypeURLName}${path}${params}`,
    headers: {
      ...headers,
      countryCd: "kr",
      locale: locale
    },
    data: body,
    ...rest
  });
};

// axios header Authorization 설정
const axiosAuthorize = () => {
  instance.interceptors.request.use(
    (config) => {
      if (validRefreshToken()) {
        const token = useToken();
        config.headers.Authorization = `Bearer ${token?.tokenId}`;
        return config;
      } else {
        config.headers.Authorization = "";
        return config;
      }
    },
    (error) => {
      return Promise.reject(error);
    }
  );

  instance.interceptors.response.use(
    (response) => {
      // 200에 대한 처리
      if (response.data.status === 403) {
        alert("접근 권한이 없습니다.");
        goBack();
        return;
      } else if (response.data.status === 401) {
        alert(response.data.message);
        return;
      } else if (response.data.status === 404) {
        alert(response.data.message);
        return;
      }
      return response;
    },
    async (error) => {
      const {
        config,
        response: { status }
      } = error;
      const originalRequest = config;
      if (error.code === "ECONNABORTED") {
        push("/504");
      } else {
        if (status === 401) {
          if (!getAxiosFlag()) {
            alert("세션이 만료되었습니다. 다시 로그인 해주세요.");
            store.dispatch({ type: LOGOUT });
            setAxiosFlag(true);
          }
          throw error;
        } else if (status === 503) {
          push("/503");
          throw error;
        } else if (status === 504) {
          push("/504");
          throw error;
        } else if (status === 403) {
          alert("접근 권한이 없습니다.");
          goBack();
          throw error;
        } else if (status === 404) {
          alert("해당 URL을 찾을 수 없습니다.");
          throw error;
        } else if (status === 413) {
          alert("파일 업로드 용량을 초과하였습니다.");
          throw error;
        }
      }

      throw error;
    }
  );
};
```
