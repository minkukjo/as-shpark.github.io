---
title2: 노드
category: 쿠버네티스
order: 1
---

#### 1. 노드의 컨디션 ####
- Ready: 노드가 파드를 수용할 준비가 되어 있음을 나타낸다.(true/false) 노드 컨트롤러가 일정 시간 동안 응답을 받지 못한다면 Unkonwn이 된다. 
- MemoryPressure: 메모리 상에 압박이 있는지 여부를 나타낸다.(true/false)
- PIDPressure: 프로세스 상에 압박이 있는지 여부를 나타낸다.(true/false)
- DiskPressure: 디스크 용량에 압박이 있는지 여부를 나타낸다.(true/false)
- NetworkUnavailable: 노드의 네트워크가 올바로 구성되어 있는지 여부를 나타낸다(true/false)

#### 2. 노드의 작동 특징 ####
- ready의 상태가 kube-controller-manager로 넘겨지는 인수 값(pod-eviction-timeout)보다 더 길게 Unknown / False로 유지되는 경우, 노드 상의 모든 파드는 삭제되도록 스케쥴 된다.
- 단, apiserver가 kubelet과 통신이 불가능하다면 파드에 대한 삭제 명령은 전달되지 못한다.

#### 3. 노드의 관리 ####
- 노드는 쿠버네티스에 의해 노드 오브젝트로 관리된다.
- 쿠버네티스는 metadata.name 필드를 근거로 상태 체크를 수행하여 노드의 유효성을 확인한다.

#### 4. 노드와 연관된 컴포넌트 ####
- 노드 컨트롤러:
> CIDR 할당 설정이 되어 있는 경우, CIDR 블럭을 할당한다.  
> 노드 리스트를 최신 상태로 유지한다.  
> 노드의 동작 상태를 모니터링 하고 각 상태마다 적절한 조치를 취한다.

- kubelet: 
> kubelet은 NodeStatus와 리스 오브젝트를 생성하고 업데이트 한다.  
> kubelet은 상태가 변경되거나 구성된 상태에 대한 업데이트가 없는 경우, NodeStatus를 업데이트한다. 기본 업데이트 주기는 5분이다.  
> kubelet은 10초마다 리스 오브젝트를 생성하고 업데이트한다.

