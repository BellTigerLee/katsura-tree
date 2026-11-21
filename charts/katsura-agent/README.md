# katsura-agent

Linux 노드마다 하나의 Pod에서 `lldpd`와 Go 수집기를 실행한다. 두 컨테이너는 [values.yaml](values.yaml)의 `image.repository`와 `image.tag`로 지정한 이미지를 공유한다. 이미지를 비공개로 운영한다면 설치 네임스페이스의 레지스트리 Secret을 `imagePullSecrets`에 지정한다.

## 설치

저장소 루트에서 다음과 같이 설치한다. 배치 전에 대상 노드에 기존 `lldpd`나 동일한 DaemonSet이 없는지 확인한다.

```sh
helm upgrade --install katsura-agent ./charts/katsura-agent \
  --namespace katsura-system --create-namespace
```

## 노드와 인터페이스 선택

`image.tag`를 비워 두면 차트 `appVersion`(`0.2.0`)을 쓴다. 아래 예시는 설치 없이 렌더링만 한다.

```sh
helm template katsura-agent ./charts/katsura-agent \
  --set-string image.repository=belltigerlee/katsura-agent \
  --set-string image.tag=0.2.0 \
  --set-string collector.mode=grpc \
  --set-string collector.grpcListen=127.0.0.1:9579
```

기본 모드 `grpc`에서 주기는 요청자인 lumber가 정한다. 기본 리스너는 로컬 전용이다. 원격에서는 인증된 터널 또는 접근이 제한된 주소를 사용한다. `hostNetwork: true`에서 `0.0.0.0`은 인증 없는 리스너를 모든 노드 인터페이스에 노출한다. 기존 폴링 동작은 `--set collector.mode=poll-lldp --set collector.interval=30s`로 선택한다(0.2.0 바이너리 기준).

기본 `nodeSelector: []`는 라벨 제한을 두지 않는다. Linux/amd64 이미지를 혼합 클러스터에 배포한다면 아래처럼 `my-values.yaml`에 대상을 지정하고 설치 명령에 `-f my-values.yaml`을 추가한다.

```yaml
nodeSelector:
  kubernetes.io/os: linux
  kubernetes.io/arch: amd64
```

`hostNetwork: true`는 호스트 NIC를 관측하기 위한 기본값이다. `interfaceName`을 비우면 LLDP는 `en*`, gRPC의 sysfs·IP 원본은 모든 인터페이스가 대상이다. 단일 NIC는 `--set-string interfaceName=eth0`로 지정한다. 전체 설정은 [values.yaml](values.yaml)을 참고한다.
