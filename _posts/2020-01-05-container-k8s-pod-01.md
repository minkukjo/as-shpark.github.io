---
layout: post
title: 'Pod의 Lifecycle'
subtitle: ''
categories: container
tags: Kubernetes
comments: false
---

#### 1. 파드(Pod)의 주요 특징 ####
- 파드란 쿠버네티스에 의해 관리되는 컨테이너들의 묶음이다. 
- 파드 내에는 여러 컨테이너가 존재할 수 있으며 각 컨테이너들은 동일한 IP 주소공간에 속하게 된다.
- 파드 내의 컨테이너 하나에는 여러 port를 할당할 수 있지만 파드 내 port는 중복이 되면 안된다.
- 파드는 생성 시 하나의 IP를 할당 받는데 이는 쿠버네티스 클러스터 내부에서만 접근이 가능하다. 

#### 2. 파드의 상태(Phase) ####
- phase란 Pod의 Lifecycle의 각 상태를 말한다. 
- 파드가 가지는 단계는 다음과 같다.
> Pending: 파드 생성이 승인되었지만 아직 이미지 생성이 완료되지 않은 상태  
> Running: 파드의 모든 컨테이너가 생성된 단계로써 적어도 하나의 컨테이너가 동작하거나, 시작 혹은 재시작 중인 상태  
> Succeeded: 모든 컨테이너들이 성공적으로 종료된 상태    
> Failed: 모든 컨테이너들이 종료 되었지만 하나 이상의 컨테이너가 실패로 종료된 상태  
> Unkown: 파드와의 통신 오류 등으로 상태를 알 수 없을 때
- 파드의 상태는 내부 컨테이너의 상태(status)에 의존적이다. 컨테이너의 상태는 다음과 같다.
> Waiting: 컨테이너의 기본 상태, 이미지를 내려받거나 시크릿을 적용하는 등의 작업을 수행 중인 상태이다.  
> Running: 이슈 없이 구동되는 상태  
> Terminated: 컨테이너 실행이 완료되어 구동을 멈춘 상태  

#### 3. 파드의 컨디션(condition) ####
- 파드의 상태(phase)마다 세부적인 통과 조건이 있는데 이를 condition이라고 한다.
- 컨디션 타입(type)은 아래와 같으며 각 타입에 대한 상태(status)는 True | False | Unknown으로 지정된다.
> PodScheduled: 파드가 스케쥴 완료됨  
> Ready: 파드는 요청을 수행할 수 있음  
> Initialized: 모든 초기화 컨테이너가 성공적으로 완료됨  
> Unschedulable: 스케쥴러가 어떤 이유로 인해 파드를 스케쥴할 수 없음  
> ContainerReady: 파드 내의 모든 컨테이너가 준비 상태  

```
그 외 컨디션의 하위 속성
lastProbTime: 마지막으로 조건이 조사된 시점
lastTransitionTime: 파드가 한 상태에서 다른 상태로 전환된 마지막 시점
message: 전환에 대한 사람이 읽을 수 있는 메시지
reason: 전환의 이유
```

#### * 초기화 컨테이너란? ####
```
파드의 앱 컨테이너가 실행되기 전에 먼저 실행되는 특수한 컨테이너로서
주로 앱 컨테이너가 실행되기 전에 해야 할 작업을 먼저 수행하기 위해 실행된다.
초기화 컨테이너는 여러 개를 지정할 수 있으며 각 초기화 컨테이너는 
순서대로 실행된다.   
```


#### 4. 파드의 라이프사이클에 따른 Phase와 Condition의 변화 ####
1) Pending:  
- 초기화 컨테이너가 실행될 때의 컨디션으로서 성공적으로 끝나거나 아예 설정을 하지 않았을 경우 Initialized 컨디션은 true가 된다.  
- 파드를 어느 노드에 배치할지 정하는 단계로서 결정이 끝나면 PodScheduled 컨디션은 true가 된다.  
- 위 두 과정 동안 컨테이너 상태는 wating이다. 

2) Running:  
- 모든 컨테이너가 정상적으로 실행되면 파드와 컨테이너의 상태는 Running이 된다.
- 하나 이상의 컨테이너가 비정상적이르면 파드는 Running이지만 컨테이너 상태는 Wating이다. 그리고 컨디션의 Ready와 ContainerReady는 false이다.

3) Failed / Succeeded:
- 작업 컨테이너 중 하나라도 error로 terminated 되면 Failed가 된다. 그게 아니라면 Succeeded가 된다. 
- 이 둘 경우 모두 컨디션의 Ready와 ContainerReady는 false이다.
- 파드가 Pending 중 바로 Failed로 가는 경우도 있고 통신 장애 발생 시 최초의 파드 상태는 Unknown이지만 해결이 지연 될 시 Failed로 가기도 한다.  
