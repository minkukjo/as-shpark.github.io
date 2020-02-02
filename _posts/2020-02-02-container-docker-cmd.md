---
layout: post
title: '도커 명령어 정리'
subtitle: ''
categories: container
tags: docker
comments: false
---

#### **1. 이미지 관련 명령어** ####

##### **1) 이미지 빌드 하기**  #####
* docker image build -t 이미지명[:태그명] Dockerfile의 경로
```
> docker image build -t example/echo:latest .
- t: 이미지명을 지정한다. 태그명도 지정할 수 있으며, 생략 시에는 latest 태그가 붙는다. 

> docker iamge build -f Dockerfile-test -t example/echo:latest .
- f: Dockerfile의 파일명 지정 

> docker image build --pull=true -t example/echo:latest .
- pull: 매번 베이스 이미지를 강제로 새로 받아온다. 
```

##### **2) 도커 허브에 등록된 리포지토리 검색하기** #####
* docker search [options] 검색 키워드
```
> docker search --limit 5 mysql
- limit: 최대 검색 건수를 제한한다.
- 검색 결과는 stars 순으로 매겨진다. 
```

##### **3) 이미지 내려 받기** #####
* docker image pull [options] 리포지토리명[:태그명]
```
> docker image pull jenkins:latest
```

##### **4) 보유한 이미지 목록 보기** #####
* docker image ls [options] [리포지토리[:태그]]
```
> docker image ls
```

##### **5) 이미지에 태그 부여하기** #####
* docker image tag 기반이미지명[:태그] 새이미지명[:태그]
```
> docker imaage tag example/echo:latest example/echo:0.1.0
```

##### **6) 태그 붙지 않은 이미지 조회** #####
```
> docker images -f "dangling=true" -q
```
##### **7) 태그 붙지 않은 이미지 모두 삭제** #####
```
> docker rmi $(docker images -f "dangling=true" -q)
```

#### **2. 컨테이너 관련 명령어** ####

##### **1) 컨테이너 실행하기**  #####
* docker container run [options] 이미지명[:태그] [명령] [명령인자..]  
* docker container run [options] 이미지ID [명령] [명령인자..]  
```
> docker container run -d -p 9000:8080 example/echo:latest
> docker container run -it alpine:3.7

[run 관련 주요 옵션]
-i : 컨테이너를 실행할 때 컨테이너 쪽 표준 입력과의 연결을 그대로 유지한다.
-t : 유사 터미널 기능을 활성화하는 옵션
--rm : 컨테이너를 종료할 때 컨테이너를 파기하도록 하는 옵션
-v : 호스트와 컨테이너 간에 디렉터리나 파일을 공유하기 위해 사용하는 옵션
-d : 백그라운드로 실행
-p : 포트포워딩 설정
```

##### **2) 컨테이너에 이름 붙이기**  #####
* docker container run --name [컨테이너명] [이미지명]:[태그]  
```
> docker container run -t -d --name gihyo-echo example/echo:latest

- 같은 이름의 컨테이너를 새로 만들려면 기존 컨테이너를 삭제해야 하므로, 이름 붙인 컨테이너는 운영환경에서는 거의 사용되지 않는다. 
```


##### **3) 도커 컨테이너 목록 보기** #####
* docker continaer ls [options]
```
> docker continaer ls
- 아무 옵션 없이 명령을 실행하면 현재 실행 중인 컨테이너의 목록이 출력된다. 

> docker container ls -q
- -q: 컨테이너 ID(축약형)만 추출할 수 있다. 

> docker container ls ==filter "name=echo1"
> docker container ls --filter "ancestor=example/echo"
- --filter: 특정 조건을 만족하는 컨테이너 목록을 보여준다. 

> docker continaer ls -a
- -a: 정료된 컨테이너를 포함한 컨테이너 목록을 볼 수 있다. 
```

##### **4) 컨테이너 정지하기** #####
* docker container stop 컨테이너 ID 또는 컨테이너명
```
> docker container stop echo
```

##### **5) 컨테이너 재시작하기** #####
* docker container restart 컨테이너ID 또는 컨테이너명
```
> docker container restart echo
```

##### **6) 컨테이너 파기하기** #####
* docker container rm 컨테이너ID 또는 컨테이너명
```
> docker container rm f66f6f2013da
> docker container rm -f f66f6f2013da
- -f: 실행 중인 컨테이너를 삭제할 때 사용한다. 
```

##### **7) 표준 출력 연결하기** #####
* docker container logs [options] 컨테이너ID 또는 컨테이너명
```
> docker container logs -f $(docker container ls --filter "ancestor=jenkins" -q)

- -f: 새로 출력되는 표준 출력 내용ㅇ을 계쏙 보여준다. 
- 컨테이너의 출력 내용 중 표준 출력으로 출력된 내용만 확인할 수 잇으므로 파일 등에 출력된 로그는 볼 수 없다. 
```

##### **8) 실행 중인 컨테이너에서 명령 실행하기** #####
* docker container exec [options] {컨테이ID 또는 컨테이너명} {실행할 명령}
```
> docker container exec echo pwd
> docker container exec -it echo sh
- 컨테이너 내부의 상태를 확인하거나 디버깅하는 용도로 사용할 수 있다.
- 컨테이너 안에 든 파일을 수정한느 것은 애플리케이션에 의도하지 않은 부작용을 초래할 수 있으므로 운영 환경에서는 절대 해서는 안된다. 
```

##### **9) 파일 복사하기** #####
* docker container cp [options] 컨테이너 ID 또는 컨테이너명:원본파일 대상파잉
```
> docker container cp echo:/echo/main.go .
> docker container cp dummy.txt echo/tmp
- 실행 중인 컨테이너와 파일을 주고 받는다. 
- 디버깅 중 컨테이너 안에서 생성된 파일을 호스트로 옮겨 확인할 목적으로 사용하는 경우가 대부분이다. 
```

#### **3. 운영/관리 명령어** ####

##### **1) 컨테이너 일괄 삭제** #####
* docker container prune [options]
```
> docker continer prune
- 실행 중이 아닌 모든 컨테이너를 삭제한다.
```

##### **2) 이미지 일괄 삭제** #####
* docker image prune [options]
```
> docker image prune
- 태그가 붙지 않은(dangling) 모든 이미지를 삭제한다. 
- 실행 중인 컨테이너 이미지는 삭제되지 않는다. 
```
##### **3) 이미지/컨테이너 일괄 삭제** #####
```
> docker system prune
- 사용하지 않는 도커 이미지 및 컨테이너, 볼륨, 네트워크 증 모든 도커 리소스를 일괄적으로 삭제한다. 
```
##### **4) 사용 현황 확인하기** #####
* docker container stats [options] [대상 컨테이너 ID ...]
```
> docker container stats 
- 시스템 리소스 사용 현황을 컨테이너 단위로 확인할 수 있다. 
```