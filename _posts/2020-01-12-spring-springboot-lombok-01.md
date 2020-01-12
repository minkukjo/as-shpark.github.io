---
layout: post
title: 'eclipse와 gradle로 롬복(lombok) 사용하기'
subtitle: ''
categories: spring
tags: spring-boot
comments: false
---

#### **1. 이클립스(STS)에 롬복 설치하기** ####
1) 롬복 홈페이지에서 롬복(lombok.jar)파일을 다운로드한다.     
- [롬복 다운로드](https://projectlombok.org/download)

2) cmd 창에서 다운로드 받은 폴더로 이동 후, 해당 파일을 실행한다. 
```
 java -jar lombok.jar
```

3) 파일 실행 후, 실행되는 화면에서 specify location 항목을 클릭한다.   
![01.PNG](/assets/img/lombok-01.png){: width="600" height="400"}

4) STS가 설치된 경로로 이동하여 실행파일을 선택한다.   
![02.png](/assets/img/lombok-02.png){: width="600" height="400"}

5) Quit installer를 선택한다.  
![03.png](/assets/img/lombok-03.png){: width="600" height="400"}

#### **2. gradle에서 롬복 설정하기** ####
```
dependencies {
  compileOnly 'org.projectlombok:lombok:1.18.10'
  annotationProcessor 'org.projectlombok:lombok:1.18.10'
}

...

* compileOnly
- 컴파일 경로에만 종속성을 추가하고 빌드 출력에는 추가되지 않는다. 
- 라이브러리 모듈이 런타임에 제공되지 않아야 하는 경우라면 사용 시, 어플리케이션 크기를 줄일 수 있다. 

* annotationProcessor 
- 어노테이션 프로세서 라이브러리에 종속성을 추가해야 할 때 사용한다.  
- 어노테이션 프로세서 클래스 경로에 해당 종속성을 추가하게 된다.

* Annotation Processing이란?
- 컴파일 시 어노테이션 처리를 진행한다. 
- 컴파일 시에 어노테이션을 읽어 소스 코드를 자동 생성할 수 있다.
- 사용 예) lombok / JPA의 metamodel
```

#### **3. 주요 롬복 어노테이션 정리** ####

* @Getter : get 메서드를 생성한다.

* @Setter : set 메서드를 생성한다. 

* @ToString : toString 메서드를 생성한다. 
```
> @ToString(exclude = "password")
- excute 속성을 사용하면 해당 필드를 제외시켜 준다.
```

* @EqualsAndHashCode : equal과 hashcode 메서드를 생성한다.
```
> @EqualsAndHashCode(callSuper = true)
- callSuper = true로 설정하면 부모 필드 값들도 동일한지 체크를 하게 된다. 
```

* @AllArgsConstructor : 모든 필드를 파라미터로 받는 생성자를 생성해 준다. 
```
> @AllArgsConstructor(access = AccessLevel.PUBLIC )
- access 속성을 사용하면 접근제한을 설정할 수 있다. 
```

* @NoArgsConstructor : 파라미터가 없는 기본 생성자를 생성해 준다.

* @RequiredArgsConstructor : final이나 @NonNull인 필드 값만 파라미터로 받는 생성자를 만들어준다. 

* @Data : @Getter, @Setter, @RequiredArgsConstructor, @ToString, @EqualsAndHashCode를 한꺼번에 설정해준다.

* @Builder : 자동으로 해당 필드에 builder pattern을 적용시켜준다.