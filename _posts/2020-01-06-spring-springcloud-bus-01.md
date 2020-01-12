---
layout: post
title: 'Spring Cloud Bus에 대해 알아보자'
subtitle: ''
categories: spring
tags: spring-cloud
comments: false
---

#### 1. 스프링 클라우드 버스란? ####
- 마이크로서비스를 카프카 및 래빗MQ와 같은 경량 메시지브로커에 연결한다.
- Config Server에서 일어난 구성 변경 등을 브로드캐스트 하는데 이용할 수 있다.

#### 2. 스프링 클라우드 버스 작동 순서 ####
![스프링 클라우드 서비스 작동](/assets/img/cloud-bus-001.PNG){: width="600" height="400"}
<br/>
1. 설정 파일에 변경이 발생한다.
2. config server에 설정 정보가 변경되었다는 사실을 알린다.
3. config server는 연결된 message queue에 메시지를 발행한다.
4. 마이크로서비스들은 메시지를 수신하여 변경 정보를 업데이트 한다.

#### 3. 스프링 클라우드 버스 구성 방법 ####
#### [Config Server] ####
1. build.gradle 파일에 spring-cloud-starter-bus-amqp 디펜던시를 추가한다.
```
implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
```
2. 버스 새로고침 기능을 노출하기 위해 application.properties에 다음과 같이 추가한다.
```
management.endpoints.web.base-path=/actuator
management.endpoints.web.exposure.include=bus-refresh
```
3. rabbitmq 접속 정보를 application.properties에 추가한다.
```
spring.rabbitmq.host=<IP주소>
spring.rabbitmq.username=<사용자명>
spring.rabbitmq.password=<사용자비밀번호>
```

#### [Config Client] ####
1. build.gradle 파일에 spring-cloud-starter-bus-amqp 디펜던시를 추가한다.
```
implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
```
2. rabbitmq 접속 정보를 application.properties에 추가한다.
```
spring.rabbitmq.host=<IP주소>
spring.rabbitmq.username=<사용자명>
spring.rabbitmq.password=<사용자비밀번호>
```

#### 4. 스프링 클라우드 실행 테스트 ####
1. config server 앱, congfig client 앱을 실행한다.
2. 설정 파일의 값을 변경한다.
3. config server로 /actuator/bus-refresh를 POST로 호출한다.
4. client 앱에서 설정 값이 바뀌었는지 확인한다. 