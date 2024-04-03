---
title: "Hexagonal Architecture: 유연하고 확장 가능한 디자인"
date: 2024-04-03 21:44:30 +0900
categories: [Project, Architecture]
tags: [MSA, Clean Architecture]
---

## Hexagonal Architecture 란?

---

Hexagonal Architecture 는 클린 아키텍처를 구현하는 가장 대표적인 모델로, Alistair Cockburn 이 처음로 제안한 개념이다.

이 아키텍처는 소프트웨어 시스템을 여러개의 계층(Layer)으로 분리하고, 외부 요소와 내부 요소 사이으이 결합도를 낮추는 것을 목표로 한다. 이를 통해 시스템을 더 유연하고 테스트 가능하게 만들며, 변경에 대한 영향을 최소화 하는 것을 목표로 한다.

<br />

## Hexagonal Architecture 의 구성

---

헥사고날 아키텍처는 내부(도메인 - in) 과 외부(인프라 - out) 로 구분된다.

내부영역은 순수한 비즈니스 로직은 표현하며 캡슐화된 영역이고, 기능적 요구사항에 따라 먼저 설계 된다. 외부 영역은 내부 영역에서 기술을 분리하여 구성한 영역으로 내부 영역 설계 이후 설계 된다.

![hexagonal_1.png](/assets/img/post_img/project/architecture/hexagonal_1.png){: width="500" align="center"}

<br />

## Hexagonal Architecture 프로젝트 구조

---

- Adapter
  - in
    - web
      - controller (Controller 구현체)
      - request
    - kafka
      - result-consumer.java
  - out
    - persistence
      - entity
      - repository
      - adapter (Port 구현체)
    - kafka
      - task-producer.java
- Application
  - port
    - in
      - UseCase
      - Command
    - out
      - Port
        - kafka-message-port-interface
        - service-to-repository-port-interface
  - service
    - Service.java (UseCase 구현체)
- Domain
  - domain.java

<br />

## Adapter

---

헥사고날 아키텍처(Hexagonal Architecture)에서 어댑터(Adapter)는 외부와 내부 사이의 통신을 관리하는 핵심 요소중 하나이다. 어댑터는 외부 요청을 내부에 전달하고, 내부의 응답을 외부로 반환하는 역할을 수행하여 시스템의 외부와 내부 간의 결합도를 낮추고, 시스템을 유연하게 만든다.

어댑터는 헥사고날 아키텍처에서 외부와 내부의 경계를 명확히 정의하고, 각각의 책임을 분리하여 시스템의 유연성과 테스트 용이성을 향상시키는 데 중요한 역할을 합니다.

### **1. Inbound Adapter**

외부 요청을 받아들이고 내부 시스템에서 처리할 수 있는 형식으로 변환한다. 주로 웹 요청을 처리하여 내부 시스템이 이해할 수 있는 형태로 변환하는 역할을 한다.

다시말해, Web 에서 REST api 호출을 할 때 Controller 는 Request 객체를 받아 Command (Usecase 와 직접 연결된 명령어) 객체로 변환한다.

```java
@RestController
@RequiredArgsConstructor
public class RegisterMembershipController {
    private final RegisterMembershipUseCase registerMembershipUseCase;

    @PostMapping(path = "/membership/register")
    Membership registerMembership(@RequestBody RegisterMembershipRequest registerMembershipRequest) {
        RegisterMembershipCommand registerMembershipCommand = RegisterMembershipCommand.builder()
                .name(registerMembershipRequest.getName())
                .address(registerMembershipRequest.getAddress())
                .email(registerMembershipRequest.getEmail())
                .isValid(true)
                .isCorp(registerMembershipRequest.isCorp())
                .build();

        // Usecase
        return registerMembershipUseCase.registerMembership(registerMembershipCommand);
    }
}
```

### 2. **Outbound Adapter**

내부 시스템에서 생성된 응답을 외부 시스템이 이해할 수 있는 형식으로 변환하여 반환한다 주로 내부 시스템에서 생성된 데이터를 외부 시스템에 전송하기 위해 사용된다. (Outbound Port 를 구현한 구현체)

```java
@PersistenceAdapter
@RequiredArgsConstructor
public class MembershipPersistenceAdapter implements RegisterMembershipPort, FindMembershipPort, ModifyMembershipPort {
    private final SpringDataMembershipRepository springDataMembershipRepository;

    @Override
    public MembershipJpaEntity createMembership(Membership.Name name, Membership.Email email, Membership.Address address, Membership.IsValid isValid, Membership.IsCorp isCorp) {
        return springDataMembershipRepository.save(
                new MembershipJpaEntity(
                        name.getName(),
                        email.getEmail(),
                        address.getAddress(),
                        isValid.isValid(),
                        isCorp.isCorp()
                )
        );
    }
}
```

- 외부로 Message 를 Produce 하기 위한 kafka Out-Port inferface 를 구현하는 클래스

```java
@Component
public class TaskProducer implements SendRechargingMoneyTaskPort {
    private final KafkaProducer<String, String> producer;

    private final String topic;

    public TaskProducer(
            @Value("${kafka.clusters.bootstrapservers}") String bootstrapServers,
            @Value("${task.topic}") String topic
    ) {
        Properties props = new Properties();
        props.put("bootstrap.servers", bootstrapServers);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        this.producer = new KafkaProducer<>(props);
        this.topic = topic;
    }

    @Override
    public void sendRechargingMoneyTaskPort(RechargingMoneyTask rechargingMoneyTask) {
        this.sendMessage(rechargingMoneyTask.getTaskId(), rechargingMoneyTask);
    }

    public void sendMessage(String key, RechargingMoneyTask rechargingMoneyTask) {
        ObjectMapper mapper = new ObjectMapper();
        String jsonStringToProduce;
        // jsonString
        try {
            jsonStringToProduce = mapper.writeValueAsString(rechargingMoneyTask);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }

        ProducerRecord<String, String> record = new ProducerRecord<>(topic, key, jsonStringToProduce);
        producer.send(record, (metadata, exception) -> {
            if (exception == null) {
                // System.out.println("Message sent successfully. Offset: " + metadata.offset());
            } else {
                exception.printStackTrace();
                // System.err.println("Failed to send message: " + exception.getMessage());
            }
        });
    }
}
```

<br />

## Application - Port

---

헥사고날 아키텍처(Hexagonal Architecture)에서 포트(Port)는 애플리케이션 코어의 경계를 정의하며, Adapter 를 통해 애플리케이션 코어에 접근하는 인터페이스다.

### 1. Incoming Port

내부 영역 사용을 위해 노출된 API로 Command 와 UseCase 가 있다.

- **Command**: 유스케이스 인터페이스 동작을 정의

```java
@Builder
@Data
@EqualsAndHashCode(callSuper = false)
public class RegisterMembershipCommand extends SelfValidating<RegisterMembershipCommand> {
    @NotNull
    @NotBlank
    @Pattern(regexp = "[a-zA-Z0-9-.]*", message = INVALID_NICKNAME_MESSAGE)
    @Length(min = 2, max = 23, message = INVALID_NICKNAME_MESSAGE)
    private final String name;

    @NotNull
    @NotBlank
    private final String email;

    @NotNull
    @NotBlank
    private final String address;

    @AssertTrue
    private final boolean isValid;

    private final boolean isCorp;


    public RegisterMembershipCommand(String name, String email, String address, boolean isValid, boolean isCorp) {
        this.name = name;
        this.email = email;
        this.address = address;
        this.isValid = isValid;
        this.isCorp = isCorp;

        this.validateSelf();
    }
}
```

- **UseCase**: 외부(Controller)에서 내부(Service)로 비즈니스간에 통신 하기위한 인터페이스

```java
public interface RegisterMembershipUseCase {
    Membership registerMembership(RegisterMembershipCommand registerMembershipCommand);
}
```

### 2. Outgoing Port

애플리케이션 코어가 외부에 서비스를 제공하기 위한 경로를 정의한다. 예를들면, Service에서 DB와 통신하기 위한 포트 인터페이스, Kafka message produce 를 위한 포트 인터페이스 등이 있다.

- 일반적인 DB 통신을 위한 포트 인터페이스

```java
public interface RegisterMembershipPort {
    MembershipJpaEntity createMembership(
            Membership.Name name,
            Membership.Email email,
            Membership.Address address,
            Membership.IsValid isValid,
            Membership.IsCorp isCorp
    );
}
```

- Message System 을 위한 포트 인터페이스

```java
public interface SendRechargingMoneyTaskPort {
    void sendRechargingMoneyTaskPort(RechargingMoneyTask rechargingMoneyTask);
}
```

<br />

## Application - Servcie

---

Usecase interface 의 구현체로서 애플리케이션 코어의 비즈니스 로직을 구체적으로 구현한다. 프로세스적 유효성을 검증하고, DB 통신을 요청하는 등 전체적인 과정을 처리한다.

```java
@UseCase
@RequiredArgsConstructor
@Transactional
public class ResisterMembershipService implements RegisterMembershipUseCase {
    private final RegisterMembershipPort registerMembershipPort;
    private final MembershipMapper membershipMapper;

    @Override
    public Membership registerMembership(RegisterMembershipCommand registerMembershipCommand) {
        /**
         * - 전달받은 Command 객체를 통해 연결된 DB와 통신
         * - business logic 입장에서는 DB 또한 외부 시스템 -> port/adapter 를 통해야만 밖으로 통신 가능
         */
        MembershipJpaEntity membershipJpaEntity = registerMembershipPort.createMembership(
                new Membership.Name(registerMembershipCommand.getName()),
                new Membership.Email(registerMembershipCommand.getEmail()),
                new Membership.Address(registerMembershipCommand.getAddress()),
                new Membership.IsValid(registerMembershipCommand.isValid()),
                new Membership.IsCorp(registerMembershipCommand.isCorp())
        );

        // Entity -> Domain
        return membershipMapper.mapToDomainEntity(membershipJpaEntity);
    }
}
```

<br />

## Adapter 와 Port 간에 결합도를 어떻게 낮추었나?

---

코드 상으로 봤을때 어댑터 패키지에 위치한 컨트롤러 객체가 비지니스 로직을 담당하는 Application Core (service)에 접근하기 위해서 포트 패키지에 위치한 포트 인터페이스(UseCase interface)를 통한다.

이 과정에서 컨트롤러는 포트가 사용하는 입력 객체로의 매핑 작업을 수행(Request —> Command) 하며, 이 객체는 생성 시점에서 생성자를 통해 입력의 유효성을 검사한다.

포트는 어디까지나 실질 도메인 로직이 아니라, 애플리케이션 코어에 접근하는 인터페이스 이다. 실제 구현체(Service)는 비지니스 로직을 수행하는 애플리케이션 코어 내부에 위치하고 있다.

즉, 어댑터는 특정 UseCase Interface 를 통해 애플리케이션 코어에 접근하게 되고, 해당 Ingoing Port 를 통해 실행하고자 하는 구체적인 구현체(Service)를 호출하게 된다.

어댑터의 컨트롤러는 애플리케이션 코어의 구체적인 사항을 알 필요없이 사용하는 포트만 알면 되고, 이로써 헥사고날 아키텍처는 외부 요구사항의 변경이 비즈니스 로직에 영향을 미치는 것을 방지한다.

그 결과, 각 계층(Adapter ←→ Application)은 자신의 역할에만 집중할 수 있게 되어 결합도는 낮아지고 응집도는 높아지게 된다. (해당 내용은 Service ←→ outgoing-Adapter 와의 관계에도 똑같이 일맥상통한다.)

<br />

## 마치며

---

헥사고날 아키텍처는 Inbound Adapter, Ingoing Port(Commnad, Usecase), Outgoing Port, Outbound Adapter 를 활용해 계층 간의 결합도를 낮추고, 각 계층의 책임에 집중할 수 있게 응집도를 높인다.

이러한 특징들 덕분에 헥사고날 아키텍처는 유연하고 확장 가능하며, 변경에 강한 애플리케이션을 구축하는데 매우 적합하다.
