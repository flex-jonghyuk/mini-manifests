# replicaSet

쿠버네티스의 ReplicaSet은 하나 이상의 Pod 복제본(replicas)을 유지하기 위한 리소스입니다. ReplicaSet의 주요 목적은 지정한 수의 Pod 인스턴스가 항상 실행되도록 보장하는 것입니다.

ReplicaSet은 Pod의 수를 관리하며, Pod가 충분히 실행되고 있는지 확인합니다. 만약 실행 중인 Pod의 수가 지정된 수보다 적다면, ReplicaSet은 더 많은 Pod를 시작합니다. 반대로, 실행 중인 Pod의 수가 지정된 수보다 많다면, 일부 Pod를 종료합니다.

ReplicaSet이 관리할 Pod는 spec.selector에 정의된 레이블 선택자를 통해 결정됩니다. 이 선택자는 Pod 템플릿의 레이블과 일치해야 합니다. Pod 템플릿은 ReplicaSet이 새 Pod를 생성할 때 사용하는 템플릿입니다.

그러나, 일반적으로 ReplicaSet를 직접 사용하기보다는 Deployment 리소스를 사용하는 것이 더 일반적입니다. Deployment는 ReplicaSet의 상위 개념으로, Pod의 업데이트를 처리하는 역할을 합니다. Deployment를 사용하면 롤링 업데이트, 롤백, 일시 중지 등의 기능을 사용할 수 있습니다. 따라서, 대부분의 경우에는 ReplicaSet보다 Deployment를 사용하는 것이 더 유용합니다.

## 속성

먼저, Pod과 ReplicaSet을 생성하기 위한 쿠버네티스 매니페스트 YAML 파일을 작성해 보겠습니다. 그런 다음, 각 설정 옵션에 대해 설명하겠습니다.

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
```

위의 매니페스트 파일에서 다음과 같은 주요 설정 옵션이 있습니다:

1. `apiVersion`: 사용할 쿠버네티스 API 버전을 지정합니다. ReplicaSet은 `apps/v1` API 그룹에 속해 있습니다.

2. `kind`: 쿠버네티스 리소스의 종류를 지정합니다. 여기서는 ReplicaSet을 생성합니다.

3. `metadata`: 리소스에 대한 메타데이터를 제공합니다. `name` 필드는 ReplicaSet의 이름을, `labels` 필드는 ReplicaSet을 식별하는데 사용할 레이블을 지정합니다.

4. `spec`: ReplicaSet의 동작을 제어하는 필드입니다.

   - `replicas`: 유지하려는 Pod 인스턴스의 수를 지정합니다.
   - `selector`: 이 ReplicaSet이 관리할 Pod를 선택하는 방법을 지정합니다. 여기서는 레이블이 `app: myapp`인 Pod를 선택합니다.
   - `template`: ReplicaSet이 새 Pod를 생성할 때 사용하는 템플릿을 지정합니다.

5. `template.metadata.labels`: 새로 생성되는 Pod에 부여할 레이블을 지정합니다. **이 레이블은 위의 `selector.matchLabels`와 일치해야 합니다.**

6. `template.spec`: 생성할 Pod의 스펙을 정의합니다.

   - `containers`: Pod 내에서 실행할 컨테이너 목록입니다. 각 컨테이너에는 `name`과 `image`를 지정해야 하며, 필요에 따라 `ports` 등 다른 설정을 지정할 수 있습니다.
   - `containers.name`: 컨테이너의 이름을 지정합니다.
   - `containers.image`: 컨테이너에서 사용할 도커 이미지를 지정합니다.
   - `containers.ports`: 컨테이너에서 열려야 하는 네트워크 포트를 지정합니다.

이 매니페스트를 사용하면, `nginx:1.14.2` 이미지를 사용하는 이름이 `nginx-container`인 컨테이너를 가진 Pod를 3개 생성하고 유지합니다. 이 Pod들은 `app: myapp` 레이블이 붙어 있어서, `myapp-replicaset`이라는 이름의 ReplicaSet이 관리하게 됩니다.
