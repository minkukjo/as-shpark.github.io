## 쿠버네티스에서 프로메테우스 설치 및 kubelet 데이터 조회 ##

#### 단계 ####
1. kubelet api 인증 모드 변경
2. 서비스 계정 생성
3. 클러스터롤 생성 및 서비스 계정과 바인딩
4. 프로메테우스에서 사용할 컨피그 맵 생성
5. 프로메테우스 파드 및 서비스 생성
6. 프로메테우스 대시보드 확인

#### 1. kubelet api 인증 모드 변경 ####
- 기본적으로 kubelet api에 접근하기 위해서는 인증이 필요하다. 인증모드에는 Node, ABAC, RBAC, Webhook 가 있는데 모든 인증 방식을 허용하려면 AlwaysAllow로 바꿔주면 된다. 
- 이 작업은 모든 노드(마스터 노드 포함)에 각각 적용한다. 
```
1) kubelet의 설정 파일을 연다.
> vi /var/lib/kubelet/config.yaml

2) 인증 모드를 AlwaysAllow로 변경한다. 
...
authorization:
  mode: AlwaysAllow
...

3) kubelet을 재시작 한다. 
> systemctl restart kubelet
```
- 참고: https://kubernetes.io/docs/reference/access-authn-authz/authorization/#authorization-modules
- 참고2: https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/

#### 2. 서비스 계정 생성 ####
- pod를 생성하고 인증을 위한 서비스 계정을 하나 생성한다.
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus-access
  namespace: default

* 서비스 계정은 쿠버네티스가 직접 관리하는 사용자 계정이다.  
* 각각의 서비스 계정에는 secret이 할당되어 비밀번호 역할을 하게 된다.
```


#### 3. 클러스터롤 생성 및 서비스 계정과 바인딩 ####
- 클러스터롤이란 클러스터 전체 사용 권한을 관리하는 롤이다. 
- 특정 네임스페이스만이 아닌 전체 클러스터에 대한 리소스 정보를 조회하기 위해 클러스터롤을 하나 생성하고 이를 서비스 계정과 바인딩 시키는 작업이 필요하다. 
```
#클러스터롤 생성
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus-access
rules:
  -
    apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources:
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["*"]
  - nonResourceURLs: ["*"]
    verbs: ["*"]
---
#클러스터롤바인딩
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus-access
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus-access
subjects:
- kind: ServiceAccount
  name: prometheus-access
  namespace: default


* 클러스터롤 필드 설명
- apiGroup: 롤이 사용할 API 그룹을 지정한다.
- resource: 어떤 자원에 접근할 수 있는지 명시한다. 
- verbs: 해당 자원에 어떤 작업을 할 수 있는지 명시한다. Create, Get, List, Update 등이 있지만 위에서는 모든 작업을 명시했다. 
```

#### 4. 프로메테우스에서 사용할 컨피그 맵 생성 ####
- 프로메테우스 기동 시 필요한 설정 파일(prometheus.yml)을 아래와 같이 컨피그 맵으로 만들어 놓는다. 
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 10s
    scrape_configs:
    - job_name: 'kubelet'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        insecure_skip_verify: true
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    - job_name: 'cadvisor'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        insecure_skip_verify: true
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token


* kubelet과 cadvisor 에 대한 job_name을 설정한다. 
* 인증을 위한 token 경로를 bearer_token_file에 지정한다. 
  (특정 서비스 계정으로 파드를 만들면 위에 명시된 위치로 secret의 내용이 복사가 된다)
* 서버 인증서의 유효성 검사를 건너 띄기 위에 insecure_skip_verify를 true로 설정한다. 
```
- 참고: https://prometheus.io/docs/prometheus/latest/configuration/configuration/

#### 5. 프로메테우스 파드 및 서비스 생성 ####
- 프로메테우스 파드를 실행할 Deployment를 정의한다. 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  selector:
    matchLabels:
      app: prometheus
  replicas: 1
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      serviceAccountName: prometheus-access
      containers:
      - name: prometheus
        image: prom/prometheus:v2.1.0
        ports:
        - containerPort: 9090
          name: default
        volumeMounts:
        - name: config-volume
          mountPath: /etc/prometheus
      volumes:
      - name: config-volume
        configMap:
         name: prometheus-config
---
kind: Service
apiVersion: v1
metadata:
  name: prometheus
spec:
  selector:
    app: prometheus
  type: NodePort
  ports:
  - protocol: TCP
    port: 9090
    targetPort: 9090
    nodePort: 30001


* configMap의 mount path를 /etc/prometheus로 지정한다.(프로메테우스 설정 파일 기본 경로)
* 외부에서 대시보드에 접근하기 위해 NodePort 타입으로 서비스를 만든다.  
```

#### 6. 프로메테우스 대시보드 확인 ####
- http://<노드IP>:30001 <- 주소로 접근하여 대시보드를 확인한다. 
- Status -> Targets 메뉴의 화면에서 cadvisor와 kubelet의 state가 up 인지 확인한다. 
![prometheus-01.PNG](/assets/img/prometheus-01.PNG){: width="600" height="400"}