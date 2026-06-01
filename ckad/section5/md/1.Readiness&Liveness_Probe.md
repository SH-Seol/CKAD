# Readiness & Liveness Probe

## 1. Pod Life cycle 간략 복기

| 상태 | 설명 |
|------|------|
| **Pending** | 스케줄러가 배치할 노드를 탐색 중 |
| **ContainerCreating** | 이미지 풀(pull) 후 컨테이너 시작 중 |
| **Running** | 모든 컨테이너 정상 기동. 앱 종료 시까지 유지 |

Pod에는 상태 외에 Conditions배열이 존재한다.(T/F)

| Condition       | 참이 되는 시점                    |
|-----------------|-----------------------------|
| PodScheduled    | 노드에 배정됨                     |
| Initialized     | 초기화 완료                      |
| ContainersReady | 모든 컨테이너 준비 완료               |
| Ready           | 위 세 조건 모두 True -> 트래픽 수신 가능 |

```bash
kubectl get pods # Ready 컬럼 확인
kubectl describe pod <pod-name> # Condition 상세 확인
```

---
## 2. Readiness Probe

**문제: 기본 동작의 한계**

쿠버네티스는 기본적으로 _컨테이너 생성 = Ready_로 간주한다만 실제 앱은 예열(warm-up)이 필요하다.

> 아직 준비 안 된 Pod로 트래픽 유입될 시 사용자 오류가 발생

해결책: Readiness Probe
> Probe가 성공할 때까지 Ready Condition을 True로 올리지 않는다. -> 트래픽 차단.

#### Probe 3종

1. HTTP GET
```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
```
2. TCP Socket
```yaml
readinessProbe:
  tcpSocket:
    port:3306
```
3. Exec Command
```yaml
readinessProbe:
  exec:
    command:
      - cat
      - /app/is_ready
```

#### 전체 YAML 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10 # 첫 프로브 전 대기
        periodSeconds: 5        # 프로브 실행 주기
        failureThreshold: 8     # 연속 실패 허용 횟수
```

#### 멀티 Pod 환경에서의 중요성
```
[기존 Pod A] ──┐
[기존 Pod B] ──┼── Service
[신규 Pod C] ──┘  ← 예열 중 (Probe 통과 전까지 트래픽 차단)
```
- Probe 없을 때: C가 준비되기 전에 트래픽 유입 -> 일부 사용자 오류
- Probe 있을 때: C 통과 전까지 A, B만 처리 -> 서비스 무중단

---

## 3.Liveness Probe

**문제: 컨테이너는 살아있지만 앱이 죽은 경우**
Docker에서는 컨테이너가 종료되면 수동으로 재시작해야 한다.
쿠버네티스는 크래시를 감지해 자동으로 재시작한다.
```bash
kubectl get pods
# NAME        READY   STATUS      RESTARTS   AGE
# nginx-pod   0/1     Completed   2          1d
```
그러나 컨테이너는 살아 있지만 앱이 정상 동작하지 않는 경우는 자동 복구가 되지 않는다.
> ex: 코드 버그로 앱이 무한 루프에 빠진 경우
> → 쿠버네티스는 컨테이너가 올라와 있으니 정상으로 판단
> → 사용자는 응답을 받지 못함

해결책: Liveness Probe
> 컨테이너 내부 앱의 건강 상태를 주기적으로 검사한다.
> Probe 실패 시 해당 컨테이너를 비정상으로 간주하고 재시작한다.

Probe 3종은 Readiness와 형태가 같다. 다만 `readinessProbe` → `livenessProbe`일 뿐이다.

#### 전체 YAML 예시
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /api/healthy
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 8
```

---
## 4. Readiness vs Liveness 비교

|항목| Readiness Probe | Liveness Probe       |
|--|-----------------|----------------------|
|목적| 트래픽 수신 준비 여부 확인 | 앱 정상 동작 여부 확인        |
|실패 시 동작| 트래픽 차단(Pod는 유지) | 컨테이너 재시작             |
|주요 사용 시점| 앱 시작 시 예열 기간    | 앱 실행 중 무한 루프, 데드락 감지 |
|YAML 필드| `readinessProbe` | `livenessProbe`       |


실제로는 두 Probe를 함께 사용하는 것이 일반적이다.
Readiness로 시작 전 트래픽을 막고, Liveness로 실행 중 상태를 감시한다.

#### 함께 사용한 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
      livenessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
      
        
```

---

### 요약

> Readiness: 아직 준비 안됐으니 트래픽 보내지 마
> Liveness: 앱이 죽었으니 컨테이너를 다시 시작해

- Readiness Probe - `readinessProbe` 필드, Probe 실패 시 트래픽 차단(Pod 유지), 스케일 아웃 시 무중단 배포의 핵심
- Liveness Probe - `livenessProbe` 필드, Probe 실패 시 컨테이너 재시작, 앱 크래시 없이 앱이 멈추는 상황(무한루프, 데드락) 대응
- Probe 3종: `httpGet`, `tcpSocket`, `exec` 두 Probe 모두 동일하게 적용
- 공통 옵션: `initialDelaySeconds`(초기 대기), `periodSeconds`(주기), `failureThreshold` (실패 허용 횟수)
- 실전: 두 Probe를 함께 설정하는 것이 권장 패턴