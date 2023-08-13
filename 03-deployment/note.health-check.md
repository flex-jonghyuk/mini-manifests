# health check

## probe

Kubernetes에서 헬스체크(Health Check)는 컨테이너의 상태와 서비스의 가용성을 감지하고 관리하기 위해 사용됩니다. 이를 위해 Kubernetes는 "probe"라는 개념을 도입했습니다. Probe는 주기적으로 컨테이너를 폴링(polling)하고 컨테이너가 정상 작동 중인지 확인합니다.

"Probe"라는 용어는 기본적으로 "탐사"나 "조사"라는 의미를 가지고 있습니다. Kubernetes 컨텍스트에서 "probe"는 컨테이너가 정상적으로 동작하는지, 그리고 요청을 처리할 준비가 되었는지 등의 상태를 주기적으로 확인하거나 "조사"하는 메커니즘이라고 할 수 있습니다.

Kubernetes Probe는 이런 각각의 상태를 주기적으로 "조사"하며, 이를 통해 컨테이너의 생명주기와 서비스의 가용성을 관리합니다. Probe의 설정에 따라 Kubernetes는 자동으로 컨테이너를 재시작하거나, 트래픽을 분산시키는 등의 조치를 취할 수 있습니다.

## probe의 종류

1. Liveness Probe: 컨테이너가 정상적으로 동작 중인지를 확인합니다. 만약 Liveness Probe가 실패한다면, 해당 컨테이너는 **재시작**됩니다. 이는 서비스가 중단되었을 때 자동 복구를 위해 사용됩니다. -> 컨테이너가 살아있는지 확인

2. Readiness Probe: 컨테이너가 요청을 처리할 준비가 되었는지 확인합니다. 만약 Readiness Probe가 실패한다면, 해당 Pod는 서비스의 **트래픽을 받지 못하게 됩니다.** 이는, 예를 들어, 서비스가 시작하는 동안의 초기화 작업 중일 때 트래픽을 받지 않게 하기 위해 사용됩니다. -> 컨테이너가 요청을 처리할 수 있는지 확인. pod이 떠있기는 하는데 가용 상태는 아닌 것

3. Startup Probe: 컨테이너가 애플리케이션 내부에서 준비되었는지 확인합니다. Startup Probe는 애플리케이션의 시작 시간이 오래 걸리는 경우 유용하며, Startup Probe가 성공하기 전까지는 Liveness와 Readiness Probe는 무시됩니다. -> 컨테이너가 애플리케이션 내부에서 준비되었는지 확인
   - pod 실행은 되는데 성공하기 전까지는 가용 상태가 아님
   - 이게 성공해야 liveness readiness로 넘어감

### 계속 실패하는 경우

Probe가 계속 실패하는 경우 Kubernetes는 Probe의 유형과 관련 설정에 따라 다양한 조치를 취합니다.

1. **Liveness Probe**:

   - **계속 실패할 경우**: 해당 컨테이너는 재시작됩니다. `failureThreshold` 설정값을 초과하는 횟수만큼 Probe가 연속해서 실패하면 컨테이너는 재시작됩니다. 예를 들어, `failureThreshold`가 3으로 설정되어 있으면, Liveness Probe가 3번 연속 실패하면 컨테이너가 재시작됩니다.

2. **Readiness Probe**:

   - **계속 실패할 경우**: 해당 Pod는 서비스의 로드 밸런싱 대상에서 계속해서 제외됩니다. 즉, 해당 Pod로는 새로운 트래픽이 전송되지 않습니다. 이 상태는 Readiness Probe가 `successThreshold` 설정값을 초과하는 횟수만큼 연속해서 성공할 때까지 지속됩니다.

3. **Startup Probe**:
   - **계속 실패할 경우**: 다른 Probe(Liveness, Readiness)는 무시되고, `failureThreshold` 설정값을 초과하는 횟수만큼 Probe가 연속해서 실패하면 컨테이너는 재시작됩니다. 이는 Liveness Probe의 동작과 유사하나, 애플리케이션의 초기 시작에 초점을 맞춘 Probe입니다.

각 Probe의 `failureThreshold` 및 `successThreshold` 설정을 통해 얼마나 많은 실패나 성공이 연속해서 발생해야 조치를 취할지, 또는 상태를 변경할지 결정할 수 있습니다. 또한 `periodSeconds` 설정을 통해 Probe를 얼마나 자주 체크할지 결정할 수 있습니다.

Probe가 계속 실패하는 경우, 원인을 파악하고 문제를 해결하는 것이 중요합니다. Probe의 실패는 애플리케이션 오류, 설정 오류, 리소스 부족 등 다양한 원인이 될 수 있으므로 로그나 모니터링 도구를 활용하여 원인을 찾아야 합니다.

### 실패한 경우 -> 다시 성공하는 경우

Probe가 실패했다가 다시 성공하는 경우 Kubernetes는 다양한 방식으로 반응하며, Probe의 유형 (Liveness, Readiness, Startup)에 따라 반응이 달라집니다.

1. **Liveness Probe**:

   - **실패할 경우**: Kubernetes는 해당 컨테이너를 재시작합니다. -> Liveness Probe가 실패하여 컨테이너가 재시작되면, 컨테이너가 다시 시작될 때까지 그리고 Probe 설정에 따라 첫 번째 Probe를 시작할 때까지 약간의 시간이 소요
   - **다시 성공할 경우**: Kubernetes에서 특별한 조치를 취하지 않습니다. 컨테이너는 정상적으로 실행되는 것으로 간주되며, 아무런 추가적인 행동 없이 계속 실행됩니다.

2. **Readiness Probe**:

   - **실패할 경우**: 컨테이너가 서비스 요청을 처리할 준비가 되지 않았다고 판단되므로, 해당 Pod는 서비스의 로드 밸런싱 대상에서 제외됩니다. 즉, 새로운 트래픽이 해당 Pod로 전송되지 않습니다.
   - **다시 성공할 경우**: Kubernetes는 해당 Pod를 다시 서비스의 로드 밸런싱 대상에 포함시킵니다. 이렇게 되면 새로운 요청들이 해당 Pod로 전송될 수 있게 됩니다.

3. **Startup Probe**:
   - **실패할 경우**: 다른 Probe(Liveness, Readiness)가 일시적으로 무시됩니다. 그리고 Startup Probe는 설정된 주기에 따라 계속해서 체크됩니다.
   - **다시 성공할 경우**: 컨테이너 시작이 정상적으로 완료된 것으로 간주되며, 이후에는 Liveness와 Readiness Probe가 정상적으로 작동하게 됩니다.

Probe의 실패 및 성공에 따른 Kubernetes의 동작은 환경 설정에 따라 다를 수 있으며, `failureThreshold`, `successThreshold`, `timeoutSeconds` 등의 파라미터를 통해 세부 동작을 조절할 수 있습니다.

## 헬스체크 방법

각 Probe는 다음 세 가지 방법 중 하나를 사용하여 컨테이너를 체크할 수 있습니다:

1. HTTP GET: 특정 경로에 HTTP GET 요청을 보내 응답을 확인합니다.
2. TCP Socket: 특정 포트에서 TCP 소켓을 열어 연결이 성공하는지 확인합니다.
3. Exec: 컨테이너 내에서 특정 명령어를 실행하고, 그 결과(종료 상태 코드)를 확인합니다.

## 실제 설정과 옵션

readinessProbe 및 livenessProbe는 Kubernetes Deployment의 Pod 템플릿 내의 각 컨테이너 스펙에서 설정할 수 있습니다. 이 프로브(probe)들은 컨테이너의 건강 상태를 확인하고, Kubernetes가 서비스 트래픽을 어떻게 처리할지 결정하는 데 도움을 줍니다.

각 프로브는 다음과 같은 속성들을 가지고 있습니다:

- initialDelaySeconds: 컨테이너가 시작된 후 프로브가 시작되기 전에 대기하는 시간(초)
- periodSeconds: 프로브가 수행되는 간격(초) -> **폴링 간격**
- timeoutSeconds: 프로브가 타임아웃되기 전의 최대 시간(초)
- successThreshold: 프로브가 성공으로 간주되기 위한 최소 연속 성공 횟수
- failureThreshold: 프로브 실패 후 재시도하기 전의 실패 허용 횟수

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: myapp:1.0.0
      ports:
        - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
```

livenessProbe 또는 readinessProbe를 따로 설정하지 않으면, Kubernetes는 특별한 헬스 체크를 실행하지 않습니다. 즉, 컨테이너가 정상적으로 시작되었다면 실행 중으로 간주하고, Pod를 정상 상태라고 판단합니다.
