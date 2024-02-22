---
title: "[React] NextJS 13 / 14 version 특이점"
date: 2024-02-21 15:30:30 +0900
categories: [JavaScript, React]
tags: [JavaScript, React]
---

## Next 새버전 출시

Vercel 에서 1년을 주기로 Next 13[^Next13]에 이어 Next 14[^Next14] 버전을 출시하였다.  
13 버전 이후로 업데이트되면서 이전의 NextJS 구성 방식과는 많은 차이가 생겼는데 주요 특징을 알아보겠다.

---

## NextJS 13 불러온 변화

### 1. Routing

> Page Router -> App Router

13 버전이 등장하며 가장 큰변화가 생긴 부분 중 하나는 페이지 라우팅 방식이다.
이번 버전까지는 `pages/` 디렉토리에 파일들을 생성하며 페이지를 구성했었지만 13 버전 부터는 `app/` 디렉토리를 제공하며 다음의 추각적인 기능들을 제공하여 라우팅 방식의 변화를 보여준다.

- Layout
- Server Component
- Streaming

Next 13 이전 버전에서의 기본 폴더 구조는 다음과 같았다.

![Currying Image](/assets/img/post_img/coding/react/nextjs_1.JPG){: width="500" class="normal"}

Next 13 이후 버전 부터는 App Router 를 사용한 방식으로 아래와 같이 변경되었다.

![Currying Image](/assets/img/post_img/coding/react/nextjs_2.JPG){: width="500" class="normal"}

### 2. Data Fetching

> getServerSideProps/getStaticProps -> async fetch/cache/next revalidate

13 이전 버전까지는 getServerSideProps / getStaticsProps 등을 정의하며 SSR과 SSG 렌더링 방식을 활용하였다.

```javascript
export const getServerSideProps = async () => {
  try {
    const response = await fetch("/api/test/server");

    if (response.status === 200) {
      const data = await response.json();
      return {
        props: { data }
      };
    }

    return { props: {} };
  } catch (error) {
    console.error(error);
    return { props: {} };
  }
};

export const getStaticProps = async () => {
  try {
    const response = await fetch("/api/test/static");

    if (response.status === 200) {
      const data = await response.json();
      return {
        props: { data }
      };
    }

    return { props: {} };
  } catch (error) {
    console.error(error);
    return { props: {} };
  }
};
```

13 버전 이후부터는 fetch() API 옵션을 적용하여 Component 단위에서도 SSR, SSG 렌더링 방식을 활용할 수 있게 되었다.

```javascript
export const DataFetch = async () => {
  const staticDataFetch = await fetch("/api/test/static", {
    cache: "force-cache"
  });

  const dynamicDataFetch = await fetch("/api/test/server", {
    cache: "no-store"
  });

  const revalidateDataFetch = await fetch("/api/test/revalidate", {
    next: { revalidate: 100 }
  });
};
```

---

## NextJS 14 의 주요 변화

### 1. 메타데이터 설정 변경

App Router 에서는 메타데이터를 Layout 과 Page 에 따로 작성한다. 그래서 기존 13.4 버전에서는 description 이나 og tag 와 같은 기본적인 메타태그는 물론 viewport, colorScheme, themeColor 모두 Metadata 에서 설정해서 사용해야 했다.
하지만 14 버전에서는 유저 경험에 영향을 주는 viewport, colorScheme, themeColor 정보는 기존 Metadata 타입과 분리되어 별도로 정의해야 한다. 다른 메타데이터와 달리 페이지 콘텐츠를 서버에서 내려받기 전에 필요한 해당 필드는 기존 Metadata 타입에서 만료 처리되었고, 추후 메이저 업데이트에서 제거될 예정이다.

```javascript
import { Viewport } from "next";

export const viewport: Viewport = {
  themeColor: "black"
};

export default function Page() {}
```

### 2. 서버 액션 안정화

장기적으로 가장 발전이 기대되는 부분은 서버 액션 기능이다. NextJS 는 React Framework 라서 프론트엔드만 사용한다고 생각하는 분들이 많지만, NextJS 예전부터 API 라우터도 제공하고 있다.

NextJS 는 여기서 더 나아가 서버 액션을 통해 API 라우터를 별도로 생성하지 않아도 되도록 만들었다.
동일한 NextJS 프로젝트에서만 사용할 거라면, 별도로 API 라우터까지 만들지 않고 서버 컨포넌트에서 한 번에 데이터베이스에 접근이 가능하도록 제공한다.
13.4에서도 서버 액션을 제공하긴 했지만, 사용하기 위해선 next.config.js에 따로 설정이 필요했다. 이제 Next.js 14 버전 부터는 서버 액션이 안정화 되면서 해당 설정 없이도 서버 액션 기능을 이용할 수 있다.

<br />
<br />
<br />

[^Next13]: 2022년 10월 25일 출시
[^Next14]: 2023년 10월 26일 출시
