# Docker Web Server Workshop

## 0. 프로젝트 개요
이 프로젝트는 Docker 기초 실습을 통해 웹 서버 컨테이너를 실행하고, 이미지 빌드, 포트 매핑, 바인드 마운트, 볼륨, 권한 설정을 직접 확인하는 것을 목표로 한다.

실습을 통해 다음 내용을 수행했다.

- 작업 폴더 구조 생성
- 파일 및 디렉터리 권한 설정
- Docker 설치 확인
- `hello-world` 컨테이너 실행
- NGINX 커스텀 이미지 빌드
- 여러 포트로 컨테이너 실행
- 바인드 마운트 동작 확인
- Docker Volume 생성 및 연결
- Volume 영속성 확인

---
## 1. 실행 환경
- OS: macOS
- Shell: zsh
- Docker: 29.4.0
- Git: 2.53.0

## 2. 폴더 구조
```bash
docker-webserver-workshop/
├── app/
├── bind-app/
├── screenshots/
└── README.md
```

---

## 3. 실습 내용

### 3-1. 권한 설정
권한 실습을 위해 파일과 디렉터리의 권한을 설정했다.

- `perm-file.txt` : `644`
- `perm-dir` : `755`

확인 명령어:
```bash
ls -l perm-file.txt
ls -ld perm-dir
```

---

### 3-2. Docker 설치 확인
Docker가 정상적으로 설치되어 있는지 확인했다.

```bash
docker --version
```

또한 테스트용 컨테이너를 실행하여 Docker 동작을 검증했다.

```bash
docker run hello-world
```

---

### 3-3. NGINX 커스텀 이미지 빌드
NGINX 기반의 커스텀 이미지를 빌드했다.

```bash
docker build -t my-nginx .
```

이미지 생성 후 다음 명령어로 확인했다.

```bash
docker images
```

---

### 3-4. 포트 매핑 실습
동일한 웹 서버 이미지를 서로 다른 포트로 실행하여 브라우저에서 접속을 확인했다.

접속 주소:
- `localhost:8080`
- `localhost:8081`
- `localhost:8082`

실행 중인 컨테이너는 아래 명령어로 확인했다.

```bash
docker ps
```

---

### 3-5. 바인드 마운트 실습
호스트의 `bind-app` 폴더를 컨테이너와 연결하여, 파일 수정 내용이 브라우저에 즉시 반영되는지 확인했다.

수정 전후 페이지 변경을 통해 바인드 마운트가 정상적으로 동작함을 확인했다.

---

### 3-6. Docker Volume 실습
Docker Volume을 생성하고 컨테이너에 연결하여 데이터를 저장했다.

생성한 볼륨 이름:
- `my-nginx-data`

확인 명령어:
```bash
docker volume ls
docker inspect my-nginx-data
```

---

### 3-7. 볼륨 영속성 확인
컨테이너를 삭제한 뒤 다시 실행해도 Volume 데이터가 유지되는지 확인했다.

이를 통해 Docker Volume이 컨테이너와 독립적으로 데이터를 보존한다는 점을 확인했다.

---

## 4. 사용 명령어 정리

### Docker 관련
```bash
docker --version
docker run hello-world
docker build -t my-nginx .
docker images
docker ps
docker volume ls
docker inspect my-nginx-data
```

### 권한 확인
```bash
ls -l perm-file.txt
ls -ld perm-dir
```

---

## 5. 스크린샷

### 브라우저 화면
1. `screenshots/01-port-8080.png`  
   - `localhost:8080` 접속 화면

2. `screenshots/02-port-8081.png`  
   - `localhost:8081` 접속 화면

3. `screenshots/03-port-8082.png`  
   - `localhost:8082` 접속 화면

### 터미널 결과
4. `screenshots/04-docker-ps.png`  
   - `docker ps` 실행 결과

5. `screenshots/05-docker-images.png`  
   - `docker images` 실행 결과

6. `screenshots/06-docker-volume.png`  
   - `docker volume ls`  
   - `docker inspect my-nginx-data`

7. `screenshots/07-permissions.png`  
   - `ls -l perm-file.txt`  
   - `ls -ld perm-dir`

---

## 6. 실습 결과
이번 실습을 통해 Docker의 기본 사용법을 직접 확인할 수 있었다.

특히 다음 내용을 이해할 수 있었다.

- 컨테이너 실행과 이미지 관리 방법
- 포트 매핑을 통한 웹 서비스 노출 방식
- 바인드 마운트를 통한 실시간 파일 반영
- Docker Volume을 이용한 데이터 영속성
- 리눅스 파일 및 디렉터리 권한 확인 방법

---

## 7. 결론
Docker를 사용하면 웹 서버 실행, 테스트, 데이터 관리 등을 효율적으로 수행할 수 있다.  
이번 실습은 컨테이너 기반 개발 환경의 기본 개념을 익히는 데 도움이 되었으며, 앞으로 더 복잡한 애플리케이션 배포와 운영에도 활용할 수 있다.
```
