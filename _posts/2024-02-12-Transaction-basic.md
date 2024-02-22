---
title: "[Spring] Transaction의 개념 이해"
date: 2024-02-12 22:25:30 +0900
categories: [Spring, Transaction]
tags: [Java, Spring]
---

# INTRO

데이터를 저장할 때, 그냥 파일에 저장하지 않고 데이터베이스에 저장하는 이유는 무엇일까?
가장 대표적인 이유로는 트랜잭션이라는 개념을 데이터베이스가 지원하기 때문이다.
트랜잭션은 말그대로 `거래` 라는 뜻인데 풀어서 말하자면 하나의 거래를 안전하게 처리하도록 보장해주는 것을 뜻한다.

그렇다면 거래에 있어서 안전한 처리가 무슨 뜻일까?
하나의 거래를 안전하게 처리하려면 생각보다 고려해야 할 점이 많다.

> A 에서 B 로 5,000원 계좌이체
>
> 1. A의 은행 잔고 이체 금액 만큼 감소
> 2. B의 은행 잔고 이체 금액 만큼 증가

이렇게 2가지 작업이 합쳐져 하나의 작업처럼 동작해야 한다. 만약 시스템장애로 인해 1번만 성공하고 2번은 실패하게 된다면 돈이 사라져 버리는 심각한 문제가 발생하게 된다.

데이터 베이스가 제공하는 트랜잭션 기능을 활용한다면 1, 2번 과정이 모두 성공해야 commit 되어 DB에 정상 반영이 되고, 작업중 일부가 실패하게 된다면 Rollback 처리를 통해 거래 이전으로 되돌릴수 있다.

<br >

## Transaction ACID

---

트랜잭션은 ACID([https://en.wikipedia.org/wiki/ACID](https://en.wikipedia.org/wiki/ACID))를 보장해야 한다.

### 원자성 (Atomicity)

- 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공 하거나 모두 실패해야 한다.

### 일관성 (Consistency)

- 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 한다. 예를 들어 데이터베이스에서 정한 무 결성 제약 조건을 항상 만족해야 한다.

### 격리성 (Isolation)

- 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리한다. 예를 들어 동시에 같은 데이터 를 수정하지 못하도록 해야 한다. 격리성은 동시성과 관련된 성능 이슈로 인해 트랜잭션 격리 수준(Isolation level)을 선택할 수 있다.

### 지속성 (Durability)

- 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다. 중간에 시스템에 문제가 발생해도 데이 터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 한다.

<br >

ACID 중 문제는 격리성이다. 완벽한 격리성을 보장하려면 트랜잭션을 사실상 순서대로 실행해야 한다. 하지만 이렇게 하게 되면 동시 처리 성능이 매우 나빠질 가능성이 있다. 이렇기 때문에 `ANSI 표준`은 트랜잭션의 겨리 수준을 4단게로 나누었다.

### **트랜잭션 격리 수준 - Isolation level**

- READ UNCOMMITED(커밋되지 않은 읽기)
- READ COMMITTED(커밋된 읽기)
- REPEATABLE READ(반복 가능한 읽기)
- SERIALIZABLE(직렬화 가능)

<br>

## 데이터베이스 연결 구조와 DB 세션

---

![Currying Image](/assets/img/post_img/coding/spring/transaction_basic_1.png){: width="600" align="center"}

- 사용자는 WAS나 DB 접근 툴 같은 클라이언트를 사용해서 데이터베이스 서버에 접근할 수 있다. 클라이언트는 데이터베이스 서버에 연결을 요청하고 커넥션을 맺게 되는데, 이때 데이터베이스 서버는 내부에 세션이라는 것을 만든다.
- 해당 커넥션을 통한 모든 요청은 이 세션을 통해서 실행하게 된다.
- 클라이언트를 통해 SQL을 전달하면 현재 커넥션에 연결된 세션이 SQL을 실행한다.
  - 세션이 트랜잭션을 시작하고 커밋/롤백을 통해 트랜잭션을 종료한다.
  - 이후에 새로운 트랜잭션을 다시 시작할 수 있다.

<br >

## Transaction 기본 동작 구현

---

트랜잭션을 사용하려면 먼저 자동/수동 commit 을 이해해야 한다. 자동커밋으로 설정이 되어 있다면 각각의 쿼리 실행 직후에 자동으로 커밋을 호출한다.
때문에 트랜잭션 기능을 제대로 수행하기 위해서는 수동커밋으로 설정해줘야 한다.

> 수동 커밋 모드로 설정하는 것을 "트랜잭션 시작" 이라고 표현할 수 있다.

### session1 과 session 2의 간의 계좌이체

![Currying Image](/assets/img/post_img/coding/spring/transaction_basic_2.png){: width="600" align="center"}

A 가 B 에서 2,000원을 이체하는 상황에서 B로 송금하는 과정중에 SQL 문제가 발생한다면 A의 돈 2,000을 줄이는 것에는 성공하겠지만, B의 돈을 2,000원 증가시키는 것에는 실패할 것이다.

```sql
set autocommit false;
update member set money=10000 - 2000 where member_id = 'A'; //성공
update member set money=10000 + 2000 where member_iddd = 'B'; //쿼리 예외 발생
```

![Currying Image](/assets/img/post_img/coding/spring/transaction_basic_3.png){: width="600" align="center"}

이럴 때는 rollback 을 호출하여 트랜잭션을 시작하기 전 단계로 데이터를 복구해야 한다.
rollback 을 호출하여 계좌이체를 실행하기 전 상태로 돌아왔다. A의 돈, B의 돈 모두 10,000원 으로 유지되는 것을 확인할 수 있다.

```sql
rollback;
```

<br >

## 실제 애플리케이션에서 DB 트랜잭션 구현

---

애플리케이션에서 트랜잭션을 어떤 계층에 걸어야 할까?
즉, 트랜잭션을 어디에서 시작하고, 어디에서 커밋해야할까?

![Currying Image](/assets/img/post_img/coding/spring/transaction_basic_4.png){: width="600" align="center"}

- 트랜잭션은 비즈니스 로직이 있는 서비스 계층에서 시작해야 한다. 비즈니스 로직이 잘못되면 해당 비즈니스 로직 으로 인해 문제가 되는 부분을 함께 롤백해야 하기 때문이다.
- 그런데 트랜잭션을 시작하려면 커넥션이 필요하다. 결국 서비스 계층에서 커넥션을 만들고, 트랜잭션 커밋 이후에 커넥션을 종료해야 한다.
- 애플리케이션에서 DB 트랜잭션을 사용하려면 `트랜잭션을 사용하는 동안 같은 커넥션을 유지`해야한다. 그래야 같 은 세션을 사용할 수 있다.

![Currying Image](/assets/img/post_img/coding/spring/transaction_basic_5.png){: width="600" align="center"}

- 애플리케이션에서 같은 커넥션을 유지하려면 어떻게 해야할까? 가장 단순한 방법은 커넥션을 파라미터로 전달해서 같 은 커넥션이 사용되도록 유지하는 것이다.

```java
import hello.jdbc.domain.Member;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.support.JdbcUtils;

import javax.sql.DataSource;
import java.sql.*;
import java.util.NoSuchElementException;

/**
 * JDBC - ConnectionParam
 */
@Slf4j
public class MemberRepositoryV2 {
    private final DataSource dataSource;

    public MemberRepositoryV2(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values(?, ?)";

        Connection con = null;
        PreparedStatement pstmt = null;
        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, member.getMemberId());
            pstmt.setInt(2, member.getMoney());
            pstmt.executeUpdate();
            return member;
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    public Member findById(String memberId) throws SQLException {
        String sql = "select * from member where member_id = ?";
        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            rs = pstmt.executeQuery();
            if (rs.next()) {
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));
                return member;
            } else {
                throw new NoSuchElementException("member not found memberId=" +
                        memberId);
            }

        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, rs);
        }
    }

    public Member findById(Connection con, String memberId) throws SQLException {
        String sql = "select * from member where member_id = ?";
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            rs = pstmt.executeQuery();
            if (rs.next()) {
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));
                return member;
            } else {
                throw new NoSuchElementException("member not found memberId=" +
                        memberId);
            }
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            // connection 은 여기서 닫지 않는다.
            JdbcUtils.closeResultSet(rs);
            JdbcUtils.closeStatement(pstmt);
        }
    }

    public void update(String memberId, int money) throws SQLException {
        String sql = "update member set money=? where member_id=?";
        Connection con = null;
        PreparedStatement pstmt = null;
        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    public void update(Connection con, String memberId, int money) throws
            SQLException {
        String sql = "update member set money=? where member_id=?";
        PreparedStatement pstmt = null;
        try {
            pstmt = con.prepareStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            // connection 은 여기서 닫지 않는다.
            JdbcUtils.closeStatement(pstmt);
        }
    }

    public void delete(String memberId) throws SQLException {
        String sql = "delete from member where member_id=?";
        Connection con = null;
        PreparedStatement pstmt = null;
        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        JdbcUtils.closeConnection(con);
    }

    private Connection getConnection() throws SQLException {
        Connection con = dataSource.getConnection();
        log.info("get connection={} class={}", con, con.getClass());
        return con;
    }
}

```

- 커넥션 유지가 필요한 두 메서드는 파라미터로 넘어온 커넥션을 사용해야 한다. 따라서 `con = getConnection()` 코드가 있으면 안된다.
- 커넥션 유지가 필요한 두 메서드는 리포지토리에서 커넥션을 닫으면 안된다. 커넥션을 전달 받은 리포지토리 뿐만 아니라 이후에도 커넥션을 계속 이어서 사용하기 때문이다.
- 이후 서비스 로직이 끝날 때 트랜잭션을 종료하고 닫아야 한다.

```java
import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV2;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

/**
 * # 트랜잭션 - 파라미터 연동, 풀을 고려한 종료
 * - 반복되는 (커넥션 생성 / 트랜잭션 시작 / 예외처리 / 롤백 / 커넨션 풀 반납)은 충분히 문제가 될 요소이기 때문에
 *   다음 MemberService 버전에서 해결방법이 제시 된다.
 */
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV2 {
    private final DataSource dataSource;
    private final MemberRepositoryV2 memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Connection connection = dataSource.getConnection();
        try {
            /**
             * <트랜잭션 시작>
             * - 트랜잭션을 시작하려면 자동 커밋 모드를 꺼야한다.
             * - 커넥션을 통해 세션에 set autocommit false 가 전달되어 수동 커밋모드로 동작한다.
             */
            connection.setAutoCommit(false);
            Member fromMember = memberRepository.findById(connection, fromId);
            Member toMember = memberRepository.findById(connection, toId);

            int fromMemberMoney = fromMember.getMoney();
            int toMemberMoney = toMember.getMoney();

            memberRepository.update(connection, fromId, fromMemberMoney - money);
            validation(toMember);
            memberRepository.update(connection, toId, toMemberMoney + money);

            connection.commit(); // 성공시 커밋
        } catch (Exception e) {
            connection.rollback(); // 실패시 롤백
            throw new IllegalStateException(e);
        } finally {
            release(connection);
        }
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체중 예외 발생");
        }
    }

    private void release(Connection connection) {
        if (connection != null) {
            try {
                /**
                 * 해당 커넥션은 종료되는 것이 아니라 풀에 반납되기 때문에 autocommit 을 true 로 전환 하지 않으면
                 * 다른 비즈니스 로직에서도 트랜잭션이 시작하게 된다.
                 */
                connection.setAutoCommit(true);
                connection.close();
            } catch (Exception e) {
                log.info("error", e);
            }
        }
    }
}
```

- `Connection con = dataSource.getConnection();`
  - 트랜잭션을 시작하려면 커넥션이 필요하다.
- `con.setAutoCommit(false); // 트랜잭션 시작`
  - 트랜잭션을 시작하려면 자동 커밋 모드를 꺼야한다. 이렇게 하면 커넥션을 통해 세션에 `set autocommit false` 가 전달되고, 이후부터는 수동 커밋 모드로 동작한다. 이렇게 자동 커밋 모드를 수동 커밋 모드로 변경하는 것을 트랜잭션을 시작한다고 보통 표현한다.
- `bizLogic(con, fromId, toId, money);`
  - 트랜잭션이 시작된 커넥션을 전달하면서 비즈니스 로직을 수행한다.
  - 이렇게 분리한 이유는 트랜잭션을 관리하는 로직과 실제 비즈니스 로직을 구분하기 위함이다.
  - `memberRepository.update(con..)` : 비즈니스 로직을 보면 리포지토리를 호출할 때 커넥션을 전달 하는 것을 확인할 수 있다.
- `con.commit(); // 성공시 커밋`
  - 비즈니스 로직이 정상 수행되면 트랜잭션을 커밋한다.
- `con.rollback(); // 실패시 롤백`
  - `catch(Ex){..}` 를 사용해서 비즈니스 로직 수행 도중에 예외가 발생하면 트랜잭션을 롤백한다.
- `release(con);`
  - `finally {..}` 를 사용해서 커넥션을 모두 사용하고 나면 안전하게 종료한다. 그런데 커넥션 풀을 사용하면 `con.close()` 를 호출 했을 때 커넥션이 종료되는 것이 아니라 풀에 반납된다.
  - 현재 수동 커밋 모드 로 동작하기 때문에 풀에 돌려주기 전에 기본 값인 자동 커밋 모드로 변경하는 것이 안전하다.

## 서비스 계층의 Transaction 기능 구현의 문제점

서비스 계층에 트랜잭션을 기능을 구현한 덕분에 계좌이체가 실패하는 경우 롤백을 수행해서 모든 데이터를 정상적으로 초기화 할 수 있게 되었다.

하지만 이런식의 애플리케이션 개발은 초반에 말했듯이 서비스 계층이 매우 지저분해지고, 복잡한 코드를 요구해 유지보수성이 떨어지게 된다. 추가적으로 커넥션을 전달하고 자원을 회수하는 과정의 코드 또한 복잡하고 반복되는 문제가 있다.

이러한 문제는 Spring 에서는 트랜잭션의 추상화와 동기화를 통해 해결하고 있으므로 이를 통해 트랜잭션 기능 구현을 개선할 수 있다.
