# Docker 웹서버 개발 워크스테이션 구축

## 1. 프로젝트 개요

터미널(리눅스 CLI), Docker(컨테이너), Git/GitHub(버전 관리)를 활용하여
**누구나 동일하게 재현 가능한 개발 환경**을 구축하는 미션이다.

- 터미널로 작업 디렉토리와 파일 권한을 관리한다.
- Dockerfile로 웹 서버(NGINX)를 컨테이너화하고, 포트 매핑으로 접속을 확인한다.
- 바인드 마운트로 **변경 즉시 반영**, Docker 볼륨으로 **데이터 영속성**을 검증한다.
- Git/GitHub로 결과물을 버전 관리하고 제출한다.

---

## 2. 실행 환경

| 항목 | 내용 |
|------|------|
| OS | macOS (서울캠퍼스 환경) |
| 쉘/터미널 | zsh / 기본 터미널 |
| 컨테이너 런타임 | OrbStack (sudo 권한 제약으로 Docker Desktop 대신 사용) |
| Docker | v28.5.2 |
| Git | v2.53.0 |
| 에디터 | VSCode (GitHub 연동) |
| 실행기간 | 2026.07.28~07.30 |

> **OrbStack 사용 이유**: 서울캠퍼스 보안 정책상 sudo 권한이 제한되어
> 일반적인 Docker 설치가 불가능하다. OrbStack은 sudo 없이 Docker 엔진을
> 구동할 수 있어, 실행 후 터미널에서 `docker` 명령을 동일하게 사용할 수 있다.
(단, 내부적으로 경량 가상머신을 거치므로 순수 Linux 호스트 환경과 네트워크 포트 매핑 시 미세한 차이가 발생할 수 있다.)

---

## 3. 수행 항목 체크리스트

- [x] 터미널 기본 조작 (이동/생성/복사/이름변경/삭제/내용확인)
- [x] 파일/디렉토리 권한 변경 실습 (변경 전/후 비교)
- [x] Docker 설치 확인 (`docker --version`, `docker info`)
- [x] Docker 기본 운영 명령 (`images`, `ps -a`, `logs`, `stats`)
- [x] hello-world 컨테이너 실행
- [x] ubuntu 컨테이너 실행 및 내부 명령 수행
- [x] attach vs exec 차이 관찰 및 정리
- [x] Dockerfile 기반 커스텀 이미지 빌드 (my-nginx)
- [x] 포트 매핑 3개 실행 및 브라우저 접속 확인 (8080/8081/8082)
- [x] 바인드 마운트 변경 즉시 반영 확인
- [x] Docker 볼륨 생성 및 영속성 검증 (컨테이너 삭제 후 데이터 유지)
- [x] Git 사용자 정보/기본 브랜치 설정
- [x] GitHub + VSCode 연동

---

## 4. 터미널 조작 로그

### 4-1. 현재 위치 및 목록 확인

```bash
$ pwd

/Users/ymru996022/dev-setup-codyssey

$ ls -al

total 0
drwxr-xr-x   9 ymru996022  ymru996022  288 Jul 28 16:40 .
drwxr-x---+ 21 ymru996022  ymru996022  672 Jul 28 15:07 ..
drwxr-xr-x  15 ymru996022  ymru996022  480 Jul 28 15:13 .git
drwxr-xr-x   4 ymru996022  ymru996022  128 Jul 28 15:02 app
drwxr-xr-x   3 ymru996022  ymru996022   96 Jul 28 15:02 bind-app
drwxr-xr-x   2 ymru996022  ymru996022   64 Jul 28 15:13 perm-dir
-rw-r--r--   1 ymru996022  ymru996022    0 Jul 28 15:02 perm-file.txt
-rw-r--r--   1 ymru996022  ymru996022    0 Jul 28 16:21 README.md
drwxr-xr-x   9 ymru996022  ymru996022  288 Jul 28 15:02 screenshots
```

### 4-2. 생성 / 파일 내용 확인

```bash
$ mkdir test-dir              # 디렉토리 생성
$ touch test-file.txt         # 빈 파일 생성
$ echo "hello" > test-file.txt
$ cat test-file.txt           # 파일 내용 확인
hello
```

### 4-3. 복사 / 이동·이름변경 / 이동 / 삭제

```bash
$ cp test-file.txt test-copy.txt      # 복사
$ mv test-copy.txt renamed.txt        # 이름 변경
$ cd test-dir                         # 이동
$ cd ..                               # 상위로 이동
$ rm renamed.txt test-file.txt        # 파일 삭제
$ rmdir test-dir                      # 디렉토리 삭제
```

> **절대 경로 vs 상대 경로**
> - 절대 경로: 루트(`/`)부터 시작하는 전체 경로. 예: `/Users/username/dev-setup-codyssey/app`
> - 상대 경로: 현재 위치 기준 경로. 예: `./app`, `../screenshots`
> - 호스트 환경에서 스크립트를 작성할 때는 어디서 실행하든 같은 곳을 가리키는 절대 경로, 프로젝트 내부 이동이나 컨테이너 내부 작업 시에는 이식성이 좋은 상대 경로를 사용하는 것을 권장한다.
    (예시: 컨테이너 내부에서 작업할 때 cd /usr/share/nginx/html처럼 절대 경로를 쓰거나, 현재 위치에서 cd ../처럼 상대 경로를 유연하게 활용할 수 있다.)

---

## 5. 권한 실습 (변경 전/후 비교)

### 5-1. 파일 권한: perm-file.txt (644 적용)

```bash
# 변경 전 (실습을 위해 임의로 모든 권한을 제거한 상태)
$ chmod 000 perm-file.txt
$ ls -l perm-file.txt

----------  1 ymru996022  ymru996022  0 Jul 28 15:02 perm-file.txt

# 권한 변경
$ chmod 644 perm-file.txt

# 변경 후
$ ls -l perm-file.txt

-rw-r--r--  1 ymru996022  ymru996022  0 Jul 28 15:02 perm-file.txt
```

### 5-2. 디렉토리 권한: perm-dir (755 적용)

```bash
# 변경 전 (실습을 위해 임의로 모든 권한을 부여한 상태)
$ chmod 777 perm-dir
$ ls -ld perm-dir

drwxrwxrwx  2 ymru996022  ymru996022  64 Jul 28 15:13 perm-dir

# 권한 변경
$ chmod 755 perm-dir

# 변경 후
$ ls -ld perm-dir

drwxr-xr-x  2 ymru996022  ymru996022  64 Jul 28 15:13 perm-dir
```

> **권한 표기 해석 규칙**
> - `r(4)` 읽기 / `w(2)` 쓰기 / `x(1)` 실행 — 숫자의 합으로 표현
> - 세 자리는 순서대로 **소유자 / 그룹 / 기타 사용자**
> - `644` = `rw- r-- r--` : 소유자만 수정 가능, 나머지는 읽기만
> - `755` = `rwx r-x r-x` : 소유자는 모든 권한, 나머지는 읽기+실행(디렉토리 진입)
> - 디렉토리의 `x`는 "실행"이 아니라 **디렉토리 진입 권한**을 의미한다.

> - 권한 설정 이유: 보안상 일반 텍스트 파일은 불필요한 실행을 막기 위해 최소 권한인 644를, 디렉토리는 내부 탐색이 가능해야 하므로 755를 적용했다.
> - 실무 적용 사례: 실행이 필요한 쉘 스크립트(.sh)는 755를, 민감한 인증 키 파일은 소유자만 읽을 수 있도록 600 권한을 부여하는 것이 권장 패턴이다.
> - 소유자 변경: chmod로 권한 비트를 바꾸는 것 외에도, 필요시 chown 명령어를 사용해 파일이나 디렉토리의 소유자(User)와 그룹(Group) 자체를 변경하여 실무적인 접근 제어를 할 수 있다.
📸 증거: [screenshots/07-permissions.png](screenshots/07-permissions.png)

---

## 6. Docker 설치 및 기본 점검

### 6-1. 버전 확인

```bash
$ docker --version

Docker version 28.5.2, build ecc6942
```

### 6-2. 데몬 동작 확인

```bash
$ docker info

Client:
 Version:    28.5.2
 Context:    orbstack

Server:
 Containers: 3
  Running: 3
  Paused: 0
  Stopped: 0
 Images: 1
 Server Version: 28.5.2
 ... (중략) ...
 Operating System: OrbStack
 OSType: linux
 Architecture: x86_64
 CPUs: 6
 Total Memory: 15.67GiB
 Name: orbstack
 ... (이하 생략)
```

📸 증거: [screenshots/08-docker-info.png](screenshots/08-docker-info.png)

---

## 7. Docker 기본 운영 명령

### 7-1. 이미지 목록

```bash
$ docker images

REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
my-nginx      latest    7c8a26e70f2e   3 hours ago    62.4MB
nginx         latest    4e5db4761e0f   12 days ago    161MB
hello-world   latest    e2ac70e7319a   4 months ago   10.1kB
```

📸 증거: [screenshots/05-docker-images.png](screenshots/05-docker-images.png)

### 7-2. 전체 컨테이너 목록

```bash
$ docker ps -a

CONTAINER ID   IMAGE      COMMAND                  CREATED       STATUS       PORTS                                     NAMES
00d8ec84cd2b   my-nginx   "/docker-entrypoint.…"   3 hours ago   Up 3 hours   0.0.0.0:8082->80/tcp, [::]:8082->80/tcp   my-nginx-8082
c850ea8af3a8   my-nginx   "/docker-entrypoint.…"   3 hours ago   Up 3 hours   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   my-nginx-8081
b180f16e3f22   my-nginx   "/docker-entrypoint.…"   3 hours ago   Up 3 hours   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-nginx-8080
```

📸 증거: [screenshots/04-docker-ps-a.png](screenshots/04-docker-ps-a.png)

### 7-3. 컨테이너 로그 확인

```bash
$ docker logs my-nginx-8080

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/07/28 06:36:26 [notice] 1#1: using the "epoll" event method
2026/07/28 06:36:26 [notice] 1#1: nginx/1.31.3
2026/07/28 06:36:26 [notice] 1#1: built by gcc 15.2.0 (Alpine 15.2.0) 
2026/07/28 06:36:26 [notice] 1#1: OS: Linux 6.17.8-orbstack-00308-g8f9c941121b1
2026/07/28 06:36:26 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 20480:1048576
2026/07/28 06:36:26 [notice] 1#1: start worker processes
2026/07/28 06:36:26 [notice] 1#1: start worker process 30
2026/07/28 06:36:26 [notice] 1#1: start worker process 31
2026/07/28 06:36:26 [notice] 1#1: start worker process 32
2026/07/28 06:36:26 [notice] 1#1: start worker process 33
2026/07/28 06:36:26 [notice] 1#1: start worker process 34
2026/07/28 06:36:26 [notice] 1#1: start worker process 35
```

📸 증거: [screenshots/09-docker-logs.png](screenshots/09-docker-logs.png)

### 7-4. 리소스 사용량 확인

```bash
$ docker stats --no-stream

CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O        PIDS
00d8ec84cd2b   my-nginx-8082   0.00%     6.199MiB / 15.67GiB   0.04%     1.48kB / 126B   4.1kB / 8.19kB   7
c850ea8af3a8   my-nginx-8081   0.00%     5.52MiB / 15.67GiB    0.03%     1.52kB / 126B   4.1kB / 8.19kB   7
b180f16e3f22   my-nginx-8080   0.00%     5.828MiB / 15.67GiB   0.04%     1.91kB / 126B   4.1kB / 8.19kB   7
```

📸 증거: [screenshots/10-docker-stats.png](screenshots/10-docker-stats.png)

---

## 8. 컨테이너 실행 실습

### 8-1. hello-world 실행

```bash
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
Digest: sha256:c3cbe1cc1aa588a64951ac6286e0df7b27fe2e6324b1001c619bb358770c0178
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

📸 증거: [screenshots/17-hello-world.png](screenshots/17-hello-world.png)

### 8-2. ubuntu 컨테이너 실행 및 내부 명령

```bash
$ docker run -it --rm ubuntu:22.04 bash

Unable to find image 'ubuntu:22.04' locally
22.04: Pulling from library/ubuntu
d6834b4a794c: Pull complete 
Digest: sha256:0e0a0fc6d18feda9db1590da249ac93e8d5abfea8f4c3c0c849ce512b5ef8982
Status: Downloaded newer image for ubuntu:22.04

root@a2279da0d836:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@a2279da0d836:/# echo "hello from ubuntu"
hello from ubuntu

root@a2279da0d836:/# cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.5 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy

root@a2279da0d836:/# exit
exit

ymru996022@c5r3s4 dev-setup-codyssey %
```

📸 증거: [screenshots/11-ubuntu-shell.png](screenshots/11-ubuntu-shell.png)

### 8-3. attach vs exec 차이 관찰

```bash
# exec: 실행 중인 컨테이너에 "새 프로세스"로 접속
$ docker exec -it my-nginx-8080 sh
root@xxxx:/# ls
root@xxxx:/# exit          # 나가도 컨테이너는 계속 실행됨

/ # ls
bin                   docker-entrypoint.d   etc                   lib                   mnt                   proc                  run                   srv                   tmp                   var
dev                   docker-entrypoint.sh  home                  media                 opt                   root                  sbin                  sys                   usr
/ # exit


# attach: 컨테이너의 "메인 프로세스(PID 1)"에 직접 접속
$ docker attach my-nginx-8080

^C2026/07/28 10:20:53 [notice] 1#1: signal 2 (SIGINT) received, exiting
2026/07/28 10:20:53 [notice] 31#31: exiting
2026/07/28 10:20:53 [notice] 30#30: exiting
2026/07/28 10:20:53 [notice] 35#35: exiting
2026/07/28 10:20:53 [notice] 33#33: exiting
2026/07/28 10:20:53 [notice] 34#34: exiting
2026/07/28 10:20:53 [notice] 35#35: exit
2026/07/28 10:20:53 [notice] 34#34: exit
2026/07/28 10:20:53 [notice] 31#31: exit
2026/07/28 10:20:53 [notice] 30#30: exit
2026/07/28 10:20:53 [notice] 32#32: exiting
2026/07/28 10:20:53 [notice] 32#32: exit
2026/07/28 10:20:53 [notice] 33#33: exit
2026/07/28 10:20:53 [notice] 1#1: signal 17 (SIGCHLD) received from 32
2026/07/28 10:20:53 [notice] 1#1: worker process 32 exited with code 0
2026/07/28 10:20:53 [notice] 1#1: signal 29 (SIGIO) received
2026/07/28 10:20:53 [notice] 1#1: signal 17 (SIGCHLD) received from 30
2026/07/28 10:20:53 [notice] 1#1: worker process 30 exited with code 0
2026/07/28 10:20:53 [notice] 1#1: worker process 31 exited with code 0
2026/07/28 10:20:53 [notice] 1#1: signal 29 (SIGIO) received
2026/07/28 10:20:53 [notice] 1#1: signal 17 (SIGCHLD) received from 35
2026/07/28 10:20:53 [notice] 1#1: worker process 33 exited with code 0
2026/07/28 10:20:53 [notice] 1#1: worker process 34 exited with code 0
2026/07/28 10:20:53 [notice] 1#1: worker process 35 exited with code 0
2026/07/28 10:20:53 [notice] 1#1: exit
# NGINX 로그 출력에 붙게 됨
# Ctrl+P, Ctrl+Q 로 분리(detach) — Ctrl+C 하면 컨테이너가 종료됨

```

> **관찰 정리**
> - `exec`은 **새로운 셸 프로세스를 추가로 생성**해서 접속하므로, `exit` 해도 컨테이너에 영향이 없다. → 디버깅/점검용으로 안전
> - `attach`는 **메인 프로세스(PID 1)에 직접 붙는 것**이므로, `Ctrl+C`로 종료하면
>   메인 프로세스가 죽어 컨테이너 자체가 종료된다. 반드시 `Ctrl+P, Ctrl+Q`로 분리해야 한다.
> - 결론: 실행 중인 컨테이너 내부 작업은 `exec`을 사용하는 것이 안전하다.



📸 증거: [screenshots/12-attach-vs-exec.png](screenshots/12-attach-vs-exec.png)

---

## 9. 커스텀 이미지 제작 (방식 A: 웹 서버 베이스)

### 9-1. 베이스 이미지 선택 및 커스텀 포인트

| 항목 | 내용 | 목적 |
|------|------|------|
| 베이스 이미지 | `nginx:latest` (공식 이미지) | 검증된 웹 서버를 그대로 활용 |
| 커스텀 포인트 | `index.html` 교체 (`COPY`) | 나만의 정적 콘텐츠 제공 확인 |

### 9-2. Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

### 9-3. 빌드 및 실행

```bash
$ docker build -t my-nginx .

[+] Building 2.2s (7/7) FINISHED                                                                                                                                                                                                           docker:orbstack
 => [internal] load build definition from Dockerfile                                                                                                                                                                                                  0.1s
 => => transferring dockerfile: 104B                                                                                                                                                                                                                  0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine                                                                                                                                                                                       1.7s
 => [internal] load .dockerignore                                                                                                                                                                                                                     0.1s
 => => transferring context: 2B                                                                                                                                                                                                                       0.0s
 => [internal] load build context                                                                                                                                                                                                                     0.1s
 => => transferring context: 32B                                                                                                                                                                                                                      0.0s
 => [1/2] FROM docker.io/library/nginx:alpine@sha256:4a73073bd557c65b759505da037898b61f1be6cbcc3c2c3aeac22d2a470c1752                                                                                                                                 0.0s
 => CACHED [2/2] COPY index.html /usr/share/nginx/html/index.html                                                                                                                                                                                     0.0s
 => exporting to image                                                                                                                                                                                                                                0.0s
 => => exporting layers                                                                                                                                                                                                                               0.0s
 => => writing image sha256:7c8a26e70f2e6ea1016ff2845a5470f1c1ab86a4c4c9bb2164d0d5ea193b290b                                                                                                                                                          0.0s
 => => naming to docker.io/library/my-nginx  


$ docker run -d --name my-nginx-8080 -p 8080:80 my-nginx
$ docker run -d --name my-nginx-8081 -p 8081:80 my-nginx
$ docker run -d --name my-nginx-8082 -p 8082:80 my-nginx

$ docker ps

CONTAINER ID   IMAGE      COMMAND                  CREATED       STATUS         PORTS                                     NAMES
00d8ec84cd2b   my-nginx   "/docker-entrypoint.…"   4 hours ago   Up 4 hours     0.0.0.0:8082->80/tcp, [::]:8082->80/tcp   my-nginx-8082
c850ea8af3a8   my-nginx   "/docker-entrypoint.…"   4 hours ago   Up 4 hours     0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   my-nginx-8081
b180f16e3f22   my-nginx   "/docker-entrypoint.…"   4 hours ago   Up 3 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-nginx-8080
```

📸 증거: [screenshots/13-docker-build.png](screenshots/13-docker-build.png)

> **이미지 vs 컨테이너 (불변성과 런타임 변경)**
> - 이미지(Image): 한 번 빌드되면 절대 변하지 않는 **불변성(Immutable)**을 가진 읽기 전용 템플릿이다.
> - 컨테이너(Container): 이미지를 바탕으로 실행된 런타임 인스턴스로, 상단에 **쓰기 가능한 레이어(Writable Layer)**가 추가되어 내부에서 파일 생성이나 변경이 가능하다.

> - 빌드 최적화 팁: .dockerignore 파일을 생성하여 불필요한 파일이 이미지 빌드 컨텍스트에 포함되는 것을 방지하면, 빌드 속도 향상과 용량 최적화가 가능하다.
> - 이미지 캐시: 이미지를 변경하고 재빌드할 때, 변경되지 않은 이전 레이어는 캐시를 그대로 사용하여 빌드 속도를 최적화할 수 있다.
---

## 10. 포트 매핑 및 접속 증거

| 접속 주소 | 컨테이너 | 결과 |
|------|------|------|
| http://localhost:8080 | my-nginx-8080 | ✅ 접속 성공 |
| http://localhost:8081 | my-nginx-8081 | ✅ 접속 성공 |
| http://localhost:8082 | my-nginx-8082 | ✅ 접속 성공 |

📸 증거 (주소창 포함):
- [screenshots/01-port-8080.png](screenshots/01-port-8080.png)
- [screenshots/02-port-8081.png](screenshots/02-port-8081.png)
- [screenshots/03-port-8082.png](screenshots/03-port-8082.png)

> **포트 매핑이 필요한 이유**
> 컨테이너는 격리된 네트워크 공간에서 실행되므로, 호스트에서 직접 접근할 수 없다.
> `-p <호스트포트>:<컨테이너포트>` 옵션으로 호스트의 포트를 컨테이너 내부 포트에
> 연결해야 브라우저 접속이 가능하다. 또한 하나의 이미지를 서로 다른 호스트 포트
> (8080/8081/8082)로 여러 번 실행할 수 있어, **동일 환경의 반복 재현**이 가능함을 확인했다.

> 보안 및 접근 제어: 외부 접근이 불필요한 경우 -p 127.0.0.1:8080:80처럼 로컬호스트로만 포트를 바인딩하는 것이 안전하다. 만약 원격지 브라우저에서 접근이 안 된다면, 호스트 장비의 인바운드 방화벽 규칙과 포트 포워딩 설정을 우선 점검해야 한다.
> 네임스페이스와 포트 노출: 컨테이너는 호스트와 완전히 독립된 네트워크 네임스페이스를 가지므로, 호스트의 포트와 컨테이너 내부 포트를 바인딩해야만 트래픽이 전달될 수 있다.
---

## 11. 바인드 마운트: 변경 즉시 반영

### 11-1. 실행 명령
(전제조건: 터미널이 프로젝트 최상단 디렉토리에 위치해야 하며, 사전에 bind-app 폴더가 존재해야 한다.)

```bash
$ docker run -d --name my-nginx-bind -p 8090:80 \
  -v $(pwd)/bind-app:/usr/share/nginx/html my-nginx

  41219362a4b7cf299a6a812b82169288cce5ef8b4af088eb4c4e404c67037693
```

### 11-2. 변경 전/후 비교

```bash
# 변경 전: 브라우저에서 기존 내용 확인

# 호스트에서 파일 수정
$ echo "<h1>Updated!</h1>" > bind-app/index.html
$ curl localhost:8090 

<h1>Updated!</h1>

# 변경 후: 브라우저 새로고침 → 즉시 반영 확인 (재빌드/재시작 없음)
```

> **관찰**: 바인드 마운트는 호스트 디렉토리를 컨테이너 내부 경로에 직접 연결하므로,
> 호스트에서 파일을 수정하면 **이미지 재빌드 없이 즉시** 컨테이너에 반영된다.
> → 개발 중 코드 수정을 바로 확인하는 용도에 적합하다.

📸 증거: [screenshots/14-bind-mount.png](screenshots/14-bind-mount.png) 

---

## 12. Docker 볼륨: 데이터 영속성 검증

### 12-1. 볼륨 생성 및 연결

```bash
$ docker volume create my-nginx-data

my-nginx-data

$ docker volume ls

DRIVER    VOLUME NAME
local     my-nginx-data

$ docker volume inspect my-nginx-data

[
    {
        "CreatedAt": "2026-07-29T11:24:18+09:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my-nginx-data/_data",
        "Name": "my-nginx-data",
        "Options": null,
        "Scope": "local"
    }
]

$ docker run -d --name my-nginx-vol -p 8091:80 \
  -v my-nginx-data:/usr/share/nginx/html my-nginx

  9b61de94ba8a482ef05584c8ce7457fba309da609e52688c8cf9062b94085b38
```

### 12-2. 데이터 생성 → 컨테이너 삭제 → 데이터 유지 확인

```bash
# 1) 컨테이너 안에 데이터 생성
$ docker exec my-nginx-vol sh -c 'echo "persistent data" > /usr/share/nginx/html/data.txt'


# 2) 삭제 전 확인
$ docker exec my-nginx-vol cat /usr/share/nginx/html/data.txt

persistent data


# 3) 컨테이너 완전 삭제
$ docker rm -f my-nginx-vol

my-nginx-vol


# 4) 같은 볼륨으로 새 컨테이너 실행
$ docker run -d --name my-nginx-vol2 -p 8091:80 \
  -v my-nginx-data:/usr/share/nginx/html my-nginx

1696e46c504902f653aaff77afb74e291cac395797b68e5e70b9b373fadfd114


# 5) 삭제 후 확인 → 데이터 유지됨 ✅
$ docker exec my-nginx-vol2 cat /usr/share/nginx/html/data.txt

persistent data
```

> **볼륨 vs 바인드 마운트**
> - 바인드 마운트: 호스트의 특정 경로를 연결 → 개발 중 실시간 반영에 적합
> - 볼륨: Docker가 관리하는 저장 공간 → 컨테이너의 생명주기와 무관하게 데이터가
>   유지되므로 **DB 데이터 등 영속 데이터 보관**에 적합하다.
> - 컨테이너는 삭제되면 내부 데이터도 사라지지만, 볼륨에 저장된 데이터는 살아남는다.
>   → 위 실험에서 컨테이너 삭제 후에도 `data.txt`가 유지됨을 직접 검증했다.

📸 증거: [screenshots/06-volume-persistence.png](screenshots/06-volume-persistence.png)


### 12-3. 데이터 손실 방지를 위한 볼륨 백업 절차

```bash
# 볼륨(my-nginx-data)의 데이터를 현재 호스트 디렉토리($(pwd))의 backup.tar 파일로 압축하여 백업
$ docker run --rm -v my-nginx-data:/data -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /data

tar: removing leading '/' from member names
/data/
/data/data.txt

# 백업 파일 생성 확인
$ ls -al backup.tar

-rw-r--r--  1 ymru996022  ymru996022  10240 Jul 29 11:25 backup.tar


> - **💡 볼륨 데이터 백업 절차**
> - 볼륨은 데이터를 안전하게 보관하지만, 만약의 사태(볼륨 자체의 삭제나 손상)에 대비한 백업 절차가 필요하다. 
> - 일회용 임시 컨테이너(`--rm`)를 띄워서 기존 볼륨(`my-nginx-data`)과 호스트의 디렉토리(`$(pwd)`)를 동시에 마운트한 뒤, 볼륨 안의 데이터를 압축(`tar`)하여 호스트로 빼내는 방식을 사용한다.
> - 자동화 권장: 실제 운영 환경에서는 데이터 손실을 완벽히 방지하기 위해, 이러한 볼륨 백업 명령을 쉘 스크립트로 작성하여 일간(Daily) 또는 주간(Weekly) 단위로 자동화하는 것을 권장한다.

---

## 13. Git 설정 및 GitHub 연동

### 13-1. Git 사용자 정보 / 기본 브랜치 설정

```bash
$ git config --global user.name "유영민"
$ git config --global user.email "ymru99@gmail.com"
$ git config --global init.defaultBranch main

$ git config --list
credential.helper=osxkeychain
user.name=유영민
user.email=ymru99@gmail.com
init.defaultbranch=main
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
remote.origin.url=https://github.com/imyoman99/dev-setup-codyssey.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
branch.main.merge=refs/heads/main
branch.main.vscode-merge-base=origin/main
'''
'''

📸 증거: [screenshots/15-git-config.png](screenshots/15-git-config.png)

### 13-2. GitHub + VSCode 연동

- VSCode에서 GitHub 계정 로그인 완료
- 본 저장소를 VSCode에서 clone/push 하여 연동 확인

📸 증거: [screenshots/16-vscode-github.png](screenshots/16-vscode-github.png)
        정상적으로 Push 된 원격 저장소 확인 => [https://github.com/imyoman99/dev-setup-codyssey.git](https://github.com/imyoman99/dev-setup-codyssey.git)

> **Git vs GitHub**
> - Git: 내 컴퓨터에서 동작하는 **로컬 버전 관리 도구** (커밋, 브랜치, 이력 관리)
> - GitHub: Git 저장소를 호스팅하는 **원격 협업 플랫폼** (공유, PR, 이슈, 리뷰)
> - Git 없이 GitHub는 쓸 수 없고, GitHub 없이도 Git은 로컬에서 동작한다.

---

## 14. 트러블슈팅

### 사례 1: 포트 충돌로 컨테이너 실행 실패

- **문제**: `docker ru -d -p 8080:80 ...` 실행 시 `failed: port is already allocated` 에러 발생
- **원인 가설**: 이미 8080 포트를 점유한 컨테이너 또는 프로세스가 존재
- **확인**:
  ```bash
  $ docker ps            # 8080 사용 중인 컨테이너 확인
  $ lsof -i :8080        # 호스트 프로세스 확인
  ```

- **문제**: `docker run -p 8080:80 ...` 실행 시 `port is already allocated` 에러 발생
- **원인 가설**: 이미 8080 포트를 점유한 컨테이너 또는 프로세스가 존재
- **해결**: 다른 호스트 포트(예: 8081)로 매핑하여 실행. 컨테이너 포트(80)는 그대로 두고
           호스트 포트만 바꾸면 되는 것이 포트 매핑의 장점임을 확인했다. (컨테이너 이름 확인 후 $ docker rm -f 으로 기존 컨테이너 삭제 후 재실행하는 방법도 있다.)

📸 증거: [screenshots/18-port-conflict.png](screenshots/18-port-conflict.png)

### 사례 2: 컨테이너 라이프사이클 위반 (실행 중인 컨테이너 삭제 시도)

- **문제**: 불필요한 컨테이너를 정리하기 위해 `docker rm my-safe-nginx`를 실행했으나, `You cannot remove a running container...` 에러가 발생하며 삭제가 거부됨.
- **원인 진단**: 에러 메시지를 통해, 해당 컨테이너가 현재 서비스 중(`Up` 상태)이므로 도커 데몬이 시스템 보호를 위해 삭제를 원천 차단했음을 파악함. 이는 단순 오류가 아닌 도커의 **컨테이너 생명주기(Lifecycle) 보호 정책**에 의한 방어 기제임.
- **해결**: 도커의 원칙에 따라 `docker stop my-safe-nginx` 명령어로 컨테이너를 안전하게 종료(`Exited` 상태로 전환)시킨 뒤, 다시 `docker rm`을 수행하여 정상적으로 삭제 완료함. 실무에서 운영 중인 서버의 실수에 의한 강제 삭제를 막아주는 중요한 개념임을 배움. (긴급 시에는 `rm -f`로 강제 삭제 가능함도 확인)

- 📸 증거: [screenshots/19-lifecycle-error.png](screenshots/19-lifecycle-error.png)

---

## 15. 검증 방법 요약 (요구사항 ↔ 증거 매핑)

| 검증 항목 | 사용한 명령 | 증거 위치 |
|------|------|------|
| 터미널 조작 | `pwd`, `ls -al`, `mkdir`, `touch`, `cat`, `cp`, `mv`, `rm` | [4번 섹션](#4-터미널-조작-로그) |
| 권한 변경 | `chmod 644/755`, `ls -l`, `ls -ld` | [5번 섹션](#5-권한-실습-변경-전후-비교), screenshots/07 |
| Docker 설치 | `docker --version`, `docker info` | [6번 섹션](#6-docker-설치-및-기본-점검), screenshots/08 |
| 운영 명령 | `docker images`, `ps -a`, `logs`, `stats` | [7번 섹션](#7-docker-기본-운영-명령), screenshots/04, 05, 09, 10 |
| 컨테이너 실행 | `docker run` (hello-world, ubuntu) | [8번 섹션](#8-컨테이너-실행-실습), screenshots/11, 12 |
| 커스텀 이미지 | `docker build -t my-nginx .` | [9번 섹션](#9-커스텀-이미지-제작-방식-a-웹-서버-베이스), screenshots/13 |
| 포트 매핑 | `docker run -p 8080:80` 외 2개 | [10번 섹션](#10-포트-매핑-및-접속-증거), screenshots/01, 02, 03 |
| 바인드 마운트 | `docker run -v $(pwd)/bind-app:...` | [11번 섹션](#11-바인드-마운트-변경-즉시-반영), screenshots/14 |
| 볼륨 영속성 | `docker volume create`, `rm -f` 후 재확인 | [12번 섹션](#12-docker-볼륨-데이터-영속성-검증), screenshots/06 |
| Git/GitHub | `git config --list`, VSCode 연동 | [13번 섹션](#13-git-설정-및-github-연동), screenshots/15, 16 |

---

## 16. 폴더 구조

```
dev-setup-codyssey/
├── README.md                         # 전체 프로젝트 설명서
├── backup.tar                        # 볼륨 백업 실습 결과물 (임시 컨테이너로 추출)
├── app/
│   ├── Dockerfile                    # NGINX 커스텀 이미지 빌드용
│   └── index.html                    # 커스텀 이미지에 포함되는 정적 콘텐츠
├── bind-app/
│   └── index.html                    # 바인드 마운트 실습용
├── perm-file.txt                     # 권한 실습용 파일 (644)
├── perm-dir/                         # 권한 실습용 디렉토리 (755)
│   └── .gitkeep                      # 빈 폴더를 Git 시스템에 추적/유지시키기 위한 더미 파일
└── screenshots/
    ├── 01-port-8080.png              # localhost:8080 접속 화면
    ├── 02-port-8081.png              # localhost:8081 접속 화면
    ├── 03-port-8082.png              # localhost:8082 접속 화면
    ├── 04-docker-ps.png              # docker ps 결과
    ├── 05-docker-images.png          # docker images 결과
    ├── 06-docker-volume.png          # docker volume / inspect 결과
    ├── 07-permissions.png            # 권한 확인 결과
    ├── 08-docker-info.png            # docker info 결과
    ├── 09-docker-ps-a.png            # docker ps -a (전체 컨테이너)
    ├── 10-docker-logs.png            # docker logs 결과
    ├── 11-docker-stats.png           # docker stats 결과
    ├── 12-ubuntu-shell.png           # Ubuntu 컨테이너 내부 쉘
    ├── 13-attach-vs-exec.png         # attach vs exec 비교
    ├── 14-git-config.png             # git config --list 결과
    ├── 15-vscode-github.png          # VSCode GitHub 연동
    ├── 16-github-vscode.png          # GitHub + VSCode 확인
    ├── 17-hello-world.png            # hello-world 실행 결과 화면
    ├── 18-port-conflict.png          # 사례 1: 포트 충돌 진단(lsof) 및 우회 해결
    └── 19-lifecycle-error.png        # 사례 2: 도커 생명주기 위반 에러 진단 및 안전 종료(stop) 해결
```

---

## 17. 배운 점 정리 (과제 목표 자가 점검)

- **절대 경로 vs 상대 경로**: 루트 기준(`/Users/...`) vs 현재 위치 기준(`./app`) — 4번 섹션
- **권한 표기(r/w/x, 755/644)**: 숫자 합산 규칙과 소유자/그룹/기타 구분 — 5번 섹션
- **커스텀 이미지 제작**: 공식 nginx 베이스 + COPY로 콘텐츠 교체 — 9번 섹션
- **이미지 vs 컨테이너**: 이미지는 불변성(Immutable)을 가진 읽기 전용 템플릿이고, 컨테이너는 Writable Layer가 추가된 런타임 인스턴스 — 9번 섹션
- **포트 매핑의 필요성**: 격리된 컨테이너 네트워크를 호스트와 연결 — 10번 섹션
- **Docker 볼륨**: 컨테이너 생명주기와 독립적인 영속 저장소 — 12번 섹션
- **Git vs GitHub**: 로컬 버전관리 도구 vs 원격 협업 플랫폼 — 13번 섹션
- **백업 주기**: 주기적인 데이터 보호를 위해 일간 또는 주간 단위로 백업을 자동화하는 것을 권장
---

## 18. 참고 사항

저장소 이름이 `ai-codyssey`에서 `dev-setup-codyssey`로 변경되었으며, 이로 인해 문서 내 일부 경로·표기와 스크린샷 파일명/설명이 기존 자료와 조금 다르게 보일 수 있습니다. 다만 실습 내용, 실행 결과, 파일 구조, Docker 및 Git 동작 방식 자체는 동일하게 반영되었습니다.

