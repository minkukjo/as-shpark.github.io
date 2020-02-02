---
layout: post
title: 'headless 서비스에 대해 알아보자'
subtitle: ''
categories: container
tags: Kubernetes
comments: false
---

### 1. 헤드리스 서비스란? ###
- 헤드리스 서비스란 클러스터 IP가 없는 서비스를 말한다. 
- 주로 파드에 로드밸런싱이 필요 없거나 단일 서비스 IP가 필요 없을 때 사용한다. 
- 헤드리스 서비스를 만들고 파드를 연결 시키면, 연결된 파드는 DNS 서버에 {호스트명}.{서비스명} 형태로 등록이 되므로 다른 파드에서 해당 도메인명으로 직접 접근이 가능하다.

### 2. 헤드리스 서비스의 등록 예 ###
```
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: headless-pod
  ports:
  - port: 8080
    targetPort: 8080
```
- clusterIP 속성에 None으로 설정을 하게 되면 헤드리스 서비스가 생성된다. 
- 아래와 같이 cluste ip가 none인 것을 확인할 수 있다. 
![headless-01](/assets/img/headless-01.PNG)

### 3. 헤드리스 서비스에 파드를 연결 시키면 어떻게 될까? ###
```
apiVersion: v1
kind: Pod
metadata:
  name: headless-pod1
  labels:
    app: headless-pod
spec:
  hostname: headless-pod
  subdomain: headless-service
  containers:
  - name: container
    image: kubetm/init
```
- headless 서비스에 연결하기 위해서는 hostname 속성과 subdomain 속성을 지정해야 한다. subdomain 속성은 headless 서비스명과 동일하게 지정한다. (서비스명과 동일하게 지정하지 않으면 도메인이 생성되지 않으므로 주의!)

- 연결된 서비스의 dns 레코드를 조회해 보면 아래와 같이, 해당 서비스의 도메인명에 파드의 IP가 연결되어 있는 것을 확인할 수 있다. 

![headless-02](/assets/img/headless-02.PNG)

- 아래와 같이 {호스트명}.{서비스명}으로 도메인이 생성된 것도 확인할 수 있다. 해당 도메인 명으로 파드의 IP가 연결된 것을 확인할 수 있다. 

![headless-03](/assets/img/headless-03.PNG)