---
title: "안전한 Domain Class 설계"
date: 2024-04-03 20:35:30 +0900
categories: [Beckend, Java]
tags: [Java, Architecture, Domain]
---

Clean Architecture 관점에서 Domain 객체는 외부로부터 오염이 되면 안되는 클래스이다.

왜냐하면 Domain 클래스는 크리티컬한 정보를 담고 있고, 각 Service 에 있어서 핵심이기 때문에 안전하게 관리되어야 한다.

도메인 모델 성격의 클래스를 관리함에 있어서 각각의 멤버변수들을 private final 하게 정의하고, 각각의 value 들은 정의된 static 클래스에 의해서만 접근하고, 생성자의 접근 수준이 Private 으로 설정되어 외부에서 직접 생성자에 접근할 없게 되어 안전하게 사용되어 질 수 있도록 강제된다.

```java
import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Value;

@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Membership {

    @Getter
    private final String membershipId;

    @Getter
    private final String name;

    @Getter
    private final String email;

    @Getter
    private final String address;

    @Getter
    private final boolean isValid;

    @Getter
    private final boolean isCorp;

    public static Membership generateMembership(
            MembershipId membershipId,
            Name name,
            Email email,
            Address address,
            IsValid isValid,
            IsCorp isCorp
    ){
        return new Membership(
	        membershipId.membershipId,
	        name.name,
	        email.email,
	        address.address,
	        isValid.isValid,
	        isCorp.isCorp
	      );
    }


    @Value
    public static class MembershipId {
        String membershipId;

        public MembershipId(String membershipId) {
            this.membershipId = membershipId;
        }
    }

    @Value
    public static class Name {
        String name;

        public Name(String name) {
            this.name = name;
        }
    }

    @Value
    public static class Email {
        String email;

        public Email(String email) {
            this.email = email;
        }
    }

    @Value
    public static class Address {
        String address;

        public Address(String address) {
            this.address = address;
        }
    }

    @Value
    public static class IsValid {
        boolean isValid;

        public IsValid(boolean isValid) {
            this.isValid = isValid;
        }
    }

    @Value
    public static class IsCorp {
        boolean isCorp;

        public IsCorp(boolean isCorp) {
            this.isCorp = isCorp;
        }
    }
}
```

<br />

```java
@RequiredArgsConstructor
@Transactional
public class ResisterMembershipService implements RegisterMembershipUseCase {
    private final RegisterMembershipPort registerMembershipPort;
    private final MembershipMapper membershipMapper;

    @Override
    public Membership registerMembership(RegisterMembershipCommand registerMembershipCommand) {
		    Membership membership = Membership.generateMember(
						new Membership.Name(registerMembershipCommand.getName()),
						new Membership.Email(registerMembershipCommand.getEmail()),
						new Membership.Address(registerMembershipCommand.getAddress()),
						new Membership.IsValid(registerMembershipCommand.isValid()),
						new Membership.IsCorp(registerMembershipCommand.isCorp())
				)

        MembershipJpaEntity membershipJpaEntity = registerMembershipPort.createMembership(membership);

        // Entity -> Domain
        return membershipMapper.mapToDomainEntity(membershipJpaEntity);
}
```

<br />

## 정적 팩토리 메서드를 통한 도메인 객체 생성

---

정적 팩토리 메서드(Static Factory Method)는 클래스의 인스턴스를 생성하는 메서드로, 해당 클래스의 생성자 댕신에 사용될 수 있다. 이 메서드는 보통 클래스의 정적(static) 메서드로 정의되며, 객체 생성에 대한 세부적인 구현을 캡슐화하고 유연성을 제공한다.

상기 Membership 클래스에서도 generateMembership 을 정의하여, 해당 메서드를 통하지 않고서는 Membership 객체를 만들수 없도록 강제되어 지고 있다. 이로써 Membership 클래스는 충분히 보호받고 있다고 볼 수 있다.

```java
public static Membership generateMembership(
        MembershipId membershipId,
        Name name,
        Email email,
        Address address,
        IsValid isValid,
        IsCorp isCorp
){
    return new Membership(
      membershipId.membershipId,
      name.name,
      email.email,
      address.address,
      isValid.isValid,
      isCorp.isCorp
    );
}
```

<br />

## 각 멤버 변수의 값 지정

---

각각의 멤버 변수들을 value 로써 관리할 수 있는 클래스들을 선언이 되어 있어, Membership 에 직접적으로 접근하여 값을 지정할 수는 없고 Membership 내의 MembershipId 와 같은 클래스의 생성자를 통해 값을 지정할 수 있다.

```java
@Value
public static class MembershipId {
    String membershipId;

    public MembershipId(String membershipId) {
        this.membershipId = membershipId;
    }
}
```
