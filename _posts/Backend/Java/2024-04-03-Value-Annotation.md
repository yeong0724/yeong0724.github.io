---
title: "@Value 어노테이션의 기능"
date: 2024-04-03 20:41:30 +0900
categories: [Beckend, Java]
tags: [Java, Annotation]
---

@Value 어노테이션은 Lombok 라이브러리에서 제공하는 어노테이션 중 하나로, 불변(Immutable) 한 값을 가지는 클래스를 간편하게 생성할 때 사용된다.

**`@Value`** 어노테이션을 사용하면 클래스의 필드를 final로 선언하고, 해당 필드들에 대한 getter 메서드를 자동으로 생성해준다. 또한 클래스에 대한 equals, hashCode, toString 메서드도 자동으로 생성된다.

**`@Value`** 어노테이션은 **`@Data`** 어노테이션과 유사하지만, **`@Value`** 어노테이션으로 생성된 클래스는 불변(Immutable)하므로 setter 메서드가 생성되지 않는다. 이는 클래스의 객체를 생성한 후에는 내부 상태를 변경할 수 없도록 하기 위해 사용된다.

```java
import lombok.Value;

@Value
public class Person {
    String name;
    int age;
}
```

`Person` 클래스에 `@Value` 어노테이션을 적용하면, 다음과 같은 코드가 자동으로 생성됩니다:

- `private final`로 선언된 필드인 `name` 과 `age`
- `name` 과 `age` 에 대한 getter 메서드
- `equals`, `hashCode`, `toString` 메서드

@Value 어노테이션을 통해 Person 클래스는 불변한 객체로 사용될 수 있으며, 객체의 상태를 변경할 수 없다. 따라서 불필요한 상태 변경을 방지하고 안정성을 높일 수 있다.
