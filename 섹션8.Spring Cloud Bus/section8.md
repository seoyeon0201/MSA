🚀 Spring Cloud Bus를 사용하여 마이크로서비스가 변경사항을 효율적으로 가져갈 수 있도록 함

## 1. Spring Cloud Bus 개요

✔️ 이전

- Config 서버의 값을 변경했을 때 각각의 마이크로서비스가 변경된 값을 가져가는 방법: `서버 재기동`, `Actuator refresh`, `Spring cloud bus` 사용

- `서버재기동` : 너무 비효율적

- `Actuator refresh` : application이 많은 경우 모든 application을 refresh 시켜줘야 함

1. Spring Cloud Bus 사용

- 분산 시스템의 노드(`마이크로서비스`)끼리 연결시키는 메세지 브로커(`RabbitMQ`) 준비 > 상태 및 구성에 대한 변경 사항을 연결된 노드에 전달 (broadcast)

=> 노드 사이의 미들웨어가 존재해 노드 간의 직접 연결보다 안정적으로 메세지 전달 가능

⭐️ Spring Cloud Config Server + Spring Cloud Bus => AMQP(Advanced Message Queuing Protocol) 사용

- 각각의 마이크로서비스가 가지고 있어야 하는 정보가 존재한다면 Spring Cloud Config Server 옆의 Spring Cloud Bus가 마이크로서비스에 데이터를 직접 Push해 변경사항을 알려줌

- 각각 refresh를 하지 않아도 됨

2. Spring Cloud Bus 개념

✔️ AMQP

- AMQP (Advanced Message Queuing Protocol)은 메세지 지향 미들웨어를 위한 개방형 표준 응용 계층 프로토콜로, 메세지 지향, 큐잉, 라우팅(P2P, Publisher-Subcriber), 신뢰성, 보안

- Erlang, `RabbitMQ`에서 사용

⚡️ Windows에서 RabbitMQ 설치 시 Erlang 설치 후 진행

⚡️ RabbitMQ를 사용해 configuration의 변경 사항을 각 마이크로서비스에 알려주는 실습 진행 예정

✔️ Kafka 프로젝트

- Apache Software Foundation이 Scalar 언어로 개발한 오픈 소스 메세지 브로커 프로젝트

- 분산형 스트리밍 플랫폼

- 대용량의 데이터 처리 가능한 메세징 시스템

⚡️ 이후 Kafka를 이용해 데이터 동기화 진행 예정

3. RabbitmQ vs Kafka

✔️ RabbitMQ

- 메세지 브로커

- 초당 20개 이상의 메세지를 소비자에게 전달

- 메세지 전달 보장, 시스템 간의 메세지 전달

- 브로커, 소비자 중심

⚡️ 적은 데이터를 안전하게 보내는 경우 사용

✔️ Kafka

- 초당 100K 이상의 이벤트 처리

- Pub/Sub 존재해 Topic에 메세지 전달

- Ack를 기다리지 않고 전달

- 생산자 중심

⚡️ 많은 데이터를 빠르게 보내는 경우 사용

4. Actuator bus-refresh Endpoint

✔️ 준비사항

- 각각의 서비스는 메세지 브로커와 연결

- 변경사항을 서비스에 전달

✔️ Git(local, remote)또는 파일시스템(native)에 저장된 데이터를 config server에 가져옴

✔️ 외부의 별도 클라이언트가 POST 방식으로 `busrefresh`라는 actuator 호출

- 변경된 사항이 있다는 busrefresh가 호출되면 Spring Cloud Bus가 변경 사항을 감지하고 또다른 모든 클라이언트(마이크로서비스)에게 변경 사항 업데이트

- 이때 busrefresh는 config server, cloud bus, apigateway-server, microservice 어디에서 호출되든 상관없이 한 번만 호출되면 됨

## 2. RabbitMQ 설치

1. Erlang 설치

- 23.1 버전 추천

- 환경 변수 편집에 erl 추가

2. RabbitMQ 설치

- Windows의 installer에서 선택

- 3.8.11 버전 추천

- Window의 서비스에서 설치 확인

- 환경 변수 편집에 RabbitMQ 추가

- management plugin 설치 권장

`powershell`에 아래 명령어 입력
`rabbitmq-plugins enable rabbitmq_management`

- port번호 15672, username과 password는 기본으로 guest

참고
<https://velog.io/@ililil9482/Spring-Cloud-Bus>

## 3. AMQP 사용

1. Dependency 추가 : pom.xml

`Config Server`

- AMQP를 위한 Spring Cloud Bus

- Actuator

- 부트스트랩 정보가 자동으로 갱신되지 않는 경우도 있기에 Bootstrap dependency 추가

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

`user-servicee`, `apigateway-service`

- AMQP를 위한 Spring Cloud Bus

- Users Microservice와 Gateway Service는 Actuator, bootstrap 이전에 추가

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

2. application.yml 수정

- `Config Server`, `Users Microservice`, `Gateway Service`의 yml 파일 수정

- 각각의 microservice가 RabbitMQ의 노드로 연결됨을 의미

- 브라우저에서 연결할 때에는 15672번이지만, 시스템에서 연결할 때에는 포트번호 5672

```
spring:
    application:
        name: [application 이름]
    rabbitmq:
        host: 127.0.0.1
        port: 5672
        username: guest
        password: guest
    ...

management:
    endpoint:
        web:
            exposure:
                include: refresh, health, beans, httptrace, busrefresh
```

3. 테스트

`RabbitMQ Server` > Spring Cloud `Config Service` > `Eureka Discovery Service` > Spring Cloud `Gateway Service` > Users `Microservice`

## 4. Spring Cloud Bus 테스트

- config-service의 config 정보는 native로 파일로부터 가져옴

- user-service와 apigateway-service의 bootstrap.yml에서 name을 config-service

1. POSTMAN으로 테스트

- 회원등록 > 로그인 > 로그인으로부터 가져온 토큰 정보를 넣어 health_check

2. config 정보 변경

- application.yml 파일의 secret을 user*token_native_application_changed*#1로 변경 후 저장

- 웹브라우저에 config-service/default로 반영 확인

3. busrefresh

- user-service/actuator/busrefresh를 POST로 전달

=> RabbitMQ와 연결된 모든 서비스에 Push됨을 알 수 있음
