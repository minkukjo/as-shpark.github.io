---
title2: 컨트롤러
category: 쿠버네티스
order: 1
---

#### 1. 컨트롤러의 역할 ####
- 하나 이상의 쿠버네티스 리소스의 유형을 추적하고, 상태를 관리한다. 
- 예) 잡(job) 컨트롤러는 여러 파드를 실행하여 작업을 수행하고 중지 시키는 쿠버네티스 리소스이다. 
잡 컨트롤러가 API 서버에 명령을 내리면 kubelet이 작업을 수행하기에 적합한 수의 파드를 실행하게 한다. 
- 클러스터 외부의 오브젝트를 컨트롤 하는 경우도 있다.(ex 클러스터 오토스케일링)
