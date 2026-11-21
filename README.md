# Katsura Helm Charts

Katsura 구성 요소의 배포용 Helm 차트를 모아 둔다. 각 차트는 `charts/<차트 이름>/`에서 독립적으로 버전을 관리하고 설치한다. 애플리케이션 코드와 Docker 이미지는 이 저장소에서 빌드하지 않는다.

| 차트 | 용도 |
| --- | --- |
| [katsura-agent](charts/katsura-agent/) | Linux 노드의 LLDP 이웃을 수집하는 DaemonSet |

## Helm 저장소 등록 및 설치

GitHub Pages에 차트가 게시된 후 사용할 수 있다. Git 저장소를 복제할 필요는 없다.

```sh
helm repo add katsura https://belltigerlee.github.io/katsura-tree
helm repo update katsura
helm search repo katsura/katsura-agent --versions
```

다음 명령은 현재 Kubernetes 컨텍스트의 `katsura-system` 네임스페이스에 에이전트를 설치한다. 같은 이름의 릴리스가 이미 있으면 지정한 버전으로 업데이트한다.

```sh
helm upgrade --install katsura-agent katsura/katsura-agent \
  --version 0.2.0 --namespace katsura-system --create-namespace
```

클러스터에 설치하지 않고 차트만 내려받으려면 게시 후 `helm pull katsura/katsura-agent --version 0.2.0`을 실행한다.

## 차트 검증

저장소 루트에서 실행한다. 클러스터를 변경하지 않는다.

```sh
helm lint charts/katsura-agent --strict
helm template katsura-agent charts/katsura-agent --namespace katsura-system
```

## Helm 저장소 배포

### 관리자 게시 절차

**이미지는 GHCR, Helm 차트는 GitHub Pages (`main` 브랜치의 `/docs`)에 게시한다.** 클러스터 설치와는 별개다. 차트를 먼저 올리면 노드가 아직 없는 이미지를 pull할 수 있으므로 이미지부터 확인한다.

**1. 실행 이미지 게시** (Bash). 로컬 이미지가 없을 때만 agent 저장소에서 `./hack/build-image.sh --tag 0.2.0`을 실행한다. 로그인 시 Password 프롬프트에는 `write:packages` 권한의 GitHub PAT(classic)를 입력한다. 토큰을 명령줄이나 Git 파일에 쓰지 않는다.

```bash
V=0.2.0
IMAGE=ghcr.io/belltigerlee/katsura-agent:$V
docker login ghcr.io -u BellTigerLee
docker tag belltigerlee/katsura-agent:$V "$IMAGE"
docker push "$IMAGE"
docker buildx imagetools inspect "$IMAGE"
```

이미지 공개 여부와 클러스터의 pull 권한도 확인한다. 비공개로 유지하면 설치 네임스페이스에 `read:packages` 권한의 `imagePullSecrets`가 필요하다. 배포 이미지가 나온 소스 커밋을 별도로 기록하고, 이미 배포한 버전에 다른 내용을 덮어쓰지 않는다.

**2. 차트 게시** (`katsura-tree` 저장소 루트). 새 버전은 `Chart.yaml`의 `version`(차트)과 `appVersion`(기본 이미지 태그)을 확인하고 `helm lint charts/katsura-agent --strict`, `helm package charts/katsura-agent -d docs`, `helm repo index docs --url https://belltigerlee.github.io/katsura-tree` 순서로 준비한다. 기존 버전 `.tgz`는 유지한다. **이번 0.2.0은 이미 패키지와 인덱스가 준비되어 있으므로** 아래부터 실행한다.

```bash
cd /home/work8/workspace/katsura-tree
V=0.2.0
helm show chart "docs/katsura-agent-$V.tgz"
git add -- README.md charts/katsura-agent docs/index.yaml "docs/katsura-agent-$V.tgz"
git diff --cached --stat
# 검토한 변경만 있는지 확인한 뒤:
git commit -m "Publish katsura-agent chart $V"
git push origin main
```

**3. Pages 배포 완료 후 패키지 다운로드 확인** (클러스터 변경 없음):

```bash
helm repo add katsura https://belltigerlee.github.io/katsura-tree --force-update
helm repo update katsura
helm pull katsura/katsura-agent --version 0.2.0 --destination "$(mktemp -d)"
```

`charts/`는 소스 디렉터리다. `helm repo add`를 지원하려면 별도 배포 위치(예: GitHub Pages)에서 `index.yaml`과 각 버전의 차트 패키지(`.tgz`)를 HTTPS로 제공해야 한다. Git 저장소 URL 자체는 Helm 저장소 주소가 아니다. 이미 배포된 버전은 그대로 두고, 차트 변경을 배포할 때는 해당 `Chart.yaml`의 `version`을 올린다.
