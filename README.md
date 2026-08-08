# CKAD 공부

ckad를 취득하기 위해 공부한 내용을 정리합니다.

섹션 1은 introduction이므로 섹션 2부터 정리합니다.

## 주제별 바로가기

### Section 2. Core Concepts
- [1. Kubernetes Architecture](<ckad/section2/md/1. kubernetes_architecture.md>) — 노드/클러스터, 컨트롤 플레인 구성요소
- [2. Docker vs ContainerD](<ckad/section2/md/2. docker_vs_containerD.md>) — 컨테이너 런타임과 CRI
- [3. Pods](<ckad/section2/md/3. pods.md>) — 쿠버네티스의 최소 배포 단위
- [4. ReplicaSet](<ckad/section2/md/4. replicaset.md>) — 복제본 관리와 셀렉터
- [5. Deployment](<ckad/section2/md/5. Deployment.md>) — 선언적 배포 관리
- [6. Namespace](<ckad/section2/md/6. namespace.md>) — 클러스터 논리 분리
- [7. Service](<ckad/section2/md/7.service.md>) — ClusterIP / NodePort / LoadBalancer

### Section 3. Configuration
- [1. 컨테이너 이미지 정의·빌드·수정](<ckad/section3/md/1.define_build_modify_container_images.md>)
- [2. Command & Arguments](<ckad/section3/md/2.command_and_arguments_in_docker_kubernetes.md>) — ENTRYPOINT/CMD와 command/args
- [3. ConfigMap](<ckad/section3/md/3.configmap.md>) — 설정 값 주입
- [4. Secret](<ckad/section3/md/4.secret.md>) — 민감 정보 관리
- [5. Resource Requirements](<ckad/section3/md/5.resources_requirements.md>) — requests/limits
- [6. Service Account](<ckad/section3/md/6.service_account.md>) — 파드의 API 인증 주체
- [7. Taints & Tolerations](<ckad/section3/md/7.taints_and_tolerations.md>) — 노드가 파드를 밀어내는 방식
- [8. Node Selector & Affinity](<ckad/section3/md/8.node_selector_and_affinity.md>) — 파드를 특정 노드로 보내는 방식

### Section 4. Multi-Container Pods
- [1. Multi-Container Pods](<ckad/section4/md/1.multi_container_pods.md>) — 사이드카/앰배서더/어댑터 패턴
- [2. Init Containers](<ckad/section4/md/2. init_containers.md>) — 본 컨테이너 실행 전 초기화

### Section 5. Observability
- [1. Readiness & Liveness Probe](<ckad/section5/md/1.Readiness&Liveness_Probe.md>) — 헬스체크
- [2. Logging](<ckad/section5/md/2.Logging.md>) — 로그 확인
- [3. Monitoring](<ckad/section5/md/3.monitoring.md>) — 메트릭 수집

### Section 6. Pod Design
- [1. Labels, Selectors and Annotations](<ckad/section6/md/1. Labels, Selectors and Annotations.md>)
- [2. Rolling Updates & Rollbacks](<ckad/section6/md/2. Rolling Updates & Rollbacks in Deployments.md>)
- [3. Blue-Green & Canary Deployments](<ckad/section6/md/3. Blue-Green & Canary Deployments.md>)
- [4. Jobs & CronJobs](<ckad/section6/md/4. Jobs & CronJobs.md>)

### Section 7. Services & Networking
- [1. Network Policies](<ckad/section7/md/1. Network Policies.md>) — 파드 간 트래픽 제어
- [2. Ingress](<ckad/section7/md/2. Ingress.md>) — L7 라우팅

### Section 8. State Persistence
- [1. Volumes in Kubernetes](<ckad/section8/1. Volumes in Kubernetes.md>) — 볼륨 기본
- [2. Persistent Volume & Claims](<ckad/section8/2. Persistent Volume & Claims.md>) — PV / PVC / StorageClass

## 이런 게 궁금하다면

| 궁금한 점 | 문서 |
| --- | --- |
| 쿠버네티스는 어떻게 구성돼 있나? | [Kubernetes Architecture](<ckad/section2/md/1. kubernetes_architecture.md>) |
| 앱을 어떻게 배포하고 업데이트하나? | [Deployment](<ckad/section2/md/5. Deployment.md>), [Rolling Updates & Rollbacks](<ckad/section6/md/2. Rolling Updates & Rollbacks in Deployments.md>) |
| 외부에서 내 앱에 어떻게 접근하나? | [Service](<ckad/section2/md/7.service.md>), [Ingress](<ckad/section7/md/2. Ingress.md>) |
| 설정값·비밀번호는 어디에 두나? | [ConfigMap](<ckad/section3/md/3.configmap.md>), [Secret](<ckad/section3/md/4.secret.md>) |
| 파드를 원하는 노드에 배치하려면? | [Node Selector & Affinity](<ckad/section3/md/8.node_selector_and_affinity.md>), [Taints & Tolerations](<ckad/section3/md/7.taints_and_tolerations.md>) |
| 앱이 죽었는지 어떻게 아나? | [Readiness & Liveness Probe](<ckad/section5/md/1.Readiness&Liveness_Probe.md>) |
| 데이터를 영구 저장하려면? | [Volumes](<ckad/section8/1. Volumes in Kubernetes.md>), [PV & PVC](<ckad/section8/2. Persistent Volume & Claims.md>) |
| 배치 작업·정기 작업은? | [Jobs & CronJobs](<ckad/section6/md/4. Jobs & CronJobs.md>) |
| 파드 간 통신을 막으려면? | [Network Policies](<ckad/section7/md/1. Network Policies.md>) |

## 저장소 구조

```
ckad/
└── section2~8/
    ├── md/        # 개념 정리 문서
    └── activity/  # 실습 매니페스트 (section2~4)
```

> 문서는 모두 한국어로 작성되어 있으며, 개념 설명 + 예시 YAML + kubectl 명령어 중심으로 정리했습니다.
