---
layout: post
title: 'Spring Cloud Config Server'
subtitle: ''
categories: spring
tags: spring-cloud
comments: false
---

#### 1. Spring Cloud Config Server 특징 ####
- 중앙 집중식 마이크로서비스 구성을 지원한다.
- 중앙 집중식 구성 서버는 모든 다른 환경에 속한 마이크로서비스의 모든 구성을 보유한다. 
- 일반적으로 프로덕션 구성에 대한 액세스가 다른 환경에 비해 더 제한적이길 원한다. 최소한 프로덕션을 위한 별도의 중앙 집중식 구성 서버를 사용하는 것이 좋다.
- 다중 마이크로서비스의 구성은 단일 깃 리포지토리에 저장된다. 


#### 2. Spring Cloud Config 구성요소 ####
- Spring Cloud Config Server: 버전 관리 리포지토리(Git or Subversion)로 백업된 중앙 집중식 구성 노출을 지원한다. 
- Spring Cloud Config Client: 애플리케이션이 스프링 클라우드 컨피그 서버에 연결하도록 지원한다. 

#### 3. Spring Cloud Config Server 구현 단계 ####
1) Spring Cloud Config Server를 설정한다.
```
1) 의존성에 org.springframework.cloud:spring-cloud-config-server 를 추가한다.
2) 메인 클래스에 @EnableConfigServer 어노테이션을 추가한다.
```
2) 로컬 깃 리포지토리를 설정하고, 이를 스프링 클라우드 컨피그 서버에 연결한다. 
```
1) git을 설치하고 로컬 깃 리포지토리 폴더를 만든다.
  cd d:
  mkdir git_repo
  git init

2) 컨피스 서버 프로젝트에서 application.properties에 다음과 같이 구성한다.
  spring.application.name=config-server
  server.port=8888
  spring.cloud.config.server.git.uri=file:///d:git_repo

```
3) Spring Cloud Config Client를 사용해 마이크로서비스를 구성 서버에 연결한다.
```
1) gradle.build에 의존성 추가

dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-config'
}

ext {
	set('springCloudVersion', 'Hoxton.SR1')
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}

2) application.properties의 이름을 bootstrap.properties로 바꾸고 다음 내용을 입력한다.
spring.application.name=microservice-a
spring.cloud.config.uri=http://localhost:8888

3) 만약 dev 환경에 대한 설정을 진행하는 경우, 설정 파일 이름을 microservice-a-dev.properties라고 지정하고, bootstrap.properties에는 spring.profiles.active=dev 라고 내용을 추가하면 된다. 

* 부트스트랩 애플리케이션 콘텍스트
- 마이크로서비스 애플리케이션의 상위 콘텍스트 이다.
- 외부 구성 로드(예: 스프링 클라우드 구성 서버) 및 구성 파일 암호 해독(외부 및 로컬)을 담당한다.
- bootstrap.yml 또는 bootstrap.properties를 사용해 구성한다.
- bootstrap.yml은 ApplicationContext에 의해 로드되며 application.yml을 사용하기 전에 로드된다. 
- 시작 시, 스프링 클라우드는 애플리케이션 이름으로 설정 서버에 HTTP 호출을 하고 해당 애플리케이션 구성을 검색한다. 
- 부트스트랩 프로세스 중에 검색된 구성은 내부에 정의된 구성보다 우선한다.
```
