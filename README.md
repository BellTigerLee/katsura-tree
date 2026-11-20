# Katsura Helm Charts

Katsura 구성 요소의 배포용 Helm 차트를 모아 둔다. 각 차트는 `charts/<차트 이름>/`에서 독립적으로 버전을 관리하고 설치한다. 애플리케이션 코드와 Docker 이미지는 이 저장소에서 빌드하지 않는다.

| 차트 | 용도 |
| --- | --- |
| [katsura-agent](charts/katsura-agent/) | Linux 노드의 LLDP 이웃을 수집하는 DaemonSet |

## 차트 검증

저장소 루트에서 실행한다. 클러스터를 변경하지 않는다.

```sh
helm lint charts/katsura-agent --strict
helm template katsura-agent charts/katsura-agent --namespace katsura-system
```

## Helm 저장소 배포

`charts/`는 소스 디렉터리다. `helm repo add`를 지원하려면 별도 배포 위치(예: GitHub Pages)에서 `index.yaml`과 각 버전의 차트 패키지(`.tgz`)를 HTTPS로 제공해야 한다. Git 저장소 URL 자체는 Helm 저장소 주소가 아니다. 이미 배포된 버전은 그대로 두고, 차트 변경을 배포할 때는 해당 `Chart.yaml`의 `version`을 올린다.
