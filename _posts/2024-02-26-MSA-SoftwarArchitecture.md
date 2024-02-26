---
title: "MSA 와 Software Architecture 의 관계"
date: 2024-02-26 14:54:30 +0900
categories: [MSA, Architecture]
tags: [MSA, Project, spring]
---

## 1. MSA 환경에서 소프트웨어아키텍처가 중요한 이유

- MSA 는 모놀리스부터 분리/분해 된 Micro Service 들의 모임
- Micro Services 들을 이루는 비즈니스 / 도메인은 시간이 흐름에 따라 성격, 복잡도 등이 변함
- 개별적인 Micro Service 내에서도 비즈니스적인 목적에 따라 별개의 기능, 효율을 중요시 해야하는 상황 발생할 수 있다.

> Micro Service 를 이루는 소프트웨어 아키텍처에 따라, 각 Micro Service의 Business Capabilities 변화 대응할 수 있다.

<br >

## 2. MSA === Sofiware Architecture

- MSA micro services 들로 이루어져 있고 micro services 는 Sofiware Architecture 로 Dependency 가 걸려서 만들어지므로 넓은 의미로 MSA는 소프트웨커 아키텍처라고 할 수 있다.

<br >

### 유연성, 확장성, 낮은 응집도/결합도 등 MSA 의 특성이 각 Micro Service 안에서도 필요한 이유

1. 당장 요청에 대한 완료 응답을 할 필요는 없지만,높은 성능을 요구하는 작업
   - 대규모 Command 작업을 수행하는 Cron Job
2. 당장 응답해야만 하지만, 적은 성능을 요구하는 작업
   - 일반적인 조회-Query 서비스
3. 호출하던 외부 서비스의 스펙이 변경되는 경우
   - Request/Response Params 가 변동 되었고, 영향도 없이 빠르게 적용이 필요한 경우
   - 성능이 조정되어, 최적의 Connection Pool 갯수 혹은 timeout 수치가 변경되는 경우
4. 통신 방식이 변경되는 경우
   - Sync 방식에서 Async 방식으로 변경되어 비동기 Callback 방식으로 같은 로직을 처리할 경우

<br>

![msa_sa_1.png](/assets/img/post_img/coding/architecture/msa_sa_1.png){: width="600" align="center"}

<center>[그림 .1] MSA는 느슨하게 결합된 서비스의 집합 </center>

<br>

## 3. Layered Architecture (계층화 아키텍처)

- 대표적인 Software Architecture 중 하나

### Layered Architecture 란?

> 구현에 대한 로직 의존성은 단방향, 접근은 양방향

- 일반적인 회사에서 적용되어지고 있는 보편적인 아키텍처 이다.
- 여러개의 계층으로 나누어 각 계층에서 하는 일을 한정시켜 계층별로 독립적으로 개발, 배포, 확장이 가능
- Presentation / Application / Domain / Infra Layer 등으로 이루어져 있다.
  - 각 Layer 입장에서는 하위계층으로 의존성 존재(아래 방향)
- 각 계층내에서는 코드 수정과 유지 보수 등이 비교적 쉽고 계층간 접근이 자유롭다.
  - 계층간 접근의 자유도에 의해 완전히 다른 계층의 메서드 call 가능성이 존재한다.

![msa_sa_2.png](/assets/img/post_img/coding/architecture/msa_sa_2.png){: width="600" align="center"}

<center>[그림 .2] Layered Architecture </center>

<br>

### Layered Architecture 장점

- 비교적 자유로운 접근 방향성으로 인한 빠른 개발, 코드의 재사용 그리고 편한 테스팅 가능한다.
  - 그러나 처음 개발 이후, 큰 비즈니스 로직의 변화가 존재하지 않는다면 오히려 안정적

<br>

### Layered Architecture 단점

- DB 로 인한 각 계층의 역할, 목적과 계층 간 의존성 구분이 희미해진다.

<br>

## 4. Hexagonal Architecture (육각형 아키텍처)

### Hexagonal Architecture 란?

- 각 계층에서 하던 일들을 `내부` 와 `외부` 라는 개념으로 나누어 각각에 맞는 별도의 인터페이스를 정의한다.
  - Adapter / Port
- `내부` 의 로직은 오직 `외부` 인터페이스를 통해서만 접근이 가능하다.
  - 모든 외부 시스템과의 직접적인 상호작용은 `Adapter` 의 역할
  - 각 서비스에서 비즈니스 로직에 맞게 정의된 인터페이스는 `Port` 이다.
  - 즉, 외부 서비스와의 상호작용(**Adapter**)는 비즈니스 로직과의 작업을 정의한 인터페이스(**Port**)랑만 서로 통신
- 모든 비즈니스 로직은, 오직 외부에서 내부 방향 / 내부에서 외부 방향으로만 호출이 가능해요.
  - Inbound Adapter --> Inbound Port --> 비즈니스 로직
  - 비즈니스 로직 --> Outbound Port --> Outbound Adapter

<br>

### Adapter

- 서비스의 입장에서 이 서비스가 사용하는 외부 시스템과의 직접적인 구현 및 상호작용을 처리
- Inbound Adapter
  - 외부 시스템(UI)으로 부터 들어온 Request가 가장 처음 만나는 Controller
  - 메세지브로커(kafka)로 부터 Consume하는동작을 처리하는 Logic Handler
- Outbound Adapter
  - DB(MySQL 등등) 에 직접적으로 접근하여 다양한 작업(CRUD) 을 처리하기 위한 DAO

<br>

### Port

- 비즈니스 로직 입장에서 **Adapter**와 통신하기 위한 동작을 정의한 Interface
- Inbound Port
  - Controller 로부터 들어온 요청으로 부터 특정 비즈니스 로직을 수행하기 위한 동작을 정의한 인터페이스
  - Consume한 메세지를 처리하기 위한 비즈니스 로직의 동작을 정의한 인터페이스
- Outbound Port
  - 비즈니스 로직에서 DB 접근을 위해서 정의한 Repository 인터페이스는

<br>

![msa_sa_3.png](/assets/img/post_img/coding/architecture/msa_sa_3.png){: width="600" align="center"}

<center>[그림 .3] 도식화된 Hexagonal Architecture </center>
<br>

### Hexagonal Architecture 의의

- `Adapter` 를 통해 외부 서비스의 의존성을 분리하여 언제든 쉽게 교체하여 유연한 확장성 있는 대처를 하고, `Port` 를 통해서 내부 비즈니스 로직과 인터페이스를 분리하여 내부 로직의 구현은 인터페이스와 무관하게 개발 가능하도록 하는 것

<br>

## 5. 결론

- MSA 는 다수의 Micro Service 의 소프트웨어 아키텍처의 모음
- MSA 의 최대 목적인 Business Capabilities를 만족하기 위한 Key는 빠른 확장성과 유연성
- MSA 를 이루는 각 Micro Service 도 높은 확장성과 유연성이 필수적

> 즉, Hexagonal Architecture 는 MSA 환경에서 매우 적절한 Software Architecture 이다.
