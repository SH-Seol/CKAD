# docker vs containerD

이전에는 Docker가 컨테이너 시장을 지배했기에 K8s도 docker를 사용했지만, K8s의 수요가 늘면서 표준화된 인터페이스가 필요해졌다.

1. 주요 개념 및 표준
    - OCI (Open Container Initiative): 컨테이너 이미지와 런타임에 대한 국제 표준이며 덕분에 Docker로 만든 이미지를 다른 런타임(containerD)에서도 쓸 수 있다.
    - CRI (Container Runtime Interface): k8s가 다양한 컨테이너 엔진과 소통하기 위해 만든 표준 규격이다.
    - Docker Shim: Docker는 CRI가 나오기 전에 만들어졌기 때문에 k8s와 연결하기 위해 임시로 만든 다리였으나 1.24 이후로 사라져서 Docker를 직접 지원하지 않게 되었다.
2. ContainerD의 등장
    - Docker의 핵심구성 요소 중 하나였지만 현재는 CNCF의 독립된 프로젝트
    - 쿠버네티스에서는 Docker 전체를 설치할 필요 없이, 컨테이너 실행에 꼭 필요한 containerD만 런타임으로 사용한다.

---

#### 컨테이너 관리 CLI 도구 비교

Docker를 대신해 컨테이너 관리하고 디버깅할 때 사용하는 세가지 도구를 알아두자.

| 도구명 | 관리 주체 | 주요 용도 | 특징 |
| --- | --- | --- | --- |
| ctr | containerD 커뮤니티 | 내부 디버깅 | 기능이 제한적이고 복잡함. 잘 안쓰임. |
| nerdctl | containerD 커뮤니티 | 일반 컨테이너 관리 | Docker와 거의 동일, docker 명령어를 그대로 쓰면서 추가 기능(p2p, 암호화) 제공 |
| crictl | Kubernetes 커뮤니티 | k8s 노드 디버깅 | 모든 cri 호환 런타임에서 작동하며 pod단위 조회 가능 |

---

#### 컨테이너 실행 (ctr vs nerdctl)

ctr
```bash
ctr images pull docker.io/library/redis:alpine
ctr run docker.io/library/redis:alpine redis
```

nerdctl
```bash
nerdctl run --name redis redis:alpine
nerdctl run --name webserver -p 80:80 -d nginx
```
---
#### `crictl` 을 사용하자.

시험 환경이나 워커 노드 장애 조치 시 `crictl` 은 Docker와 유사한 문법을 제공하여 편하다.

- 이미지 확인: `crictl images`
- 실행 중인 컨테이너 확인: `crictl ps -a`
- 로그 확인: `crictl logs <컨테이너 id>`
- 포드 목록 확인: `crictl pods`
  
예시

```bash
$ crictl pull busybox
$ crictl images
$ crictl ps -a
```

주의사항

1.24 이후로 docker 지원을 안해서 `crictl` 사용하려면 엔드포인트 지정해야한다.

```bash
crictl --runtime-endpoint
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/containerd/containerd.sock
```

docker와 비슷한 사용감을 원한다면 `nerdctl`을 사용하자. 하지만 ckad 실습환경에서 컨테이너 상태, 로그보기 에서는 `crictl` 에 익숙해지자.