---
layout: post
title: '쿠버네티스에서 private registry 이미지 사용하기'
subtitle: ''
categories: container
tags: Kubernetes
comments: false
---

## 쿠버네티스에서 private registry 이미지 사용하기 ##
쿠버네티스에서 private registry에 있는 이미지를 이용하여 파드를 생성하는 방법에 대해서 알아본다. 작업 순서는 아래와 같다.

1. 도커 설치  
2. registry 서버 구축  
3. insecure-registries 설정  
4. 이미지 빌드 및 푸시  
5. 파드 생성  

### 1. 도커 설치(CenOS7 기준) ###
우선 private registry를 구축하기 위해 도커를 설치해야 한다.  
도커를 설치할 노드에 다음과 같은 순서로 설치를 진행한다.
```
1) 도커 설치 전, 관련 패키지를 설치한다.
> yum install -y yum-utils device-mapper-persistent-data lvm2 

2) 저장소를 설정한다.
> yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

3) 도커 패키지를 설치한다.
> yum update && yum install docker-ce

4) 만약 설치시 도커가 실행되지 않았다면 다음 명령어로 도커를 시작한다.
> systemctl restart docker
```

### 2. private registry 컨테이너 띄우기 ###
도커를 설치했으면 레지스트리 이미지를 가지고, private registry를 구축하는 방법을 알아본다. 
```
1) 레지스트리 이미지를 가져온다.
> docker pull registry:latest

2) 해당 이미지로 레지스트리 컨테이너를 띄운다.
> docker run --name private-registry -v /etc/registry:/var/lib/registry/docker/registry/v2 -d -p 5000:5000 registry

3) 정상 실행하는지 확인한다.
> docker ps -l
```

### 3. insecure-registries 설정 ###
다른 도커 노드에서 생성된 레지스트리에 접근하여 push/pull 등의 작업을 하기 위해서는 기본적으로 https 통신을 사용한다. 테스트 목적으로 사용하려는 경우, 인증서 설치가 번거로우니 다음과 같이 레지스트리와 통신하려는 도커 노드에 다음과 같이 insecure-registries를 등록한다. 
```
1) 쿠버네티스의 각 worker node의 daemon.json 파일을 수정한다.
> vi /etc/docker/daemon.json

{

 "insecure-registries": ["<ip>:5000"]

}

2) 파일을 수정하고 나서는 도커를 재시작 해준다.
> systemctl restart docker
```

* 만약 Window PC에서 이미지를 push 하는 경우, docker desktop의 setting > daemon > insecure-registries 항목에 ip:port 정보를 추가해 준다. 

### 4. 로컬 PC에서 이미지 빌드 및 push ###
로컬 PC에서 자신의 app 이미지를 빌드하고 registry 서버로 push 한다.
```
1) 로컬 PC에서 앱 이미지를 빌드한다. 
> docker build -t <registry server IP:Port>/test/test01:latest .


2) 빌드한 이미지를 레지스트리 서버에 push한다.

> docker push <registry server IP:Port>/test/test01:latest

3) 이미지가 registry server에 제대로 푸시되었는지 확인한다.
- 이미지 목록 확인
> curl -X GET http://<registry server IP:Port>/v2/_catalog 

- 특정 이미지의 tag 확인
> curl -X GET http://<registry server IP:Port>/v2/test/test100/tags/list
```

* 이미지 빌드 시, registry server의 IP와 Port 정보는 반드시 들어가야 한다. 

### 5. 쿠버네티스에서 파드 생성 ###
쿠버네티스 환경에서 파드를 생성할 때 다음과 같이 registry에 등록된 이미지를 지정한다.
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
     app: pod
spec:
  containers:
  - name: container
    image: <registry server IP:Port>/test/test01
    ports:
    - containerPort: 8080
```
