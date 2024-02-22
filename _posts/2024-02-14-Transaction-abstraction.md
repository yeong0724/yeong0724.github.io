---
title: "[Spring] Transaction 추상화 & 동기화"
date: 2024-02-14 21:10:30 +0900
categories: [Spring, Transaction]
tags: [Java, Spring]
---

## Application 구조

---

여러가지 애플리케이션 구조가 있지만, 가장 단순하면서 많이 사용하는 방법은 역할에 따라 아래 그림과 같이 3가지 계층으로 나누는 구조이다.

![Currying Image](/assets/img/post_img/coding/spring/transaction_1.JPG){: width="600" align="center"}

- 여기서 가장 중요한 곳은 어디일까? 바로 핵심 비즈니스 로직이 들어있는 서비스 계층이다. 시간이 흘러서 UI(웹) 와 관련된 부분이 변하고, 데이터 저장 기술을 다른 기술로 변경해도, 비즈니스 로직은 최대한 변경없이 유지되어야 한다.
- 비즈니스 로직의 순수성을 유지하기 위해서는 최대한 특정 기술 구현에 종속적이지 않게 개발해야 한다.
- Application 구조를 계증으로 나눈 이유도 서비스 계층을 최대한 수순하게 유지하기 위한 목적이다.
- 서비스 계층이 특정 기술에 종속되지 않아야 비즈니스 로직을 유지보수 하기도 쉽고, 테스트 하기도 쉽다.

### 1. 프레젠테이션 계층

- 프레젠테이션 계층은 클라이언트가 접근하는 UI와 관련된 기술인 Web, Servlet, HTTP 와 관련된 부분을 담당해줌으로서 서비스 계층을 UI와 관련된 기술로 부터 보호해준다.
- 예를들어 HTTP API 를 사용하다가 gRPC 같은 기술로 변경해도 프레젠테이션 계층 코드만 변경하고, 서비스 계층의 코도는 변경하지 않아도 된다.

### 2. 데이터 접근 계층

- 데이터 접근 계층은 데이터를 저장하고 관리하는 기술을 담당해주므로써, `JDBC`, `JPA` 와 같은 구체적인 데이터 접근 기술로부터 서비스 계층을 보호해준다.
- 예를 들어서 JDBC를 사용하다가 JPA로 변경해도 서비스 계층은 변경하지 않아도 된다.
- 물론 서비스 계층에서 데이터 접근 계층을 직접 접근하는 것이 아니라, 인터페이스를 제공하고 서비스 계층은 이 인터페이스에 의존하는 것이 좋다. 그래야 서비스 코드의 변경 없이 JdbcRepository 를 JpaRepository 로 변경할 수 있다.

<br >

## 서비스 계층 Transaction 의 문제점

---

1. 트랜잭션을 구현하기 위한 특정 기술 의존 문제(ex. JDBC, JPA ...)
2. 예외 누수 문제 (ex. SQLException)
3. 반복되는 구현 코드

```java
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

@Slf4j
@RequiredArgsConstructor
public class MemberServiceVer1 {
	private final DataSource dataSource;
	private final MemberRepositoryV2 memberRepository;

	public void accountTransfer(String fromId, String toId, int money) throws SQLException {
	Connection con = dataSource.getConnection();
		try {
			con.setAutoCommit(false); //트랜잭션 시작
			//비즈니스 로직
			bizLogic(con, fromId, toId, money);
			con.commit(); //성공시 커밋
		} catch (Exception e) {
				con.rollback(); //실패시 롤백
				throw new IllegalStateException(e);
		} finally {
				release(con);
		}
	}
	private void bizLogic(Connection con, String fromId, String toId, int money) throws SQLException {
		Member fromMember = memberRepository.findById(con, fromId);
		Member toMember = memberRepository.findById(con, toId);
		memberRepository.update(con, fromId, fromMember.getMoney() - money);
		memberRepository.update(con, toId, toMember.getMoney() + money);
	}
}
```

- 트랜잭션은 코드와 같이 비즈니스 로직이 있는 서비스 계층에서 시작하는 것이 좋다.
- 문제는 트랜잭션을 사용하기 위해서 `javax.sql.DataSource` , `java.sql.Connection` ,
  `java.sql.SQLException` 같은 JDBC 기술에 의존해야한다.
- 위에 코드만 보더라도 트랜잭션을 적용하기 위해 JDBC 기술에 의존하며, JDBC를 사용해 트랜잭션을 처리하는 코드가 많아 졌다.
- 이러한 코드 방향은 JDBC에서 JPA 같은 다른 기술로 바뀌게 되면 서비스 코드도 모두 함께 변경해야 하는 상황이 발생하고, 이런식의 개발은 유지보수성을 망치는 길이다.
- 추가적으로 `try-catch-finally` , DB 연동 과정 등의 반복되는 코드들도 문제이다.

<br >

## Transaction의 추상화

앞서말한
