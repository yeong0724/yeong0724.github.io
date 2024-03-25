---
title: "@Transaction - AOP 를 이용해서 트랜잭션 처리하기"
date: 2024-02-15 23:30:30 +0900
categories: [Beckend, Spring]
tags: [Java, Spring]
---

## INTRO

---

트랜잭션을 처리하기 위해 트랜잭션 추상화, 트랜잭션 동기화, 트랜잭션 템플릿 등을 도입했다. 덕분에 트랜잭션을 처리하는데 있어서 반복되는 코드를 해결할 수 있었지만, 아직 Application 구조상의 서비스 계층 순수함은 충족시키지 못했다.
이를 해결하기 위해 스프링 AOP를 통한 프록시를 활용해 보겠다.

---

<br >

## 프록시를 통한 문제 해결

![transaction_aop_1](/assets/img/post_img/coding/spring/transaction/transaction_aop_1.png){: width="500" align="center"}

<center>[그림. 1]프록시 도입전</center>
- 프록시를 도입하기 전에는 그림과 같이 서비스 로직에서 트랜잭션을 직접 시작한다.

![transaction_aop_2](/assets/img/post_img/coding/spring/transaction/transaction_aop_2.png){: width="500" align="center"}

<center>[그림. 2]프록시 도입후</center>
- 프록시를 이용한다면 이처럼 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리 할 수 있다.

```java
public class TransactionProxy {
     private MemberService target;
	public void logic() {
		// 트랜잭션 시작
		TransactionStatus status = transactionManager.getTransaction(..);
		try {
			// 실제 대상 호출
			target.logic(); transactionManager.commit(status); // 성공시 커밋
		} catch (Exception e) {
			transactionManager.rollback(status); // 실패시 롤백
			throw new IllegalStateException(e);
		}
	}
}
```

<center>[코드. 1]트랜잭션 프록시 코드 예</center>

```java
public class Service {
	public void logic() {
		//트랜잭션 관련 코드 제거, 순수 비즈니스 로직만 남음
		bizLogic(fromId, toId, money);
	}
}
```

<center>[코드. 2]트랜잭션 프록시 적용 후 서비스 코드 예</center>

- 프록시를 도입하고 나면 트랜잭션 프록시가 처리 로직을 모두 가져가 트랜잭션을 시작한 후에 실제 서비스를 대신 호출한다.
- 덕분에 서비스 계층에는 순수한 비즈니스 로직만 남길 수 있게 됐다.

<br >

```java
import hello.jdbc.domain.Member;
import hello.jdbc.repository.MemberRepositoryV3;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.transaction.annotation.Transactional;
import java.sql.SQLException;

/**
 * 트랜잭션 - @Transactional AOP */
@Slf4j
@RequiredArgsConstructor
public class MemberServiceV3_3 {
    private final MemberRepositoryV3 memberRepository;

    /**
     * @Transactional 어노테이션을 통해 순수 비즈니스 로직만 남기고 트랜잭션 코드는 모두 생략 가능 해진다.
     * - 클래스에 붙이면 외부에서 호출 가능한 public 메서드가 AOP 적용 대상이 된다
     */
    @Transactional
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Member fromMember = memberRepository.findById(fromId);
        Member toMember = memberRepository.findById(toId);

        int fromMemberMoney = fromMember.getMoney() - money;
        int toMemberMoney = toMember.getMoney() + money;

        memberRepository.update(fromId, fromMemberMoney);
        validation(toMember);
        memberRepository.update(toId, toMemberMoney);
    }

    private void validation(Member toMember) {
        if (toMember.getMemberId().equals("ex")) {
            throw new IllegalStateException("이체중 예외 발생");
        }
    }
}
```

<center>[코드. 3]트랜잭션 프록시 적용 후 서비스 코드</center>

- 순수한 비즈니스 로직만 남기고, 트랜잭션 관련 코드는 모두 제거했다.
- 스프링이 제공하는 트랜잭션 AOP를 적용하기 위해 `@Transactional` 애노테이션을 추가했다.
- `@Transactional` 애노테이션은 메서드에 붙여도 되고, 클래스에 붙여도 되지만, 클래스에 붙이면 외부에서 호출 가능한 `public` 메서드가 AOP 적용 대상이 된다.

<br>

## 트랜잭션 AOP 의 적용 흐름

![transaction_aop_3](/assets/img/post_img/coding/spring/transaction/transaction_aop_3.png){: width="700" align="center"}

<center>[그림. 3]트랜잭션 AOP 적용 전체 흐름</center>
<br>

## 선언적 트랜잭션 관리 (Declarative Transaction Management)

- `@Transactional` 애노테이션 하나만 선언해서 매우 편리하게 트랜잭션을 적용하는 것을 선언적 트랜잭 션 관리라 한다.
- 선언적 트랜잭션 관리가 프로그래밍 방식에 비해서 훨씬 간편하고 실용적이기 때문에 실무에서는 대부분 선언적 트랜잭션 관리를 사용한다.
- 스프링이 제공하는 선언적 트랜잭션 관리 덕분에 드디어 트랜잭션 관련 코드를 순수한 비즈니스 로직에서 제거할 수 있게 됐다.
- 트랜잭션이 필요한 곳에 `@Transactional` 애노테이션 하나만 추가하면, 스프링 트랜잭션 AOP가 자동으로 처리해준다.
