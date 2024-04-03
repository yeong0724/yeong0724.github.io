---
title: "ERD 에 대해서 배우고 작성해보기"
date: 2024-02-28 23:53:30 +0900
categories: [Project, Development]
tags: [MSA, Project, spring]
---

## ERD 란 무엇일까?

---

**ERD**(`Entity-Relationship Diagram`)는 엔터티와 엔터티 간의 관계를 시각적으로 나타내는 데이터베이스 설계 도구로, 데이터베이스의 구조와 관계를 이해하고 표현하는 데 사용됩니다.

엔터티는 데이터베이스에서 정보를 나타내는 객체를 의미하며, 관계는 엔터티 간의 연관성을 표현합니다.

> [ERD 작성 사이트 링크](https://sqldbm.com/Home/)

<br>

## ERD 구성 요소

---

- Entity (엔티티)
  - 현실 세계에서 독립적으로 존재하고 식별 가능한 객체나 사물을 나타낸다.
  - 예를 들어, "사용자(User)", "주문(Order)", "제품(Product)" 등이 엔터티가 될 수 있습니다.
- Attribute (속성)
  - Entity 의 특성이나 속성을 나타낸다.
  - 예를 들어, `User` Entity 의 속성으로 "이름", "나이", "이메일" 등이 있을 수 있습니다.
- Relationship (관계)
  - Entity 간의 연결이나 연관성을 표현한다.
  - 관계에는 일대일(1:1), 일대다(1:N), 다대다(N:M) 등 다양한 유형이 있다.
- Primary Key
  - Entity 의 고유 식별자로 사용되는 속성을 나타낸다.
  - 각 Entity 는 **Primary Key** 를 가지고 있어야 한다.
- Foreign Key
  - 다른 테이블의 기본 키를 참조하는 속성으로, 관계를 나타내는 데 사용됩니다.

<br>

## 프로젝트 ERD 적용 예제

---

![erd_1.png](/assets/img/post_img/coding/development/erd_1.png){: width="600" align="center"}

<center>[그림 1] Board & tag ERD</center>
