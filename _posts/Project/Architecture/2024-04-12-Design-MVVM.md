---
title: "[Design Pattern] MVVM"
date: 2024-04-12 23:20:30 +0900
lastmode: 2024-02-22 10:05:30 +0900
categories: [Project, Architecture]
tags: [Frontend, Project, Pattern]
---

## MVVM 패턴이란?

---

Model - View - ViewModel 의 약자로 프로그램의 비지니스 로직과 프레젠테이션 로직을 UI로 명확하게 분리하는 패턴이다.

데이터를 관리하는 로직과 UI로직을 분리하여 Application의 테스트와 유지 보수를 보다 용이하게 할 수 있게 되고, Code의 재사용성을 증대 시켜 UI Designer 와 협업을 보다 쉽게 만들어준다.

<br />

![mvvm_1.png](/assets/img/post_img/project/architecture/mvvm_1.png){: width="700" align="center"}

### 1. Model

데이터를 다루는 부분으로 비즈니스 로직을 포함한다. 데이터를 자져오고 저장하는 역할을 수행한다.
보통 데이터베이스, 네트워크 요청 또는 파일 시스템과 같은 데이터 소스와 상호 작용한다.

### 2. View

사용자 인터페이스를 담당하는 부분이다. 사용자가 보는 화면을 표시하고, 사용자 입력을 처리한다.
보통 HTML 언어를 사용하여 디자인된다.

### 3. ViewModel

View 와 Model 사이에서 중재자 역할을 수행한다. View 에서 발생하는 이벤트를 감지하고, 해당 이벤트에 맞는 비즈니스 로직을 수행한다. 그리고 View 에 표시할 데이터를 가공하여 제공하는 역할을 한다.

<br />

## MVVM 특징

---

언뜻 보기에는 MVP 와 비슷한 부분이 많다. 하지만 MVP 는 View 와 Presenter 사이의 의존관계가 1:1로 형성되어 있다면, MVVM 은 View 와 ViewModel 사이의 관계가 1:N 으로 되어 있다.
또한 데이터 바인딩을 이용한다면 View 와 ViewModel 사이의 의존성을 없앨 수 있다.

<br />

## MVVM 단점

---

- View가 커질 수록, ViewModel이 커질 수 있다.
- Application 이 커지면 Data Binding 때문에 메모리 소모가 빨라진다.
- ViewModel의 설계가 어렵다.
- 즉, 단순한 프로젝트일 경우 적합하지 않은 패턴일 수 있다.
