#### 파드 목록 가져오기 ####
```
kubectl get Pods

NAME             READY   STATUS    RESTARTS   AGE
frontend-pxj4r   1/1     Running   0          5s
pod1             1/1     Running   0          13s
pod2             1/1     Running   0          13s
```

#### yml로 오브젝트 생성하기 ####
```
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```

#### 레플리카 셋 상태 확인 ####
```
kubectl describe rs/<이름>
```

#### 레플리케이션 컨트롤러 상태 확인 ####
```
kubectl describe replicationcontrollers/<이름>
```
