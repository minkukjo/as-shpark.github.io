레플리카 셋에 대해 알아보자
---

레플리카 셋
- 명시된 파드 개수를 안정적으로 유지해서 가용성을 높이는데 목적이 있다. 
- 레플리카셋은 셀렉터를 이용해서 새 파드를 식별한다. 파드에 OwnerReference가 없거나 OwnerReference가 컨트롤러가 아니고 레플리카셋의 셀렉터와 일치한다면 레플리카셋이 즉각 파드를 가지게 될 것이다.
그러므로 단독 파드가 레플리카셋의 셀렉터와 일치하는 라벨을 가지지 않도록 하는 것이 좋다. 
- 레플리카셋은 지정된 수의 파드가 항상 실행되도록 보장하지만 파드에 대한 업데이트는 제공하지 않는다. 그러므로 레플리카셋을 직접적으로 사용하는 것보다는 디플로이먼트를 사용하는 것을 권장한다.

---

레플리카 셋 매니페스트
- apiVersion: apps/v1
- kind: ReplicaSet
- spec.template: 파드 템플릿
- spec.template.metadata.labels: 파드에 붙일 라벨 지정, 다른 컨트롤러가 취하지 않도록 다른 컨트롤러의 셀렉터와 겹치지 않도록 주의해야 한다.
- spec.template.spec.restartPolicy: 재시작 정책, 기본값이 Always만 허용된다.
- spect.selector.matchLabels: 라벨 셀렉터, 소유할 파드를 식별하는데 사용
- spec.template.metadata.labels는 spec.selector과 일치해야 하며 그렇지 않으면 API에 의해 거부된다.
- spec.replicas: 동시에 동작하는 파드의 수를 지정할 수 있다. (default: 1)

---

레플리카 셋에서 파드 제거 방법
- 라벨을 변경하면 레플리카셋에서 파드를 제거할 수 있다.
- spec.replicas 필드를 업데이트 한다. 

레플리카 셋을 HPA(Horizontal Pod Autoscalers)의 대상으로 지정하기
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50

- CPU 사용량에 따라 파드를 오토스케일링 한다.
```
---

레플리카셋의 대안
- 디플로이먼트:
디플로이먼트는 레플리카셋을 소유하거나 업데이트를 하고, 파드의 선언적인 업데이트와 서버측 롤링 업데이트를 할 수 있는 오브젝트이다.
오늘날에는 주로 디플로이먼트로 파드의 생성과 삭제 그리고 업데이트를 오케스트레이션하는 메커니즘으로 사용한다.

참고: 
- [쿠버네티스 공식 홈페이지] https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/

