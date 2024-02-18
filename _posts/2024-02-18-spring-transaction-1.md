---
title: "[Spring] Transaction Propagation - (1)"
date: 2024-02-18 01:20:30 +0900
categories: [Coding, Spring]
tags: [Coding, Spring]
---

## Transaction 기본

---

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.interceptor.DefaultTransactionAttribute;

import javax.sql.DataSource;

@Slf4j
@SpringBootTest
public class BasicTxTest {
    @Autowired
    PlatformTransactionManager platformTransactionManager;

    @TestConfiguration
    static class Config {
        /**
         * @TestConfiguration: 해당 테스트에서 필요한 스프링 설정을 추가로 할 수 있다.
         * DataSourceTransactionManager 를 스프링 빈으로 등록했다. 이후 트랜잭션 매니저인 PlatformTransactionManager 를 주입 받으면
         * 방금 등록한 DataSourceTransactionManager 가 주입된다.
         */
        @Bean
        public PlatformTransactionManager platformTransactionManager(DataSource dataSource) {
            return new DataSourceTransactionManager(dataSource);
        }
    }

    @Test
    void commit() {
        log.info("트랜잭션 시작");
        TransactionStatus transactionStatus = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());
        log.info("트랜잭션 커밋 시작");
        platformTransactionManager.commit(transactionStatus);
        log.info("트랜잭션 커밋 완료");
    }

    @Test
    void rollback() {
        log.info("트랜잭션 시작");
        TransactionStatus transactionStatus = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());
        log.info("트랜잭션 롤백 시작");
        platformTransactionManager.rollback(transactionStatus);
        log.info("트랜잭션 롤백 완료");
    }

    @Test
    void double_commit() {
        log.info("트랜잭션1 시작");
        TransactionStatus transactionStatus1 = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());
        log.info("트랜잭션1 커밋");
        platformTransactionManager.commit(transactionStatus1);
        log.info("트랜잭션2 시작");
        TransactionStatus transactionStatus2 = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());
        log.info("트랜잭션2 커밋");
        platformTransactionManager.commit(transactionStatus2);
    }

    @Test
    void double_commit_rollback() {
        /**
         * 두 서로 다른 트랜잭션은 서로 다른 connection 을 사용하기 때문에 (커밋 / 롤백)이 구분 된다.
         */
        log.info("트랜잭션1 시작");
        TransactionStatus tx1 = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());
        log.info("트랜잭션1 커밋");
        platformTransactionManager.commit(tx1);

        log.info("트랜잭션2 시작");
        TransactionStatus tx2 = platformTransactionManager.getTransaction(new DefaultTransactionAttribute());
        log.info("트랜잭션2 롤백");
        platformTransactionManager.rollback(tx2);
    }
}
```

1. commit(), rollback() : 서로 별개의 메서드를 호출함으로써 아무런 영향없이 커밋/롤백이 이 실행 된다.
2. double_commit()
   - 트랜잭션1: `Acquired Connection [HikariProxyConnection@1000000 wrapping conn0]`
   - 트랜잭션2: `Acquired Connection [HikariProxyConnection@2000000 wrapping conn0]`
   - 로그를 보면 트랜잭션1과 트랜잭션2가 같은 conn0 커넥션을 사용중인데 이것은 중간에 커넥션 풀 때문에 그런 것이다.  
     트랜잭션1은 conn0 커넥션을 모두 사용하고 커넥션 풀에 반납까지 완료하고, 이후에 트랜잭션2가 conn0 를 커 넥션 풀에서 획득한 것이다. 따라서 둘은 완전히 다른 커넥션으로 인지하는 것이 맞다.
   - 히카리 커넥션 풀에서 커넥션을 획득하면 실제 커넥션을 그대로 반환하는 것이 아니라 내부 관리를 위해 히카리 프록시 커넥션이라는 객체를 생성해서 반환한다. 물론 내부에는 실제 커넥션이 포함되어 있기 때문에 이 객체의 주소를 확인하면 커넥 션 풀에서 획득한 커넥션을 구분 할 수 있다.
   - 트랜잭션이 각각 수행되면서 사용되는 DB 커넥션도 각각 다른다.

![Currying Image](/assets/img/post_img/coding/spring/transaction_propagation_1_1.png){: width="500" .normal}

3. double_commit_rollback()
   - 마찬가지로 전체 트랜잭션을 묶지 않고 각각 관리했기 때문에, 트랜잭션1에서 저장한 데이터는 커밋되고, 트랜잭션2에서 저 장한 데이터는 롤백된다.

![Currying Image](/assets/img/post_img/coding/spring/transaction_propagation_1_2.png){: width="500" .normal}
