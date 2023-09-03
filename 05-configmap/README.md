# configmap

# ConfigMap이란?

ConfigMap은 쿠버네티스(Kubernetes)에서 사용하는 리소스 중 하나로, 설정 정보나 구성 파일을 저장하여 파드(Pod)와 같은 다른 리소스와 공유할 수 있게 해줍니다. ConfigMap은 컨테이너 이미지와 분리된 설정 정보를 제공하므로, 애플리케이션 코드 변경 없이 설정을 수정하거나 재배포할 수 있습니다.

## 주요 특징

1. **환경 변수**: ConfigMap의 데이터를 파드의 환경 변수로 사용할 수 있습니다.
2. **파드의 커맨드라인 인자**: ConfigMap의 데이터를 파드의 커맨드라인 인자로 사용할 수 있습니다.
3. **파일 시스템에 볼륨으로 마운트**: 파드 내에서 파일로 접근할 수 있게 ConfigMap을 볼륨으로 마운트할 수 있습니다.

## ConfigMap 예제

다음은 ConfigMap을 생성하는 기본 예제입니다.

1. **ConfigMap 생성**:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-configmap
data:
  database.url: "https://example.com/db"
  database.user: "admin"
  database.password: "secretpassword"
```

위 예제는 `database.url`, `database.user`, `database.password`라는 3개의 키-값 쌍을 가진 ConfigMap을 정의합니다.

2. **파드에서 ConfigMap 사용**:

ConfigMap의 데이터를 파드에서 환경 변수로 사용하는 예제입니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-configmap
data:
  database.url: "https://example.com/db"
  database.user: "admin"
  database.password: "secretpassword"
---
apiVersion: v1
kind: Pod
metadata:
  name: use-configmap
spec:
  containers:
    - name: app-container
      image: myapp:latest
      env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: example-configmap
              key: database.url
        - name: DATABASE_USER
          valueFrom:
            configMapKeyRef:
              name: example-configmap
              key: database.user
        - name: DATABASE_PASSWORD
          valueFrom:
            configMapKeyRef:
              name: example-configmap
              key: database.password
```

이 예제에서는 `example-configmap` ConfigMap의 데이터를 환경 변수로 사용하여 `myapp:latest` 이미지를 실행하는 파드를 생성합니다.

기억해야 할 주요 사항은, ConfigMap만 변경하여도 파드에는 자동으로 반영되지 않습니다. 변경 사항을 파드에 적용하려면 파드를 재생성해야 합니다.

### 볼륨 마운트

컨픽맵을 작성하고 Pod volume에 마운트시키면 기동시점에 환경변수를 파일로 떨어트린다고 한다. 애플리케이션에서는 요걸 받아다가 참조가 가능함

흠 진작에 알았으면 요걸 한번 해봤을텐데 싶긔

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sample-configmap
data:
  config.properties: |
    databaseURL=https://example.com/db
    user=admin
    password=secret
---
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  containers:
    - name: app-container
      image: myapp:latest
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: sample-configmap
```

```js
const fs = require("fs");

const rawData = fs.readFileSync("/etc/config/app-config.json");
const config = JSON.parse(rawData);
const databaseUrl = config.database.url;
```

### deployment에 연결

conatiner.env 프로퍼티에 연결할 수 있기 때문에 deployment에도 연결이 가능함. 이렇게 하면 deployment가 생성하는 레플리카에서 모두 configmap을 참조하게끔 하는게 가능해짐

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  DATABASE_URL: "https://mydatabase.example.com"
  DATABASE_USER: "admin"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: myimage:latest
          env:
            - name: DATABASE_URL
              valueFrom:
                configMapKeyRef:
                  name: my-config
                  key: DATABASE_URL
            - name: DATABASE_USER
              valueFrom:
                configMapKeyRef:
                  name: my-config
                  key: DATABASE_USER
```

## 오늘의 실습

- 환경변수가 진짜 들어가는지 확인할 수 있는 엔드포인트를 하나 판 상태로 이미지를 하나 말아본다.
- service, configmap, deployment를 모두 사용해 서비스를 노출시키고 api요청을 해보자
- 값을 바꾸고 재기동을 시켜보자
- `minikube service myconfig-service --url`
- `http://127.0.0.1:64646/study`
