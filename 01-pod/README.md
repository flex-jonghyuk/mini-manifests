# Pod

쿠버네티스(Kubernetes)의 Pod는 쿠버네티스 클러스터에서 가장 작은 배포 단위로, 하나 이상의 컨테이너를 그룹화하고 관리하는 역할을 합니다.

Pod는 컨테이너의 집합체로서, 각 Pod는 동일한 네트워크 네임스페이스에 속하며, 필요한 경우 같은 스토리지 볼륨(volume)을 공유할 수 있습니다. 이런 특성 때문에 Pod 내부의 컨테이너들은 서로 localhost를 통해 통신할 수 있으며, 동일한 파일 시스템에 접근할 수 있습니다.

## 속성

- spec: Pod의 동작을 정의하는 스펙입니다. 이는 containers(Pod에서 실행될 컨테이너 목록), initContainers(초기화 작업을 수행하는 컨테이너), volumes(Pod에 마운트할 볼륨 목록), nodeName(Pod를 배치할 노드의 이름), hostNetwork(Pod가 노드의 네트워크 네임스페이스를 사용해야 하는지 여부), restartPolicy(Pod에서 실행되는 컨테이너가 실패했을 때의 재시작 정책) 등을 포함할 수 있습니다.

- status: Pod의 현재 상태를 나타냅니다. 이는 사용자가 설정하는 것이 아니라 쿠버네티스 시스템에 의해 관리되는 정보입니다.

## pod의 리소스 관리

Pod 내의 컨테이너는 CPU와 메모리 리소스에 대한 요청과 제한을 설정할 수 있습니다. 요청은 해당 컨테이너가 필요로 하는 리소스의 양을 나타내며, 제한은 해당 컨테이너가 사용할 수 있는 리소스의 최대량을 나타냅니다.

아래는 resource limit이 추가된 ReplicaSet 매니페스트 YAML 파일입니다:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.14.2
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 200Mi
            limits:
              cpu: 200m
              memory: 400Mi
```

매니페스트에서 추가된 resource 관련 설정은 다음과 같습니다:

1. `resources`: 컨테이너의 리소스 사용에 대한 요청과 제한을 정의합니다.

2. `requests`: 컨테이너가 필요로 하는 리소스의 양을 지정합니다. 이 값을 바탕으로 쿠버네티스 스케줄러는 컨테이너를 실행할 노드를 결정합니다.

   - `cpu`: 필요한 CPU 양을 지정합니다. 단위는 'm' (밀리CPU)입니다.
   - `memory`: 필요한 메모리 양을 지정합니다. 단위는 'Mi' (메비바이트)나 'Gi' (기비바이트) 등입니다.

3. `limits`: 컨테이너가 사용할 수 있는 리소스의 최대량을 지정합니다. 이 값이 넘어가면, 컨테이너는 종료되거나 다시 시작될 수 있습니다.

   - `cpu`: 사용할 수 있는 CPU의 최대 양을 지정합니다.
   - `memory`: 사용할 수 있는 메모리의 최대 양을 지정합니다.

이렇게 리소스 요청과 제한을 설정하면, 쿠버네티스는 각 컨테이너에 필요한 리소스를 보장하고, 하나의 컨테이너가 너무 많은 리소스를 사용해 다른 컨테이너에 영향을 주는 것을 방지할 수 있습니다.

## 하나의 Pod에 컨테이너 여러개

`spec.containers`는 하나의 Pod에 대한 컨테이너 설정입니다. 한 Pod 내에 여러 컨테이너를 정의할 수 있고, 이렇게 설정된 모든 컨테이너는 동일한 Pod 내에서 함께 실행됩니다.

이런 방식을 이용하면, 한 Pod 내의 여러 컨테이너가 서로 밀접하게 협력하여 작업을 수행할 수 있습니다. 이러한 컨테이너 그룹을 '멀티-컨테이너 Pod'라고도 부릅니다. 멀티-컨테이너 Pod는 여러 컨테이너가 밀접하게 연결되어 있어야 하는 경우에 유용합니다.

예를 들어, 한 컨테이너가 웹 애플리케이션을 실행하고, 다른 컨테이너가 그 로그를 관리하거나, 데이터를 지속적으로 불러오는 등의 역할을 수행할 수 있습니다. 이런 방식을 통해 컨테이너 간의 결합도를 낮추고, 각 컨테이너의 역할을 명확하게 분리할 수 있습니다.

## kubectl

```shell
# Manifest 적용
kubectl apply -f my-replicaset.yaml

# Pod하나 죽이기
kubectl delete -n default pod my-replicaset-생겼던pod
```

죽이자마자 팟이 하나 더생기는 걸 볼 수 있다.
