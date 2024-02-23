---
title: "트랜잭션 전파(Transaction Propagation)란 무엇일까? - (3)"
date: 2024-02-18 15:44:30 +0900
categories: [Spring, Transaction]
tags: [Java, Spring]
---

## Transaction Propagation - 내부 롤백

---

물리 트랜잭션은 외부 트랜잭션에 의해 커밋 / 롤백 을 한다. 즉 내부 트랜잭션에 의한 롤백의 경우 물리 트랜잭션에 영향을 주지 못한다. 스프링은 내부 트랜잭션에 의한 롤백 문제를 어떻게 해결할까?

```java
@Test
void inner_rollback() {
	log.info("외부 트랜잭션 시작");
	TransactionStatus outer = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());
	log.info("내부 트랜잭션 시작");
	TransactionStatus inner = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());

	log.info("내부 트랜잭션 롤백");
	platformTransactionManager.rollback(inner); // rollback-only 마킹

	log.info("외부 트랜잭션 커밋");
	assertThatThrownBy(() -> platformTransactionManager.commit(outer)).isInstanceOf(UnexpectedRollbackException.class);
}
```

![Currying Image](/assets/img/post_img/coding/spring/transaction/transaction_propagation_3_1.png){: width="500" align="center"}

```txt
# 실행 결과 - inner_rollback()
외부 트랜잭션 시작
Creating new transaction with name [null]:
PROPAGATION_REQUIRED,ISOLATION_DEFAULT
Acquired Connection [HikariProxyConnection@220038608 wrapping conn0] for JDBC
transaction
Switching JDBC Connection [HikariProxyConnection@220038608 wrapping conn0] to
manual commit
내부 트랜잭션 시작
Participating in existing transaction
내부 트랜잭션 롤백
Participating transaction failed - marking existing transaction as rollback-only
Setting JDBC transaction [HikariProxyConnection@220038608 wrapping conn0]
rollback-only
외부 트랜잭션 커밋
Global transaction is marked as rollback-only but transactional code requested
commit
Initiating transaction rollback
Rolling back JDBC transaction on Connection [HikariProxyConnection@220038608
wrapping conn0]
Releasing JDBC Connection [HikariProxyConnection@220038608 wrapping conn0] after
transaction
```

- 외부 트랜잭션 시작
  - 물리 트랜잭션을 시작
- 내부 트랜잭션 시작
  - `Participating in existing transaction`
  - 기존 트랜잭션에 참여
- 내부 트랜잭션 롤백
  - `Participating transaction failed - marking existing transaction as rollback-only`
  - 내부 트랜잭션을 롤백하면 실제 물리 트랜잭션은 롤백하지 않는다. 대신에 기존 트랜잭션을 롤백 전용으로 표시한다.
- 외부 트랜잭션 커밋
  - 외부 트랜잭션을 커밋
  - `Global transaction is marked as rollback-only`
  - 커밋을 호출했지만, 전체 트랜잭션이 롤백 전용으로 표시되어 있다. 따라서 물리 트랜잭션을 롤백

## Transaction Propagation - 내부 롤백 흐름

---

![Currying Image](/assets/img/post_img/coding/spring/transaction/transaction_propagation_3_2.png){: width="500" align="center"}

1. 로직2가 끝나고 트랜잭션 매니저를 통해 내부 트랜잭션을 롤백
2. 트랜잭션 매니저는 롤백 시점에 신규 트랜잭션 여부에 따라 다르게 동작한다. 이 경우 신규 트랜잭션이 아니기 때문에 실제 롤백을 호출하지 않는다.
   - 실제 커넥션에 커밋이나 롤백을 호출하면 물리 트랜잭션 종료
   - 아직 트랜잭션이 끝난 것이 아니기 때문에 실제 롤백을 호출하면 안되고, 물리 트랜잭션은 외부 트랜잭션을 종료할 때 까지 이어져야함
3. 내부 트랜잭션은 물리 트랜잭션을 롤백하지 않는 대신에 트랜잭션 동기화 매니저에 `rollbackOnly=true` 라는 표시를 해둔다.

4. 로직1이 끝나고 트랜잭션 매니저를 통해 외부 트랜잭션을 커밋한다.
5. 트랜잭션 매니저는 커밋 시점에 신규 트랜잭션 여부에 따라 다르게 동작한다. 외부 트랜잭션은 신규 트랜잭션이다.
   따라서 DB 커넥션에 실제 커밋을 호출해야 한다. 이때 먼저 트랜잭션 동기화 매니저에 롤백 전용 (`rollbackOnly=true` ) 표시가 있는지 확인하고 롤백 전용 표시가 있으면 물리 트랜잭션을 커밋하는 것이 아니라 롤백처리
6. 실제 데이터베이스에 롤백이 반영되고, 물리 트랜잭션도 종료
7. 트랜잭션 매니저에 커밋을 호출한 개발자 입장에서는 분명 커밋을 기대했는데 롤백 전용 표시로 인해 실제로는 롤백
   - **시스템 입장에서는 커밋을 호출했지만 롤백이 되었다는 것은 분명하게 알려주어야 하기 때문에 매우 중요한 문제**
   - 예를 들어서 고객은 주문이 성공했다고 생각했는데, 실제로는 롤백이 되어서 주문이 생성되지 않은 것이다. 스프링은 이 경우 `UnexpectedRollbackException` 런타임 예외를 던진다. 그래서 커밋을 시도했지만, 기대하지 않은 롤백이 발생했다는 것을 명확하게 알려준다.

## Transaction Propagation - 내부 롤백 정리

---

- 논리 트랜잭션이 하나라도 롤백되면 물리 트랜잭션은 롤백된다.
- 내부 논리 트랜잭션이 롤백되면 롤백 전용 마크를 표시한다.
- 외부 트랜잭션을 커밋할 때 롤백 전용 마크를 확인한다. 롤백 전용 마크가 표시되어 있으면 물리 트랜잭션을 롤백 하고, `UnexpectedRollbackException` 예외를 던진다.

---

**[출처]** [김영한 스프링 DB 2편 - 데이터 접근 활용기술](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2/dashboard)
