# ingress

## Kubernetes 인그레스란

Kubernetes 인그레스는 클러스터 외부에서 클러스터 내의 서비스로 HTTP와 HTTPS 경로를 정의해주는 API 오브젝트입니다. 이를 통해 외부에서 접근이 가능한 URL, 로드 밸런싱 트래픽, SSL, 웹 소켓과 같은 서비스를 간편하게 관리할 수 있습니다.

따라서, 일반적으로 Ingress는 7계층 (응용 계층)에서 가장 활발하게 작동하며, 필요에 따라 4계층 (전송 계층)의 일부 기능(TCP, UDP 트래픽 처리)도 지원할 수 있습니다. 그러나, Layer 4 (전송 계층) 로드 밸런싱이 필요한 경우에는 Kubernetes의 Service 오브젝트 타입 중 LoadBalancer를 사용하는 것이 더 적합할 수 있습니다.

## 인그레스 속성

- apiVersion, kind, metadata: Kubernetes 리소스를 정의하는 기본 필드들입니다.
- spec: 인그레스의 규칙 및 설정을 정의하는 곳입니다.
  - backend: 모든 요청을 하나의 서비스로 라우팅하려면 이 필드를 사용합니다.
  - rules: 호스트명과 경로를 기반으로 트래픽 라우팅 규칙을 정의합니다.
    - host: 요청의 HTTP 헤더에 지정된 호스트명입니다.
    - http: HTTP에 대한 규칙을 정의하는 곳입니다.
      - paths: 경로와 관련된 백엔드 서비스를 정의합니다.
        - path: 매치할 URL의 경로입니다.
        - backend: 경로에 매치되는 요청을 처리할 백엔드 서비스입니다.
          - service.name: 백엔드 서비스의 이름입니다.
          - service.port.name 또는 service.port.number: 백엔드 서비스의 포트입니다.

### spec.rules.http.paths.pathType

Kubernetes 1.18 버전부터 `spec.rules.http.paths.pathType` 필드에서 사용할 수 있는 경로 타입은 다음과 같습니다:

1. **ImplementationSpecific** (기본값): 이 경로 타입은 인그레스 클래스에 의존적이며, 인그레스 클래스가 경로 일치의 시멘틱을 정의합니다. 따라서 이 경로 타입은 인그레스 컨트롤러에 따라 다르게 작동할 수 있습니다.
2. **Prefix**: 이 경로 타입은 URL의 접두사가 일치하는 경우 요청을 서비스로 라우팅합니다. 예를 들어, `/api` 경로가 있고 `Prefix` 경로 타입이 설정된 경우, `/api/v1`, `/api/v2` 등 모든 경로가 일치합니다.
3. **Exact**: 이 경로 타입은 URL 경로가 정확히 일치할 때만 요청을 서비스로 라우팅합니다. 예를 들어, `/api` 경로에 `Exact` 경로 타입이 설정된 경우, 오직 `/api` 경로로의 요청만이 해당 서비스로 라우팅됩니다.

YAML 파일에서 이러한 경로 타입을 사용하는 예는 다음과 같습니다:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /admin
            pathType: Exact
            backend:
              service:
                name: admin-service
                port:
                  number: 80
```

이 예제에서, `/api` 경로는 "Prefix" 타입을 사용하므로 `/api/v1`, `/api/v2` 등의 요청도 `api-service`로 라우팅됩니다. 반면, `/admin` 경로는 "Exact" 타입을 사용하므로 오직 `/admin` 경로로의 요청만이 `admin-service`로 라우팅됩니다.

## 몇가지 활용법

여기 몇 가지 일반적인 상황과 그에 따른 Kubernetes Ingress 설정 예시가 있습니다:

### 1. fallback routing 설정

일치하는 경로나 호스트가 없을 때 특정 서비스로 요청을 라우팅하는 설정입니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: basic-ingress
spec:
  defaultBackend:
    service:
      name: default-service
      port:
        number: 80
```

### 2. 단일 서비스에 대한 호스트 기반 라우팅

호스트 이름을 기반으로 특정 서비스로 요청을 라우팅하는 설정입니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  rules:
    - host: "example.com"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend: # 서비스를 지정함
              service:
                name: example-service
                port:
                  number: 80 # 여는 포트
```

### 3. 단일 호스트에 대한 다중 경로 라우팅

단일 호스트에 대해 여러 서비스로 경로 기반 라우팅을 설정합니다. -> 우리 Next앱일때 요런 방식이었다고 할 수 있음

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-path-ingress
spec:
  rules:
    - host: "example.com"
      http:
        paths:
          - path: /service1
            pathType: Prefix
            backend:
              service:
                name: service1
                port:
                  number: 80
          - path: /service2
            pathType: Prefix
            backend:
              service:
                name: service2
                port:
                  number: 80
```

### 4. SSL/TLS를 사용한 보안 통신

SSL/TLS 인증서를 사용하여 HTTPS 통신을 활성화하는 설정입니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
    - hosts:
        - "example.com"
      secretName: example-tls-secret
  rules:
    - host: "example.com"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example-service
                port:
                  number: 80
```

이 예에서 `example-tls-secret`은 미리 생성된 TLS 시크릿이어야 합니다.

### 5. 임의의 HTTP 헤더 추가

Ingress에 임의의 HTTP 헤더를 추가할 수 있습니다 (이 기능은 사용 중인 Ingress 컨트롤러에 따라 다릅니다). 예를 들어, NGINX Ingress 컨트롤러를 사용하는 경우, 다음과 같이 설정할 수 있습니다:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: custom-headers-ingress
  annotations:
    nginx.ingress.kubernetes.io/add-headers: |
      X-Custom-Header: "custom value"
spec:
  rules:
    - host: "example.com"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example-service
                port:
                  number: 80
```

이러한 설정은 다양한 요구 사항과 환경에 따라 유용하게 활용될 수 있습니다.

## 의문점 해소

### 서비스 라우팅 설정을 어디서 하느냐? 가 무슨 차이인가?

클라우드 앞단에서도 할 수 있지만(cloudFront, Route53) ingress로 하는게 이점이 있는 경우도 있음

1. **클라우드 독립적**: Ingress는 클라우드 공급자에 구애받지 않고, 어디서든 동일한 방식으로 작동합니다. 이로 인해 멀티 클라우드 환경에서의 이식성이 높습니다.
2. **YAML 기반의 선언적 설정**: Kubernetes와 같은 선언적 접근 방식을 사용하여, 소스 코드와 함께 인프라를 버전 관리 할 수 있습니다.
3. **파드 및 서비스와의 높은 통합도**: Ingress는 Kubernetes 클러스터 내의 파드와 서비스와 자연스럽게 통합되므로, 내부 서비스 라우팅을 손쉽게 구성할 수 있습니다.
4. **세세한 트래픽 분배 규칙**: Ingress는 호스트 및 경로 기반 라우팅 뿐만 아니라, 다양한 라우팅 규칙과 어노테이션을 통한 더욱 세밀한 트래픽 제어가 가능합니다.

## 실습

- signature-backend로 각각 다른 고유하고 배타적인 엔드포인트를 가진 이미지 2개를 굽습니다
- 각 이미지를 각 서비스에 연결하고, ingress로 basepath에 따라 다른 서비스로 라우팅을 합니다
- 단일 엔드포인트에 다른 path로 요청했을 때 올바른 pod으로 요청이 가는지 확인해봅니다
- minikube에서 ingress를 로컬에 띄우려면
  - 에드온 설치 : `minikube addons enable ingress`
  - IP 확인 : `minikube ip`
  - `/etc/hosts`: 파일 수정 `192.168.99.100 jonghyuk.flex.com`
  - `kubectl describe ing my-ingress` 로 인그레스 연결 상태 확인
- 여기까지 하고 브라우저로 응답 확인 `jonghyuk.flex.com/jonghyuk`, `jonghyuk.flex.com/max`
