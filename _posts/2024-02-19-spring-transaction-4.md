---
title: "트랜잭션 전파(Transaction Propagation)란 무엇일까? - (4)"
date: 2024-02-19 20:20:30 +0900
categories: [Spring, Transaction]
tags: [Java, Spring]
---

## 옵션 - REQUIRES_NEW

스프링에는 외부 트랜잭션과 내부 트랜잭션을 완전히 분리해서 사용하기 위한 옵션이 존재한다. 외부 트랜잭션과 내부 트랜잭션을 완전히 분리해서 각각 별도의 물리 트랜잭션을 사용하는 방법으로 (commit / rollback) 이 각각 별도로 이루어지게 된다.
이 옵션은 내부 트랜잭션에 문제가 발생해서 롤백해도, 외부 트랜잭션에는 영향을 주지 않는다. 반대로 외부 트랜잭션에 문제가 발생해도 내부 트랜잭션에 영향을 주지 않는다.

![Currying Image](/assets/img/post_img/coding/spring/transaction/transaction_propagation_4_1.png){: width="500" align="center"}

```java
@Test
void inner_rollback_requires_new() {
	log.info("외부 트랜잭션 시작");
	TransactionStatus outer = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());
	log.info("outer.isNewTransaction()={}", outer.isNewTransaction()); // true

	log.info("내부 트랜잭션 시작");
	DefaultTransactionAttribute defaultTransactionAttribute = new DefaultTransactionAttribute();
	/**
	 * 내부 트랜잭션을 시작할 때 전파 옵션인 propagationBehavior 에 PROPAGATION_REQUIRES_NEW 옵션 부여
	 * - 기존 트랜잭션을 무시하고 새로운 물리 트랜잭션을 만들어서 시작
	 */
	defaultTransactionAttribute.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
	TransactionStatus inner = platformTransactionManager.getTransaction(defaultTransactionAttribute);
	log.info("inner.isNewTransaction()={}", inner.isNewTransaction()); // true

	log.info("내부 트랜잭션 롤백");
	platformTransactionManager.rollback(inner); // 롤백

	log.info("외부 트랜잭션 커밋");
	platformTransactionManager.commit(outer); // 커밋
}
```

- 내부 트랜잭션을 시작할 때 전파 옵션인 `propagationBehavior` 에 `PROPAGATION_REQUIRES_NEW` 옵션 을 주었다. 이 전파 옵션을 사용하면 내부 트랜잭션을 시작할 때 기존 트랜잭션에 참여하는 것이 아니라 새로운 물리 트랜잭 션을 만들어서 시작하게 된다.

```txt
외부 트랜잭션 시작
Creating new transaction with name [null]:
PROPAGATION_REQUIRED,ISOLATION_DEFAULT
Acquired Connection [HikariProxyConnection@1064414847 wrapping conn0] for JDBC transaction
Switching JDBC Connection [HikariProxyConnection@1064414847 wrapping conn0] to manual commit
outer.isNewTransaction()=true
내부 트랜잭션 시작
Suspending current transaction, creating new transaction with name [null]
Acquired Connection [HikariProxyConnection@778350106 wrapping conn1] for JDBC transaction
Switching JDBC Connection [HikariProxyConnection@778350106 wrapping conn1] to manual commit
inner.isNewTransaction()=true
내부 트랜잭션 롤백
Initiating transaction rollback
Rolling back JDBC transaction on Connection [HikariProxyConnection@778350106 wrapping conn1]
Releasing JDBC Connection [HikariProxyConnection@778350106 wrapping conn1] after transaction
Resuming suspended transaction after completion of inner transaction
외부 트랜잭션 커밋
```

- **외부 트랜잭션 시작**
  외부 트랜잭션을 시작하면서 `conn0` 를 획득하고 `manual commit` 으로 변경해서 물리 트랜잭션을 시작한다. 외부 트랜잭션은 신규 트랜잭션이다.( `outer.isNewTransaction()=true` )
- **내부 트랜잭션 시작**
  - 내부 트랜잭션을 시작하면서 `conn1` 를 획득하고 `manual commit` 으로 변경해서 물리 트랜잭션을 시작한다.
  - 내부 트랜잭션은 외부 트랜잭션에 참여하는 것이 아니라, `PROPAGATION_REQUIRES_NEW` 옵션을 사용했기 때 문에 완전히 새로운 신규 트랜잭션으로 생성된다.(`inner.isNewTransaction()=true` )
- **내부 트랜잭션 롤백**
  - 내부 트랜잭션을 롤백한다.
  - 내부 트랜잭션은 신규 트랜잭션이기 때문에 실제 물리 트랜잭션을 롤백한다.
  - 내부 트랜잭션은 `conn1` 을 사용하므로 `conn1` 에 물리 롤백을 수행한다.
- **외부 트랜잭션 커밋**
  - 외부 트랜잭션을 커밋한다.
  - 외부 트랜잭션은 신규 트랜잭션이기 때문에 실제 물리 트랜잭션을 커밋한다.
  - 외부 트랜잭션은 `conn0` 를 사용하므로 `conn0` 에 물리 커밋을 수행한다.

---

## **요청 흐름 - REQUIRES_NEW**

![Currying Image](/assets/img/post_img/coding/spring/transaction/transaction_propagation_4_2.png){: width="500" align="center"}

1. `txManager.getTransaction()` 를 호출해서 외부 트랜잭션을 시작한다.
2. 트랜잭션 매니저는 데이터소스를 통해 커넥션을 생성한다.
3. 생성한 커넥션을 수동 커밋 모드( `setAutoCommit(false)` )로 설정한다. - **물리 트랜잭션 시작**
4. 트랜잭션 매니저는 트랜잭션 동기화 매니저에 커넥션을 보관한다.
5. 트랜잭션 매니저는 트랜잭션을 생성한 결과를 `TransactionStatus` 에 담아서 반환하는데, 여기에 신규 트랜잭션의 여부가 담겨 있다. `isNewTransaction` 를 통해 신규 트랜잭션 여부를 확인할 수 있다. 트랜잭션을 처음 시작했으므로 신규 트랜잭션이다.( `true` )
6. 로직1이 사용되고, 커넥션이 필요한 경우 트랜잭션 동기화 매니저를 통해 트랜잭션이 적용된 커넥션을 획득해서 사용한다.

7. **REQUIRES_NEW 옵션**과 함께 `txManager.getTransaction()` 를 호출해서 내부 트랜잭션을 시작한다.

- 트랜잭션 매니저는 `REQUIRES_NEW` 옵션을 확인하고, 기존 트랜잭션에 참여하는 것이 아니라 새로운 트 랜잭션을 시작한다.

9. 트랜잭션 매니저는 데이터소스를 통해 커넥션을 생성한다.
10. 생성한 커넥션을 수동 커밋 모드( `setAutoCommit(false)` )로 설정한다. - **물리 트랜잭션 시작**
11. 트랜잭션 매니저는 트랜잭션 동기화 매니저에 커넥션을 보관한다.
    이때 `con1` 은 잠시 보류되고, 지금부터는 `con2` 가 사용된다. (내부 트랜잭션을 완료할 때 까지 `con2` 가 사용된다.)
12. 트랜잭션 매니저는 신규 트랜잭션의 생성한 결과를 반환한다. `isNewTransaction == true`
13. 로직2가 사용되고, 커넥션이 필요한 경우 트랜잭션 동기화 매니저에 있는 `con2` 커넥션을 획득해서 사용한
    다.

---

## **응답 흐름 - REQUIRES_NEW**

![Currying Image](/assets/img/post_img/coding/spring/transaction/transaction_propagation_4_3.png){: width="500" align="center"}

1. 로직2가 끝나고 트랜잭션 매니저를 통해 내부 트랜잭션을 롤백
2. 트랜잭션 매니저는 롤백 시점에 신규 트랜잭션 여부에 따라 다르게 동작한다.
   - 현재 내부 트랜잭션은 신규 트랜잭션이기 때문에 실제 롤백을 호출한다.
3. 내부 트랜잭션이 `con2` 물리 트랜잭션을 롤백
   - 트랜잭션이 종료되고, `con2` 는 종료되거나, 커넥션 풀에 반납
   - 이후에 `con1` 의 보류가 끝나고, 다시 `con1` 을 사용
4. 외부 트랜잭션에 커밋을 요청
5. 외부 트랜잭션은 신규 트랜잭션이기 때문에 물리 트랜잭션을 커밋
6. 이때 `rollbackOnly` 설정을 체크하고, `rollbackOnly` 설정이 없으므로 커밋
7. 본인이 만든 `con1` 커넥션을 통해 물리 트랜잭션을 커밋
   - 트랜잭션이 종료되고, `con1` 은 종료되거나, 커넥션 풀에 반납

---

## REQUIRES_NEW 옵션 정리

- `REQUIRES_NEW` 옵션을 사용하면 물리 트랜잭션이 명확하게 분리된다.
- `REQUIRES_NEW` 를 사용하면 데이터베이스 커넥션이 동시에 2개 사용된다는 점을 주의해야 한다.

---

**[출처]** [김영한 스프링 DB 2편 - 데이터 접근 활용기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2/dashboard)
