#### 1. 레플리케이션 컨트롤러 ###3
- 파드의 수를 유지한다.
- 한개 혹은 복제된 여러 파드를 생성해서 유지해야 하는 경우 사용한다.

#### 2. 레플리케이션 컨트롤러 예제 ####
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
- spec.template 는 파드에 대한 개요이다.
- 파드 템플릿은 적절한 라벨과 재시작 정책을 지정해야 한다.
- 재시작 정책은 Always와 동일한 spec.temlate.spec.restartPolicy만 허용되며 특별히 지정되지 않으면 기본값이 된다.
- spec.selector 필드는 라벨 셀렉터이다. 컨트롤러는 직접 생성/삭제 하거나 다른 프로세스가 생성/삭제한 파드를 구분하지 않는다.
- spec.replicas 에는 동시에 실행할 파드의 수를 설정한다. 기본값은 1이다.


#### 3. 레플리케이션 컨트롤러 삭제 방법 ####
1. kubectl delete 명령어를 사용한다. 
2. REST API나 클라이언트 라이브러리를 사용하는 경우 명시적으로 레플리카를 0으로 스케일하고 파드 삭제를 기다린 후, 레플리케이션 컨트롤러를 삭제해야 한다.
3. 레플리케이션 컨트롤러만 삭제할 경우 옵션으로 --cascade=false를 지정한다. 
4. 파드를 컨트롤러 관리 대상에서 제외 시키고자 할 경우, 파드의 라벨을 변경한다. 

#### 4. 롤링 업데이트 ####
- 레플리케이션 컨트롤러는 파드를 하나씩 교체함으로써 서비스에 대한 롤링 업데이트를 쉽게 하도록 되어있다.
