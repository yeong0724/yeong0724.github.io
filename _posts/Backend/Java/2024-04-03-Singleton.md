---
title: "Singleton Pattern 알고쓰자"
date: 2024-04-03 21:12:30 +0900
categories: [Beckend, Java]
tags: [Java, Singleton]
---

## Singleton Pattern 이란 무엇인가?

---

싱글톤 패턴은 소프트웨어 디자인에서 사용되는 패턴 중 하나로, 이 패턴은 특정 클래스가 인스턴스를 오직 하나만 생성하도록 보장하는 것을 목적으로 한다. 이는 보통 전역 변수를 사용하지 않고 객체를 하나만 생성하여 이를 공유하는 데 사용된다.

이를 구현하기 위해, 먼저 클래스 내에 해당 클래스의 인스턴스를 저장할 정적 변수를 만들고, 이 변수는 private로 선언하여 외부에서 직접 접근할 수 없도록 한다. 그리고 생성자를 private으로 만들어 외부에서 해당 클래스의 인스턴스를 직접 생성할 수 없도록 한다.

대신에, 클래스 내부에서 정적 메서드를 제공하여, 클래스의 유일한 인스턴스를 반환하도록 구현한다. 이 메서드는 클래스의 정적 변수를 확인하고, 해당 변수가 null인 경우에만 인스턴스를 생성하고 반환한다. 만약 이미 인스턴스가 존재하는 경우에는 이를 반환하게 된다.

```java
public class Singleton {
    // 정적 변수로 유일한 인스턴스를 저장
    private static Singleton instance;

    // 생성자를 private으로 선언하여 외부에서 직접 인스턴스화를 방지
    private Singleton() {
        System.out.println("Singleton instance created."); // 생성 시 로그 출력
    }

    // 유일한 인스턴스를 반환하는 정적 메서드를 구현
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

이런 방식으로 싱글톤 패턴을 사용하면 애플리케이션 전반에 걸쳐 하나의 인스턴스만을 사용할 수 있게 된다. 이는 메모리와 자원을 효율적으로 관리할 수 있게 해주며, 전역적으로 접근 가능한 인스턴스를 제공하여 편리성을 높일 수 있다.

<br />

## Multi Thread 환경에서의 동기화 문제

---

멀티스레드 환경에서의 동기화 문제는 주로 여러 스레드가 동시에 공유된 자원에 접근할 때 발생한다. 싱글톤 패턴에서는 getInstance() 메서드가 인스턴스를 반환할 때 동시에 여러 스레드가 이 메서드에 접근 할 수 있다.

이 경우, 동시에 여러 스레드가 if 조건을 통과하여 인스턴스를 생성하는 경우가 발생할 수 있게 되는데, 이러한 상황에서 여러 개의 인스턴스가 생성될 수 있으며, 이는 싱글톤 패턴이 의도한대로 작동하기 않게 된다.

그렇다면, 멀티스레드 환경에서 동기화 문제를 해결하기 위한 방법은 어떤게 있을까?

### 1. **동기화된 getInstance() 메서드**

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

getInstance() 메서드에 synchronized 키워드를 추가하여 동기화를 수행한다. syncronized 키워드를 사용하면 여러 스레드가 동시에 해당 메서드에 접근하는 것을 막을 수 있다. 하지만 이 방법은 성능에 영향을 미칠 수 있기 때문에 성능이 중요한 프로젝트라면 사용하는 것에 대해 재고할 필요가 있다.

### 2. **더블 체크 락 (Double-Checked Locking)**

```java
public class Singleton {
    private volatile static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

더블 체크 락(Double-Checked Locking)을 사용하여 동기화를 최소화한다. 이 패턴은 인스턴스가 이미 생성되어 있는 경우에는 동기화 블록을 실행하지 않고 바로 인스턴스를 반환한다. 이를 위해 volatile 키워드를 사용하여 instance 변수의 가시성을 보장한다.

### 3. **정적 초기화**

```java
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

위의 코드에서는 클래스 로딩 시점에 정적 초기화를 통해 인스턴스를 미리 생성한다. 이 방법은 클래스 로딩 시점에 초기화가 이루어지기 때문에 동기화 문제가 발생하지는 않는다.

다만, 초기화 시점이 애플리케이션 시작과 동일하므로 애플리케이션이 시작될 때부터 인스턴스가 생성된다.

<br />

## 싱글톤 패턴의 단점은?

---

싱글톤 패턴은 많은 경우에 유용하지만, 몇가지 구조적인 문제가 있을 수 있다. 어떤 문제점들이 있는지 알아보자.

### 1. **전역 상태**

싱글톤은 전역 상태를 갖게 된다. 이는 다른 클래스들 사이에서 의존성을 증가시킬 수 있고, 코드의 복잡성을 증가시킬 수 있다. 특히 다중 스레드 환경에서 공유 상태에 대한 동기화 문제가 발생할 수 있다.

### 2. 테스트 문제

싱글톤 패턴을 사용하면 의존성을 주입하거나 Mock Objects(모의 객체)를 사용하는 등의 테스트가 어려워질 수 있다. 싱글톤 클래스는 하나의 인스턴스만을 갖기 때문에 이를 모의(mock)로 대체하기가 어려울 수 있다.

### 3. **객체 생성 지연 문제**

일반적으로 싱글톤은 처음 사용될 때까지 인스턴스를 생성하지 않는다. 이로 인해 처음 인스턴스를 사용하는 시점에 초기화가 지연될 수 있고, 이 지연은 애플리케이션의 성능에 영향을 줄 수 있다.

### 4. **직렬화와 역직렬화 문제**

싱글톤 클래스가 직렬화되고 역직렬화되는 경우, 역직렬화 과정에서 새로운 인스턴스가 생성될 수 있다. 이는 원래의 싱글톤 보장을 깨뜨릴 수 있다. 이를 해결하기 위해서는 Serializable 인터페이스를 구현하고, readResolve() 메서드를 사용하여 싱글톤을 보장 해야한다.
