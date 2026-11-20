# katsura-agent

Linux 노드마다 하나의 Pod에서 `lldpd`와 Go 수집기를 실행한다. 두 컨테이너는 [values.yaml](values.yaml)의 `image.repository`와 `image.tag`로 지정한 이미지를 공유한다. 이미지를 비공개로 운영한다면 설치 네임스페이스의 레지스트리 Secret을 `imagePullSecrets`에 지정한다.

## 설치

저장소 루트에서 다음과 같이 설치한다. 배치 전에 대상 노드에 기존 `lldpd`나 동일한 DaemonSet이 없는지 확인한다.

```sh
helm upgrade --install katsura-agent ./charts/katsura-agent \
  --namespace katsura-system --create-namespace
```

## 노드와 인터페이스 선택

기본 `nodeSelector: []`는 라벨 제한을 두지 않는다. Linux/amd64 이미지를 혼합 클러스터에 배포한다면 아래처럼 `my-values.yaml`에 대상을 지정하고 설치 명령에 `-f my-values.yaml`을 추가한다.

```yaml
nodeSelector:
  kubernetes.io/os: linux
  kubernetes.io/arch: amd64
```

`hostNetwork: true`는 호스트 NIC를 관측하기 위한 기본값이다. `interfaceName`을 비우면 `en*`를 탐색하고, `eth0` 등 정확한 이름을 주면 해당 인터페이스만 관측한다. 수집 주기, 보안 컨텍스트, 리소스 등 전체 설정은 [values.yaml](values.yaml)을 참고한다.
