[Pod - container, Label, NodeSchedule]

1. Container 속성
- 파드 내에는 여러 컨테이너가 존재한다.
- 한 컨테이너는 여러개의 포트를 가질 수 있지만, 파드 내에 컨테이너끼리 포트 중복은 허용되지 않는다.
- 한 파드는 하나의 IP를 할당 받는데 클러스터 내부에서만 접근이 가능하고 재생성 시, IP는 변경이 된다.

2. Label 속성
- label은 여러 파드를 특정 목적에 따라 묶기 위해 사용된다. (web, was, db, dev, location 등..)
- label은 key와 value 쌍으로 구성된다. 
- 한 파드에는 여러 label을 지정할 수 있다. 

3. Node Schedule 속성
- 파드가 생성될 때 생성될 노드를 직접 선택하는 방법과 스케쥴러가 자동으로 선택하는 방법 2가지가 있다.
- 직접 선택하려면 노드에 label을 달고 파드를 만들 때 nodeSelector 항목에 해당 노드 label을 넣는다. 
- 스케쥴러가 판단할 때는 각 노드에 사용 가능한 리소스를 기준으로 파드가 요구하는 리소스를 비교하여 적절한 노드를 선택한다. 

[sample]

apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
     app: pod
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080

----------------------------------
[Service - ClusterIP, NodePort, LoadBalancer]
1. ClusterIP 타입
- 클러스터 내에서만 접근 가능, 아 IP로 연결된 파드에 연결 가능하다. 
- 파드를 여러 개 연결할 수 있다. 서비스가 트랙픽을 분산한다(같은 셀렉터일 경우)
- 운영자와 같은 인가된 사용자만 접근 가능, 각 서비스 디버깅이나 쿠버네티스 대시보드 관리 등에 사용

2. NodePort 타입
- 모든 노드에 똑같은 포트가 할당이 되어서 어떤 노드에서든 해당 포트로 접근하면 연결된 서비스에 접근할 수 있다.
- nodePort 속성에는 30000 ~ 32767 사이의 번호만 입력가능하며 생략했을 시 자동으로 할당하게 된다. 
- externalTrafficPolicy: Local로 하게 되면 특정 노드로 접근할 경우 해당 노드에 생성된 파드에만 트래픽이 전달된다. 
  pod가 없는 node에 접근을하면 접근이 안된다. 
- 보통 node의 호스트 IP는 내부망에서만 접근 가능하기 때문에 내부망에서 접근해야 할 경우 사용, 외부 연동용(데모)으로 잠깐 사용하기도 함

3. Load Balancer 타입
- NodePort 기능에 추가하여 Load Balancer를 생성한다. 
- 이 로드밸런서로 접근하기 위한 외부 IP는 쿠버네티스를 설치했을 때 개별적으로 생성되지 않는다. 별도로 IP를 할당해주는 플러그인이 설치되어 있어야 한다. 
- 외부에 시스템을 노출하는 용도로 사용

[sample]

apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
     app: pod
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080

---

apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30000
  type: NodePort
  externalTrafficPolicy: Local

---
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
  type: LoadBalancer

-----------------------------------------
[Volume]
emptyDir
- 파드 내 컨테이너끼리 볼륨을 공유
- 파드가 재생성되면 데이터는 사라지므로 일시적인 데이터만 저장해야 한다.

ex)
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-1
spec:
  containers:
  - name: container1
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount1
  - name: container2
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount2
  volumes:
  - name : empty-dir
    emptyDir: {}


hostPath
- 파드가 올라가 있는 노드에 볼륨을 생성하고 파드끼리 공유한다.
- 파드가 다른 노드에 재생성된다면 해당 볼륨에는 접근할 수 없다. 
- 파드 자신이 할당되어 있는 호스트의 데이터를 읽거나 쓸 경우에 사용한다. 
- 생성 시, 사전에 해당 노드에 경로가 미리 존재해야 한다.

ex)
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate

* DirectoryOrCreate : 만약 노드에 path가 없으면 자동 생성하겠다는 내용

PVC / PV
- 볼륨의 형태는 다양하다.(로컬 or 외부 클라우드, NFS)
- 퍼시스턴트 볼륨은 Admin 영역이고 퍼시스턴트 볼륨 클래임은 User 영역이다.
- PVC를 만들 때 storageClassName을 빈값("")으로 하면 현재 생성되어 있는 PV 중 가장 적합한 것이 연결된다.
- PV에 클래임이 한번 바인딩되면 PV는 다른 클래임에서 사용할 수 없다. 
-----------------------------------------
Object - ConfigMap, Secret
- 운영환경과 상용환경에서 서비스의 보안 접근이 서로 다를 경우
- 이미지의 값을 각각 관리할 수 없으므로 환경에 따라 변하는 값은 외부에서 결정할 수 있게 하는데 
  그걸 도와주는 것이 ConfigMap과 Secret이다.
- 일반적인 설엉은 ConfigMap으로 보안이 필요한 경우에는 Secret으로 한다.
- 이 두 값을 설정하면 해당 컨테이너 서비스에 환경변수(Env)로 들어가게 된다. 
  서비스에서는 환경 변수의 값을 읽어서 사용하게 된다.

ConfigMap, Secret 설정 방법
상수를 넣는 방법
- key와 value로 구성, spring 이므로 boolean값을 넣고 싶으면 ''(쿼테이션)를 달아줘야 한다.
- Secret의 value를 넣을 때는 base64 인코딩을 해서 넣어야 한다.
- 일반적인 Object 값들은 쿠버네티스 DB에 저장이 되는데 Secret 값은 메모리에 저장이 된다.
- Secret은 1MB 까지만 넣을 수 있고(시스템 자원에 영향을 준다) ConfigMap은 제약이 없다.

파일을 환경변수에 넣는 방법
- 파일을 통으로 ConfigMap에 넣을 수 있는데 파일명이 key가 되고 파일 안의 내용이 value가 된다.
- 키를 넣을 때는 파일명을 넣으면 좀 이상할 수 있으니 재정의해서 넣을 수 있다.
- 파일을 ConfigMap에 넣는 건 대시보드에서 지원안함 -> 직접 마스터에 접속해서 kubectl 명령으로 해야 한다.
  kubectl create configmap cm-file --from-file=./file.txt
- Secret의 경우 다음과 같이 명령을 날린다.
  kubectl create secret generic sec-file --from-file=./file.txt
  명령을 통해 파일 안의 내용의 base64로 인코딩 된다. 
- Pod 생성 시 ConfigMap을 한번 주입하면 끝이므로 ConfigMap을 변경한다고 해도 기존 파드에는 아무 영향이 없다.

볼륨 마은트
- 파일을 ConfigMap에 담는 것 까지는 똑같다.
- Pod를 만들 때 mount path를 정의하고 path안에 파일을 마운팅할 수 있다.
- ConfigMap이 변경되면 기존 Pod에 마운팅된 내용도 변하게 된다. 





-----------------------------------------
[컨트롤러]
1) 오토힐링: 파드나 노드에 문제 발생 시, 다른 노드에 파드 생성
2) 오토 스케일링: 파드에 부하가 걸릴 때 파드를 복제함
3) 소프트웨어 업데이트
4) Job : 컨트롤러가 필요한 순간에만 파드를 만들어 사용 후 삭제

Replication Controller(deprecated 지만 많이 사용함) -> ReplicaSet

1) templcate
- 라벨과 selector로 연결
 
 -----------------------------------------
[Deployment]
 - 한 서비스가 운영 중인데, 이 서비스를 업데이트해야 해서 재배포할 때 도움을 주는 오브젝트

1) ReCreate 방식
- 기존 v1 파드를 삭제하여 자원을 회수한 후에 v2 파드를 다시 생성한다. 
- 일시적인 정지가 가능한 서비스에서 사용 가능한 방식이다. 

2) Rolling Update 방식
- 기존 v1 파드를 삭제하기 전, v2 파드를 먼저 생성하고 v1 파드를 나중에 삭제한다.
- 가용 리소스가 충분할 때 사용할 수 있고, 다운타임이 발생하지 않는다는 장점이 있다. 

3) Blue/Green 방식
- controller를 이용하여 v1 파드를 생성한 경우 사용하는 방식으로 상당히 많이 사용된다.
- controller를 새로 생성하여 v2 파드를 생성시키고, 기존 서비스에 연결되어 있는 파드를 새로 생성한 v2 파드로 변경 시킨다. 
- v2 파드에 문제가 발생 시 기존 v1 파드로 재연결 시켜주면 되기 때문에 롤백이 쉽다는 장점이 있고, 문제가 없다면 기존 파드를 삭제시키면 된다. 
- 다운타임은 없지만 리소스가 2배 필요하게 된다.

4) Canary 방식
불특정 다수에 대한 테스트 시
- v2용 컨트롤러와 파드를 만든 후, 기존 서비스에 v1 파드들과 함께 같이 연결시킨다.
- v2 파드에 문제가 생긴다면 v2용 컨트롤러의 replicas를 0으로 바꾸기만 하면 된다. 
- 추후 문제가 없다면 v2 파드를 증가 시키고 v1 파드는 삭제한다. 

특정 인원에 대한 테스트 시
- v2용 컨트롤러와 파드, 서비스를 생성한다.
- 서비스에 접근할 수 있는 ingress controller를 통해 url에 따라 서로 다른 서비스에 접근할 수 있도록 한다.
- 추후 문제가 없다면 v2 파드를 증가 시키고 v1 파드는 삭제한다. 
- 다운타임은 없지만 리소스가 더 많이 필요하게 된다. 

 -----------------------------------------

 [DeamonSet, Job, CronJob]

1. DaemonSet
- ReplicaSet과는 달리 모든 노드에 하나씩 파드를 배치한다. 
- 모든 노드에 파드 구성이 필요한 경우 DaemonSet을 사용한다.

2. DaemonSet을 사용하는 예
1) 로그 수집(fluentd)
2) 노드의 성능 수집(Prometheus)
3) 노드들을 스토리지로 사용(GlusterFS) 

3. Pod 생성 방식에 따른 작동의 차이점
1) Pod로 생성했을 경우
- Node가 다운되면 다른 Node에 재생성되지 않는다.
2) ReplicaSet으로 생성했을 경우
- Node가 다운되면 컨트롤러가 다른 Node에 파드를 재생성한다. 
- 파드가 만약 사용되지 않는다면 파트 안의 컨테이너는 restart 되므로 replicaSet으로 만드는 파드는 항상 사용되어야 한다. 
3) Job으로 만드는 경우
- Node가 다운되면 컨트롤러가 다른 Node에 파드를 재생성한다.
- 파드가 사용되지 않으면 파드는 종료된다. (삭제되는 것은 아니다)

4. CronJob의 역할
- Job을 일정시간에 주기적으로 생성하는 역할을 한다. 
- 백업이나 메시지 전송, 업데이트 체크 등 주기적으로 반복되는 작업에 사용한다. 

5. DaemonSet의 특징
- 각 Node에는 파드를 하나씩만 만들 수 있다. 
- nodeSelector 속성을 설정하여 특정 Node에는 파드 생성을 제외할 수 있다. 
- hostPort 속성을 사용하여 서비스처럼, 파드의 특정 포트와 연결하는 node의 포트를 지정할 수 있다.

6. Job의 특징
- template를 설정하여 한번 실행할 Pod를 설정할 수 있으며, selector는 자동으로 설정된다. 
- Pod가 종료되면 Job도 종료된다. 
- completions 속성을 설정하면 해당 개수만큼 Pod가 생성되어 차례로 실행된다. 
- parallelism 속성을 설정하면 해당 개수만큼 Pod가 같이 실행된다. 
- activeDeadlineSeconds 속성을 설정하여 Job의 종료 시간을 설정할 수 있는데 주로, 
  시간이 짧게 걸리는 Job이 행이 걸려서 종료되지 못하는 경우를 방지하고자 사용한다.

7. cronJob의 특징
- JobTemplate와 schedule 속성을 이용하여 주기적으로 생성할 Job의 내용을 지정한다.
- ConcurrencyPolicy 기능을 이용하여 Job의 생성 방식을 결정한다. 
  allow: 생성 주기가 되면 기존 Job의 실행 여부와 상관없이 새로운 Job을 만들어 실행한다. 
  forbid: 기존에 생성된 Job이 있으면 다음 생성 주기 때 새로운 Job을 생성하지 않는다. 
  Replace: 기존에 생성된 Job이 있으면 다음 생성 주기 때, 새로운 Pod를 만들어 기존의 Job과 연결 시킨다.(기존 파드는 삭제가 된다)
           기존에 생성된 Job이 없으면 새로운 Job을 생성한다. 



 -----------------------------------------

[Pod - Lifecycle]

status.phase: 
- pod의 대표 상태를 나타낸다.
- Pending, Running, Succeeded, Failed, Unknown 으로 나뉜다.
status.conditions: 
- pod의 단계별 상태와 원인을 알 수 있다. 

containerStatuses.state: 
- 컨테이너의 상태와 이유를 나타낸다.
- Wating, Running, Terminated로 나뉜다. 

Pod의 상태별 구체적인 단계
1) Pending
- Initialized: 
  본 Container가 실행되기 전 초기화 시켜야 할 내용이 있을 경우InitContainer가 먼저 실행된다. 
  볼륨이나 보안세팅을 위한 사전 세팅, innitContainers라는 속성으로 설정 가능하다. 
  성공적으로 끝났거나 아예 설정을 하지 않은 경우 true가 된다. 
- PodScheduled:
  Pod를 어는 노드에 배치할지가 결정되면 true가 된다. 
- 위 두 과정동안 Container의 상태는 Wating이고 reason은 ContainerCreating 이다.

2) Running
- 컨테이너가 정상적으로 실행이되면 Pod와 Container 상태는 Running이 된다.
- 하나 또는 모든 컨테이너가 비정상적이라서 재실행 되는 경우, 컨테이너의 상태는 Wating이고 reason은 CrashLoopBackOff가 된다.
- Pod는 위와 같은 상황이더라도 Running으로 간주하지만, 내부 컨디션의 ContainerReady와 Ready는 False이다.
- 모든 컨테이너가 정상작동하면 모든 컨디션은 True가 된다.
- Pod가 Running이더라도 컨테이너가 비정상적일 수 있으므로 Pod뿐만 아니라 컨테이너 상태도 모니터링 해줘야 한다.

3) Failed / Succeeded
- 크론잡의 경우 모든 일을 마치고 파드의 상태는 Succeeded나 Failed로 바뀐다.
- 작업 컨테이너 중 하나라도 Error로 Terminated가 되면 Failed가 되고 
  모든 컨테이너가 Completed로 Terminated가 되면 Succeeded가 된다. 
- 성공이건 실패건 간에 Pod의 ContainerReady와 Ready는 False가 된다. 
- Pod가 Pending 중에 바로 Fail로 빠지는 경우도 있다. 
- 통신 장애가 발생하면 Pod는 Unknown이 되지만 빨리 해결되면 다시 기준 상태로 되지만 빨리 해결되지 않으면 Failed로 가기도 한다.

-----------------------------------------

[Service - Headliess, Endpoint, ExternalName]

1. DNS Server
- 동적 생성되는 파드가 또 다른 동적 생성되는 파드나 서비스에 접근하기 위해 필요
- 파드에서 클러스터 내부/외부의 DNS Server를 이용해서 서비스나 다른 Pod 혹은 서버의 IP 주소를 알아내어 접근할 수 있다. 

2. Headless 서비스
- Pod1과 Pod2를 연결하고 싶을 때 Headless service에 연결하면 <pod이름>.<서비스이름> 형태로 DNS 서버에 도메인이 등록되므로 
  이 도메인을 통해 각 파드에 연결할 수 있다. 
- headless 서비스를 만들면 서비스를 통하지 않고 파드에서 파드로 직접 호출이 가능하다. 
- 일반 CluseterIP service를 만들 때 DNS에 등록되는 도메인 형태
  ex) service1.default.svc.cluster.local (서비스명.네임스페이스명.서비스약어.DNS명)
- 파드가 ClusterIP service에 붙을 때 DNS에 등록된느 도메인 형태
  ex) 20-109-5-12.default.pod.cluster.local (IP주소.네임스페이스명.pod.DNS명)
      같은 네임스페이스 내에서 서비스는 앞자리(서비스명)만 입력해도 되지만 파드는 풀 네임(FQDN: Fully Qualified Domain Name)을 모두 입력해야 한다. 잎부분이 IP라 거의 쓸수가 없다)
- Headless 서비스를 만드는 방법
  서비스를 만들 때 clusterIP:None 으로 설정한다.
  파드를 만들 때, hostname 속성에 도메인을 지정하고 subdomain 속성에 headless 서비스명을 입력해야 한다. 
  DNS에 서비스의 IP는 연결된 모든 파드의 IP가 등록된다. 
  DNS에 파드의 도메인 형태
  ex) pod4.headless1.default.svc.cluster.local (파드호스트명.서비스명.네임스페이스명.svc.DNS명)
  

3. Endpoint
- 서비스와 pod를 연결할 때 selector로 lable을 지정하는데, 쿠버네티스 내부적으로는 이 정보를 가지고 Endpoint라는 오브젝트를 만든다.
  이 오브젝트의 명은 서비스명과 같고, 내부에 Pod의 접속정보를 가지고 있다.
  이런 방식으로 selector를 지정하지 않고도 Endpoint를 직접 지정하여 서비스와 Pod를 직접 연결할 수 있다. 
  외부 아이피를 가리킬 수도 있다. 하지만 외부 IP는 변경 가능성이 있으므로 이럴 때 이용하는 것이 ExternalName이다.  


4. ExternalName
- 파드가 외부 서버에 접근 시, 외부 서버 접근 경로가 변경되었을 때 파드의 재생성 없이 접근할 수 있도록 방법 제공
- 서비스에 ExternalName에 특정 도메인 이름을 등록해 놓으면, DNS server로부터 해당 Name에 대한 IP를 찾을 수 있다. 
  GitHub등으로부터 데이터를 가지고 올 수 있다. 
- 서비스의 externalName에 도메인을 설정하면, DNS cache가 내부와 외부 DNS를 찾아서 IP를 알아낸다. 
  파드는 서비스만 알고 있으면 언제들지 외부 서버로 접근할 수 있다. 변경되더라도 파드를 재배포할 필요가 없다. 



-----------------------------------------

[Volume - Dynamic Provisioning, StorageClass, Status, ReclaimPolicy]

volume 특징
- 쿠버네티스 클러스터와는 별도로 관리
- 크게 외부망 볼륨과 내부망 볼륨으로 나뉜다.
  외부망 볼륨: 아마존, 구글, 애저 등에서 제공해주는 클라우드 스토리지
  내부망 볼륨: 각 노드에서 제공 해주는 볼륨(hostPath 나 local로 만드는 로컬 볼륨 or eeph나 GlusterFS 스토리지 솔루션을 이용한 볼륨), NFS 등..

- 관리자는 클러스터 밖의 볼륨을 이용하여 PV를 만든다.(스토리지 크기와 AccessMode 지정)
- 사용자는 원하는 볼륨 크기와 액세스 모드를 정해서 PVC를 만들면 쿠버네티스가 알아서 적절한 PV와 연결시켜 준다. 
  액세스 모드 (RWO, ROM, RWM: 실제로 각 노드마다 지원되는 모드가 다르다)

Dynamic Provisioning
- 사용자가 PVC를 만들면 자동으로 PV가 만들어지고 볼륨과 연결시켜 주는 기능
- 모든 PV에는 상태가 존재한다. 
- PV를 삭제할 때는 정책적인 부분이 있다.
- DP를 사용하기 위해서는 적절한 스토리지 솔루션을 선택하여 설치를 해줘야 한다.

StorageClass 오브젝트
- 동적으로 PV를 만들 수 있다.
- PVC를 만들 때 StorageClassName에 StorageClass명을 넣어주면 자동으로 PV가 생성된다.
- SC는 추가할 수도 있고 디폴트 값을 설정(SCN을 생략했을 때 적용)해 놓을 수도 있다.

Status
- Available: 최초 PV가 생성된 상태
- Bound: PV가 PVC에 연결된 상태
         이 상태에서 볼륨과 연결되는 건 아니고, 실제 Pod가 생성되면서 PVC를 이용할 때 볼륨과 연결된다. 
- Released: PVC가 삭제되어 PV와 연결이 끊어졌을 때의 상태
- Failed: 위의 과정들 중에 오류가 발생 했을 때

ReclaimPolicy
- Retain: 
  default 정책, PV가 Released 상태가 된다, 볼륨의 데이터는 보존, 하지만 다시 PVC에 연결하지 못하므로 재사용 불가, 삭제도 수동으로 해야함
- Delete:
  PV도 같이 지워짐, StorageClass 사용시 Default, Volume에 따라 데이터가 삭제되기도 함, 재사용 불가
- Recycle:
  Deprecated
  PV의 상태가 Available이 된다, 데이터 삭제, PV 재사용 가능

-----------------------------------------

Pod (ReadnessProbe, LivenessProbe)

ReadinessProbe
- 파드가 Running 상태가 되면 서비스로부터 트개픽 유입이 시작되는데, 이 때 App이 booting 중이면 사용자들이 에러 페이지를 볼 수 있다.
  ReadinessProbe를 사용하게 되면 App 구동 순간에 트래픽 실패를 없앨 수 있다. App이 구동되기 전까지는 서비스와 연결되지 않게 해준다. 

LivenessProbe
- App에 대한 장애 상황을 감지해 준다.
- Pod에 톰캣을 띄운 컨테이너가 구동되고 있을 때, 톰캣 자체에는 문제가 없지만 톰캣에 올라와 있는 서비스에 장애(memory overflow 등)가 발생하면
  파드는 Running 상태로 트래픽 유입이 계속 된다. 하지만 서비스에 장애가 있으므로 사용자는 에러 페이지를 보게 될 것이다. 
- LivenessProbe를 달아주게 되면 해당 App에 문제가 생겼을 때 Pod를 재실행하게 하기 때문에 잠깐의 트래픽 에러는 발생할 수 있지만 지속적인 에러 상태를 방지한다.


사용방법
- ReadinessProbe나 LivenessProbe는 사용 목적이 틀릴 뿐이지 설정할 수 있는 내용은 같다.
- httpGet, Exec, tcpSocket 등으로 앱애 대한 상태를 알 수 있다.(셋 중 하나는 꼭 정해야 하는 속성이다)
- httpGet: 
  Port(8080), Host(localhost), Path(/ready), HttpHeader(language:ko), Scheme(http, https) 등 체크 가능
- Exec: 특정 명령어를 날려서 그에 따른 결과를 체크한다.
  Comand (cat /tmp/ready.txt)
- tcpSocket:
  Port(8080), Host(localhost)
- 옵션 값으로 initalDelaySeconds, periodSeconds, timeoutSeconds, successThreshold, failureThreshold 가 있다.
  initalDelaySeconds: default 0초, 최초 probe를 하기전에 delay 시간
  periodSeconds: default 10초, probe를 체크하는 시간의 간격
  timeoutSeconds: default 1초, 이 지정된 시간까지 결과가 와야 한다.
  successThreshold: default 1회, 몇 번 성공 결과를 받아야 정말 성공으로 인정을 할 건지 지정
  failureThreshold: default 3회, 몇 번 실패 결과를 받아야 정말 실패로 인정을 할 건지 지정
  설정 값을 안넣는다면 모두 default 값으로 지정이 된다.

- ReadinessProbe를 설정하면 Pod와 Container의 상태는 Running 이더라도 이 Pobe가 성공하지 않으면
  Condition의 ContainerReady와 Ready 값은 false가 된다.
  false 상태가 지속되면 Endpoint에서는 이 Pod의 IP를 NotReadyAddr로 간주를 하고 셔비스에 연결하지 않는다. 
  쿠버네티스는 컨테이너 상태가 running이 되면 initalDelaySeconds에 명시된 대로 지연하고 있다가 시간이 되면 Probe를 체크한다.
  실패를 하면 periodSeconds에 명시된 시간 후에 다시 체크한다. 
  successThreshold에 적힌 숫자만큼 성공을 하면 condition의 상태는 true가 되고 endpoint도 정상적으로 Addresses로 간주를 하면서 서비스와 연결이 된다. 




  