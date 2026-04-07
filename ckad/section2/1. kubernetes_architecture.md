# Kubernetes Architecture

### 1. 노드(Node)와 클러스터(Cluster): 내 앱이 사는 동네

내 앱이 배치될 “물리적 공간”에 대해 이해하자.

- 노드(Node): 쿠버네티스가 설치된 개별 머신(가상 머신 또는 물리 서버)이다.
- 클러스터(Cluster): 노드 여러 개를 하나로 묶은 것
    - 왜 묶을까? 한 노드가 죽어도 다른 노드에서 앱을 계속 돌려야 하기 때문이다. for High Availability

### 2. 마스터 노드 vs 워커 노드: 관리자와 일꾼

클러스터 안에서 역할 분담은 확실하다.

#### 마스터 노드 (control plane): 클러스터의 두뇌

클러스터 전체를 관리하고 결정한다. 이 구성 요소들이 어떤 일을 하는지 명칭을 익혀두자.

- API Server: 클러스터의 유일한 입구. kubectl로 내리는 명령은 모두 여기로 전달된다.
- etcd: 클러스터의 Key-Value DB. 어떤 노드에 어떤 앱이 있는지 모든 정보가 저장된다.
- Scheduler: 배정 담당자. 새로 만든 앱(컨테이너)을 리소스가 넉넉한 노드에 배치한다.
- Controller Manager: 상태 유지 관리자. 노드가 죽거나 컨테이너가 꺼지면 즉시 인지하고 대응한다.

#### 워커 노드 (Worker Node): 실제 현장

마스터의 지시를 받아 실제로 컨테이너를 실행한다.

- kubelet: 현장 소장. 마스터의 지시를 받아 컨테이너를 띄우고, 잘 돌아가는지 보고한다.
- Container Runtime: 컨테이너를 실행하는 엔진(Docker, containerD 등)

#### 개발자의 도구 `kubectl`

지금부터 익혀야 할 기본 명령어

1. 앱 배포하기: `kubectl run <이름> --image=<이미지>`
2. 클러스터 상태 확인: `kubectl cluster-info`
3. 내 일꾼 목록 보기: `kubectl get nodes`

#### 아키텍처의 흐름으로 기억할 것

내가 `kubectl` 로 명령을 보낸다 → API Server에서 명령을 받는다 → Scheduler가 적절한 노드를 정한다 → 해당 노드의 kubelet이 컨테이너를 띄운다