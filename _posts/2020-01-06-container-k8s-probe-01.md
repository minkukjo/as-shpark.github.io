컨테이너 프로브에 대해 알아보자

#### 1. 컨테이너 프로브란? ####
- kublet에 의해 주기적으로 수행되는 컨테이너 진단을 말한다.
- 진단을 위해 kublet은 컨테이너가 구현한 핸들러를 호출하는데 핸들러의 타입은 다음과 같다.
> Eexc : 컨테이너 내에서 지정된 명령을 실행, 명령어의 상태 코드가 0으로 종료되면 성공한 것으로 간주한다.  
> TcpSocket: 지정된 포트에서 컨테이너 IP 주소에 대해 TCP 검사를 수행한다. 포트가 활성화되어 있다면 성공한 것으로 간주한다.  
> HTTPGet: 지정된 포트에서 컨테이너 IP 주소에 대한 HTTP Get 요청을 수행한다. 응답 상태 코드가 200보다 크고 400보다 작으면 성공한 것으로 간주한다.


#### 2. 프로브의 종류 ####
- readnessProbe: 
> 컨테이너가 요청을 처리할 준비가 되었는지 여부를 확인한다.  
> 이 프로브를 사용하게 되면 App 구동 순간에 트래픽 실패를 없앨 수 있다.  
> App이 제대로 구동되기 전까지는 서비스와 연결되지 않게 해준다. 
- livenessProbe:
> App에 대한 장애 상황을 감지해 준다.  
> 이 프로브가 실패한다면 kublet은 해당 컨테이너를 죽이고, 컨테이너는 재시작 정책의 대상이 된다.  
> 재시작 시, 잠깐의 트래픽 에러는 발생할 수 있지만 지속적인 에러 상태는 방지해 준다.  

#### 3. 프로브의 설정 항목 ####
- ReadinessProbe나 LivenessProbe는 사용 목적이 다를 뿐 설정할 수 있는 내용은 같다.
- 프로브 설정 시, 핸들러 타입 3가지(httpGet, exec, tcpSocket) 중 하나는 꼭 정해야 한다. 
> httpGet: Port, Host, Path, HttpHeader, Scheme 등 체크 가능  
> exec: 특정 명령을 날려서 그에 따른 결과 체크
> tcpSocket: Port, Host 활성화 여부 체크  
- 옵션으로 설정할 수 있는 값은 다음과 같다.
> initialDelaySeconds: 최초 probe를 하기 전에 delay 시간(default: 0초)  
> periodSeconds: probe를 체크하는 시간의 간격(default: 10초)
> timeoutSeconds: 이 지정된 시간까지 결과가 와야 한다.(default: 1초)  
> successThreshold: 몇 번 성공 결과를 받아야 정말 성공으로 인정할 건지를 지정한다.(default 1회)  
> failureThreshold: 몇 번 실패 결과를 받아야 정말 실패로 인정할 건지를 지정한다.(default 3회)

#### 4. ReadinessProbe 설정 예 ####
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-readiness-exec1
  labels:
    app: test
spec:
  containers:
  - name: rtest
    image: kubetm/app
    ports:
    - containerPort: 8080	
    readinessProbe:
      exec:
        command: ["cat", "/test/ready.txt"]
      initialDelaySeconds: 5
      periodSeconds: 10
      successThreshold: 3
    volumeMounts:
    - name: my-volume
      mountPath: /test
  volumes:
  - name : host-path
    hostPath:
      path: /tmp/test
      type: DirectoryOrCreate
  terminationGracePeriodSeconds: 0

- 쿠버네티스는 컨테이너 상태가 Running이 되면 initialDelaySeconds(5초)에 명시된 대로 지연하고 있다가 
  시간이 되면 Probe를 체크한다.
- readinessProbe를 설정하면 Pod와 Container의 상태는 Running 이더라도 
  이 Probe가 성공하지 않으면 ContainerReady와 Ready의 값은 false가 된다. 
- 실패를 하면 periodSeconds(10초)에 명시된 시간 후에 다시 체크한다.
- false 상태가 지속되면 서비스의 endpoint에서는 이 파드의 IP를 NotReadyAddr로 간주하고
  서비스에 연결하지 않는다.
- successThreshold에 적힌 숫자만큼 성공을 하면 condition의 상태는 true가 되고 
  endpoint도 정상적으로 Addresses로 간주 하면서 서비스와 연결이 된다. 
 
```
