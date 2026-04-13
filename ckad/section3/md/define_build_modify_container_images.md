# Define, build and modify container images

우리가 Pod를 띄울 때 사용하는 이미지가 어떻게 만들어지는지 알아야 문제가 생겼을 때 이를 파악할 수 있다.

#### 🏗️ 1. 수동 배포 vs Docker 빌드

| 단계 | 수동 설치 과정(Ubuntu 기준) | Dockerfile 지시어 |
| --- | --- | --- |
| 베이스 OS | Ubuntu 설치 | `FROM Ubuntu` |
| 환경 업데이트 | `apt-get udpate` | `RUN apt-get update` |
| 의존성 설치 | `apt-get install python` , `pip instatll flask` | `RUN apt-get install ..` , `RUN pip install ..` |
| 코드 복사 | 소스 코드를 `/opt` 로 이동 | `COPY . /opt/source-code` |
| 서버 실행 | `flask run --host=0.0.0.0` | `ENTRYPOINT flask run ..` |

---

#### 📄 2. Dockerfile 핵심 작성법

Dockerfile은 지시어(Instruction) + 인수(Arguments)의 형식을 가진다. (akms와 비슷하게 여기는 frce)

```docker
# 1. 베이스 이미지 지정, 무조건 FROM으로 시작해야한다.
FROM ubuntu

# 2. 빌드 중 실행할 명령어(OS 업데이트 및 패키지 설치)
RUN apt-get update && apt-get -y install python python-setuptools python-dev
RUN pip install flask flask-mysql

# 3. 로컬의 파일을 이미지 내부로 복사(로컬 . -> 컨테이너 /opt/source-code)
COPY . /opt/source-code

# 4. 컨테이너가 시작될 때 실행될 최종 명령 (진입점)
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run --host=0.0.0.0
```

---

#### 🍰 3. 레이어드 아키텍처

Docker는 지시어 하나당 하나의 레이어를 만든다.

- 효율성: 이전 단계에서 변경사항이 없으면 기존 레이어를 캐시에서 재사용
- 속도: 소스코드(`COPY`)만 수정했다면 패키지 설치(`RUN`)과정은 건너뛰고 코드 복사 레이어부터 다시 빌드하므로 매우 빠르다.
- 확인: `docker history <이미지명>` 명령어로 레이어별 크기와 구성을 볼 수 있다.

---

#### 💻 4. 핵심 명령어 정리

이미지를 만들고 관리할 때 사용하는 필수 명령어

| 작업 | 명령어 | 설명 |
| --- | --- | --- |
| 빌드 | `docker build -f Dockerfile -t sh-seol/my-app .` | 현재 폴더(`.`)의 Dockerfile로 이미지 생성 |
| 히스토리 확인 | `docker history sh-seol/my-app` | 이미지 레이어 구조와 크기 확인 |
| 푸시 | `docker push sh-seol/my-app` | 생성한 이미지를 Docker Hub로 전송 |
| 실행 테스트 | `docker run sh-seol/myapp` | 빌드된 이미지가 잘 작동하는지 테스트 |

---

#### 주의사항

1. 캐시효율을 극대화하기 위해 RUN → FROM → COPY → ENTRYPOINT (RFCE) 순서대로 하자. 잘 바뀌는 것이 뒤로 가는 것이 좋다.
2. 나중에는 더 이상 필요 없는 프로그램을 설치하지 않고 컨테이너로 실행한 뒤 지우기만 하면 된다
3. 빌드 중 특정 단계에서 실패했다면 그 에러만 고치고 다시 빌드하면 된다. 성공한 내용은 docker에서 이미 기억하고 있다.

#### LAB에서 나온 docker commands

```docker
# 로컬에 저장된 이미지 목록 조회
docker images

# 현재 디렉토리 . 의 Dockerfile로 이미지 생성
# -t 는 tag이고 -t <이미지 이름>:<태그> -t image:v1 과 같이 가능함
docker build -t webapp-color .

# 이미지로 컨테이너 실행
# -p 8282:8080 => host:8282, container: 8080 
# localhost:8282를 container:8080에 연결
docker run -p 8282:8080 webapp-color

# 컨테이너 실행 후 명령어만 실행하고 종료
# python:3.6 이미지 실행 -> cat 명령 수행 -> 종료
# os 정보 출력을 하고 이미지 내부 확인할 때, 디버깅할 때 사용
docker run python:3.6 cat /etc/*release*
```

- build: 이미지 생성
- run: 컨테이너 실행

---

#### 필수 Docker 명령어

**이미지 관련**

```docker
# . 에 있는 Dockerfile을 myapp이라는 이름으로 이미지 생성
docker build -t myapp .

#이미지 목록 조회
docker images

# 이미지 삭제
docker rmi <image>

docker pull nginx
```

**컨테이너 실행/관리**

```docker
docker run nginx

# 백그라운드 실행 detached
docker run -d nginx

# -it 컨테이너 내부 쉘(인터랙티브 쉘, bash 등)로 들어가서 직접 대화하고 싶을 때 사용
docker run -it ubuntu bash

docker ps # 실행 중인 컨테이너
docker ps -a # 전체 컨테이너

docker stop <container>
docker start <container>
docker rm <container>
```

**디버깅**

```docker
docker logs <container>
docker exec -it <container> bash
docker inspect <container>
```

**네트워크**

```docker
docker run -p 8080:80 nginx
docker network ls
docker network create mynet
```

**볼륨**

```docker
docker volume ls
docker volume create myvol

# [호스트경로]:[컨테이너 경로]. 형식으로 내 컴퓨터의 폴더와 컨테이너 내부 폴더를 연결(마운트)
docker run -v myvol:/data nginx
```

**기타**

```docker
# 이미지에 새로운 이름이나 버전(태그)을 붙임
docker tag image newname

# 내가 만든 이미지를 도커 허브같은 원격 저장소에 푸시
docker push <repo>

# 도커 허브같은 원격 저장소에서 이미지를 받음
docker pull <repo>
```

간단 요약

- build → 이미지 생성
- run → 실행
- ps → 확인
- exec → 내부 진입