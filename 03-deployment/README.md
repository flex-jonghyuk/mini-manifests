# Deployment

Kubernetes Deployment는 애플리케이션의 상태를 관리하는데 도움을 주는 리소스입니다. Deployment는 특정 개수의 Replica를 유지하며, Replica는 애플리케이션의 복사본을 나타냅니다. 만약 Pod가 실패하거나 삭제되면, Deployment는 자동으로 새로운 Pod를 생성하여 Replica의 개수를 일정하게 유지합니다.

Deployment는 일관된 업데이트와 롤백을 제공하며, 이는 애플리케이션의 버전을 안전하게 및 그리고 효과적으로 업데이트하기 위해 사용될 수 있습니다. 만약 Deployment가 중단되거나 실패한다면, Deployment는 자동으로 롤백하여 안정적인 상태를 유지합니다.

## Deployment와 ReplicaSet의 관계

Deployment가 ReplicaSet을 자동으로 관리하고 생성합니다. 즉, Deployment는 ReplicaSet에 기반한 기능을 추가적으로 제공하는, 더 높은 수준의 개념입니다.

ReplicaSet은 지정된 수의 Pod를 유지하도록 보장하는 역할을 합니다. 즉, ReplicaSet은 단순히 지정된 수의 Pod가 동작하고 있음을 보장합니다.

Deployment는 이보다 더 많은 기능을 제공합니다. Deployment는 애플리케이션의 새 버전을 안전하게 롤아웃하거나, 예전 버전으로 롤백하는 등의 기능을 제공합니다. 이런 기능은 ReplicaSet만으로는 수행할 수 없습니다.

따라서, Deployment를 사용하면 자동으로 ReplicaSet이 생성되며, 이 ReplicaSet은 Deployment 내에서 정의한 대로 Pod를 관리합니다. 이 과정은 사용자에게는 투명하게 이루어지며, 사용자는 보통 Deployment 수준에서 작업을 관리하게 됩니다.

간단히 말하면, Deployment는 사용자에게 ReplicaSet를 직접 관리하는 수고를 덜어주고, 대신 더 높은 수준에서의 관리를 가능하게 합니다.

대부분의 경우에는 Deployment를 사용하여 ReplicaSet을 관리하는 것이 가장 좋습니다. Deployment는 선언적 업데이트, 롤아웃 및 롤백 기능, 스케일링 등의 추가적인 기능을 제공하기 때문입니다.

그러나 ReplicaSet을 직접 사용하는 일부 특별한 경우가 있을 수 있습니다. 예를 들어, 특정한 사용자 정의 롤아웃이나 롤백 로직을 구현하려는 경우에는 직접 ReplicaSet을 관리할 수 있습니다.

그러나 일반적으로는 Deployment를 사용하는 것이 좋습니다. Deployment는 ReplicaSet 위에 구축된 추상화 레이어로서, 사용자가 애플리케이션의 선언적 업데이트를 쉽게 관리할 수 있도록 도와줍니다. 또한, Deployment는 상태를 관리하고 장애 조치를 수행하는 데 도움이 됩니다. 따라서 대부분의 경우에는 Deployment를 사용하여 ReplicaSet을 관리하는 것이 가장 효과적이고 효율적인 방법입니다.

## 옵션

Kubernetes Deployment에서 설정 가능한 주요 옵션들은 다음과 같습니다:

1. `apiVersion`: 현재 사용 중인 Kubernetes API의 버전입니다. Deployment에 대한 기본 값은 "apps/v1"입니다.

2. `kind`: Kubernetes 리소스의 유형을 나타냅니다. Deployment를 생성할 때는 "Deployment"로 설정됩니다.

3. `metadata`: 리소스에 대한 메타데이터를 포함합니다. 이는 리소스의 이름, 네임스페이스, 라벨, 주석 등을 포함할 수 있습니다.

   - `name`: Deployment의 이름을 나타냅니다.
   - `labels`: Deployment에 적용되는 라벨을 나타냅니다. 라벨은 리소스를 식별하고 선택하는 데 사용됩니다.
   - `namespace`: Deployment가 생성되는 네임스페이스를 지정합니다. 이 필드가 없으면 기본 네임스페이스에 Deployment가 생성됩니다.
   - `annotations`: 사용자가 임의로 정의할 수 있는 키-값 쌍입니다. 이 정보는 Deployment를 쿼리하고 검사하는 데 도움이 될 수 있습니다.

4. `spec`: Deployment의 동작을 정의하는 스펙을 포함합니다.

   - `replicas`: Pod의 복제본 수를 나타냅니다. 이 수는 스케일링 이벤트 또는 Pod 장애에 따라 자동으로 조정될 수 있습니다.
   - `selector`: 이 선택자는 Deployment가 관리할 Pod를 결정합니다. 이 필드의 라벨은 Pod 템플릿 내의 메타데이터 라벨과 일치해야 합니다.
   - `template`: Pod 템플릿은 신규로 생성되는 Pod의 모양을 정의합니다. 이에는 라벨, 주석, 컨테이너 스펙 등이 포함됩니다.
   - `minReadySeconds`: 새로운 Replica가 최소 몇 초 동안 Ready 상태로 유지되어야 새로운 업데이트가 발생할 수 있는지를 나타냅니다.
   - `strategy`: Deployment 업데이트 전략을 정의합니다. 두 가지 유형이 있습니다: `RollingUpdate`와 `Recreate`. `RollingUpdate`는 기본 전략이며, 이는 모든 Pod를 동시에 종료하지 않고 순차적으로 업데이트합니다. `Recreate`는 모든 기존 Pod를 종료하고 새로운 Pod를 생성합니다.

5. `status`: Deployment의 현재 상태를 나타냅니다. 이 필드는 읽기 전용이며, Kubernetes 시스템에 의해 자동으로 설정됩니다.

위 내용은 기본적인 설정 옵션에 대한 설명입니다. 자세한 내용과 추가 옵션에 대해서는 Kubernetes 공식 문서를 참고하는 것이 좋습니다.

### Strategy

Deployment의 업데이트 전략(`strategy`)에는 크게 `RollingUpdate`와 `Recreate` 두 가지가 있습니다. 각 전략은 서로 다른 동작을 수행하고 있으며, 사용 사례에 따라 적절한 전략을 선택할 수 있습니다.

1. `RollingUpdate`: 이 전략은 기본 전략으로서, 모든 Pod를 동시에 종료하지 않고 순차적으로 업데이트합니다. 이를 통해 서비스 중단을 최소화하면서 새 버전의 애플리케이션으로 점진적으로 업데이트할 수 있습니다. `RollingUpdate` 전략에는 두 가지 추가적인 필드가 있습니다:

   - `maxUnavailable`: 롤링 업데이트 중에 사용할 수 없는 최대 Pod의 수 또는 비율을 나타냅니다. 이 값은 절대 수 또는 비율로 설정할 수 있으며, 기본값은 `25%`입니다.
   - `maxSurge`: 롤링 업데이트 중에 초과 생성될 수 있는 최대 Pod의 수 또는 비율을 나타냅니다. 이 값도 절대 수 또는 비율로 설정할 수 있으며, 기본값은 `25%`입니다.

   RollingUpdate 전략을 사용하면 쿠버네티스는 새로운 버전의 팟을 하나씩 띄우고, 동시에 이전 버전의 팟을 하나씩 종료합니다. 이렇게 함으로써 새로운 버전으로 점진적으로 전환되는 동안에도 서비스가 계속 사용 가능하게 유지됩니다.

2. `Recreate`: 이 전략은 모든 기존 Pod를 종료한 후 새로운 Pod를 생성합니다. 이는 신규 버전의 애플리케이션이 이전 버전과 함께 실행되면 문제가 생기는 경우에 유용할 수 있습니다. 다만, 이 방식은 기존 Pod를 종료하고 새 Pod가 시작될 때까지 서비스 중단이 발생할 수 있습니다.

`Recreate` 전략은 기존 버전의 애플리케이션과 새 버전의 애플리케이션이 동시에 실행되면 안 되는 경우에 사용하기 적합합니다. 예를 들어, 데이터베이스 스키마 업그레이드, 기존 버전과 새 버전의 애플리케이션이 서로 호환되지 않는 경우 등이 이에 해당합니다.

`RollingUpdate`는 배포 중에 서비스를 계속 사용할 수 있어야 하는 경우에 사용하기 좋습니다. 예를 들어, 웹 서버, API 서버 등이 이에 해당합니다.

### maxUnavilable, maxSurge

예를 들어 maxUnavailable이 1이고 maxSurge가 1인 경우, 쿠버네티스는 먼저 새 버전의 팟 하나를 띄우고, 이 팟이 정상적으로 시작되면 이전 버전의 팟 하나를 종료합니다. 그리고 이 프로세스를 모든 팟이 새 버전으로 전환될 때까지 반복합니다.

### minReadySeconds

`minReadySeconds`는 Kubernetes의 Deployment 설정에서 사용할 수 있는 옵션으로, Pod가 준비(Ready) 상태가 된 후에도 새로운 업데이트가 발생하기 전에 기다려야 하는 최소 시간을 초 단위로 지정합니다. 이 값은 Deployment의 `.spec.minReadySeconds` 필드에서 설정할 수 있습니다.

예를 들어, `minReadySeconds`를 10으로 설정하면, 각 Pod가 시작된 후 10초 동안은 새로운 업데이트가 발생하지 않습니다. 이 기간 동안에 Pod는 준비(Ready) 상태를 유지하고 트래픽을 수신할 수 있습니다.

`minReadySeconds` 설정은 새로운 버전의 애플리케이션을 롤아웃하는 동안 서비스의 가용성을 높이는 데 도움이 될 수 있습니다. 새로운 버전의 Pod가 준비 상태가 되면, 이전 버전의 Pod는 즉시 종료되지 않고 `minReadySeconds` 동안 실행을 계속합니다. 이렇게 하면 롤아웃 도중에 일시적으로 사용 가능한 Pod의 수가 늘어나므로, 서비스 중단을 최소화할 수 있습니다.

그러나 `minReadySeconds` 설정은 롤아웃의 속도를 늦출 수 있으므로, 이를 설정할 때는 애플리케이션의 요구 사항과 가용성, 롤아웃 속도 등을 고려해야 합니다.

### status

Deployment의 `status` 필드는 Deployment의 현재 상태를 나타내며 읽기 전용입니다. 즉, 사용자가 직접 변경할 수 없는 상태 정보입니다.

`status` 필드는 다음과 같은 하위 필드를 포함합니다:

- `observedGeneration`: 마지막으로 처리된 Deployment의 세대(generation)를 나타냅니다. 이 값은 사용자가 변경하려는 대상과 현재 시스템의 상태를 비교하는 데 사용됩니다.
- `replicas`: Deployment에서 실행 중인 ReplicaSet의 Pod 수를 나타냅니다.
- `updatedReplicas`: 새로운 Pod Template을 가진 Deployment에서 실행 중인 ReplicaSet의 Pod 수를 나타냅니다.
- `readyReplicas`: 사용자의 데이터를 제공하고 있는 Pod의 수를 나타냅니다.
- `availableReplicas`: 사용자의 데이터를 제공할 수 있는 Pod의 수를 나타냅니다.
- `unavailableReplicas`: 사용자의 데이터를 제공할 수 없는 Pod의 수를 나타냅니다.
- `conditions`: Deployment의 현재 상태를 나타내는 최근 상태 전이를 저장합니다. 예를 들어, Deployment의 롤아웃이 진행 중이거나 완료되었는지 등의 정보를 제공합니다.

이러한 정보를 통해 Deployment의 상태를 파악하고, 필요한 경우 이를 사용하여 애플리케이션 로직에 반영할 수 있습니다.

## operation

Kubernetes의 `kubectl` 명령어는 Deployment와 같은 리소스를 관리하는 데 여러 가지 명령을 제공합니다. 아래에 일반적인 `kubectl` 명령어 중 일부를 소개합니다:

1. `kubectl get deployments`: 클러스터 내의 모든 Deployment를 조회합니다.

2. `kubectl describe deployment <deployment-name>`: 특정 Deployment에 대한 상세 정보를 출력합니다. 이 정보에는 이벤트, ReplicaSet, Pod 정보 등이 포함됩니다.

3. `kubectl apply -f <file-name.yaml>`: 주어진 YAML 파일로 Deployment를 생성하거나 업데이트합니다.

4. `kubectl delete deployment <deployment-name>`: 특정 Deployment를 삭제합니다.

5. `kubectl rollout status deployment/<deployment-name>`: 특정 Deployment의 롤아웃 상태를 출력합니다. 이는 Deployment가 업데이트 중인지, 완료되었는지 등의 정보를 제공합니다.

6. `kubectl rollout history deployment/<deployment-name>`: 특정 Deployment의 업데이트 히스토리를 출력합니다.

7. `kubectl rollout undo deployment/<deployment-name>`: 특정 Deployment의 마지막 업데이트를 되돌립니다.

8. `kubectl rollout restart deployment/<deployment-name>`: 특정 Deployment를 재시작합니다. 이 명령어는 Deployment의 모든 Pod를 새로 시작합니다.

9. `kubectl scale deployment/<deployment-name> --replicas=<number>`: 특정 Deployment의 ReplicaSet 크기를 조절합니다. 이 명령어를 사용하면 실행 중인 Pod의 수를 증가시키거나 감소시킬 수 있습니다.

이 명령어들은 기본적인 Deployment 관리에 필요한 명령어들이며, 다양한 환경 및 요구 사항에 따라 추가적인 `kubectl` 명령어들이 사용될 수 있습니다.

## [챔고] Pod 상태 용어

쿠버네티스에서 Pod의 상태를 설명하는 여러 가지 용어가 있습니다. Pod는 그 생명주기 동안 다양한 상태를 가질 수 있으며, 아래는 주요한 상태들에 대한 설명입니다:

1. `Pending`: Pod가 쿠버네티스 시스템에 의해 승인되었지만, 하나 이상의 컨테이너 이미지가 아직 생성되지 않아 실행을 위한 준비가 완료되지 않은 상태를 나타냅니다.

2. `Running`: Pod가 노드에 바인딩되었고, 모든 컨테이너가 생성되었으며, 적어도 하나 이상의 컨테이너가 동작 중이거나 시작 또는 재시작 중인 상태를 나타냅니다.

3. `Succeeded`: Pod 내의 모든 컨테이너가 성공적으로 종료되었고, 더 이상 재시작하지 않을 상태를 나타냅니다. 일회성 작업에서 이 상태를 주로 보게 됩니다.

4. `Failed`: Pod 내의 모든 컨테이너가 종료되었고, 적어도 하나 이상의 컨테이너가 실패로 종료된 상태를 나타냅니다. 즉, 컨테이너는 비-제로 상태로 종료되었거나 시스템에 의해 종료되었습니다.

5. `Unknown`: 어떠한 이유로 인해 Pod의 상태를 알 수 없는 상태를 나타냅니다. 일반적으로 Pod에 대한 정보를 가져오는 데 문제가 발생한 경우에 발생합니다.

## 실습

- deployment와 replicaset, pod을 만들어본다
- 팟을 지워본다 `kubectl delete -n default pod my-deployment-67494f9f57-257lx`
- restart를 통해 rolling update, recreate의 양상을 비교해본다. 롤아웃 중에 status 명령어로 현재 어떤 상태인지 봐보자 `kubectl rollout restart deployment/my-deployment && kubectl rollout status deployment/my-deployment`
- 이미지가 바뀌었을 때 양상이 어떤지 봐보자 `kubectl apply -f my-deployment-new-image.yaml && kubectl rollout status deployment/my-deployment`
- 실패했을때의 양상(없는 태그를 가져온다거나 할때) 양상이 어떤지 봐보자
