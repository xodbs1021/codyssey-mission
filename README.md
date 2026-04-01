# 미션 1: 개발 워크스테이션 구축

## 1. 프로젝트 개요
본 프로젝트는 특정 로컬 머신에 종속되지 않고, 팀원 누구나 동일한 방식으로 실행 및 배포할 수 있는 **재현 가능한 개발 워크스테이션 환경**을 구축하는 것을 목표로 합니다. 리눅스 CLI 조작, Docker 컨테이너 격리 및 실행, 볼륨을 통한 데이터 영속성 검증 과정을 수행했습니다.

---

## 2. 실행 환경
* **OS**: macOS (Apple Silicon / aarch64)
* **Shell / Terminal**: zsh
* **Docker**: Docker version 28.5.2, build ecc6942 (OrbStack 환경)
* **Git**: git version 2.50.1 (Apple Git-155)

---

## 3. 수행 항목 체크리스트
✅ 터미널 기본 조작 및 디렉토리/파일 관리
✅ 파일 권한(chmod) 변경 실습 및 증거 기록
✅ Docker 설치(OrbStack) 및 데몬 동작 점검
✅ Docker 기본 컨테이너(hello-world, ubuntu) 실행 및 라이프사이클 관찰
✅ 커스텀 Dockerfile 작성 및 이미지 빌드
✅ 웹 서버 컨테이너 포트 매핑(-p) 및 접속 검증
✅ 바인드 마운트(-v)를 통한 실시간 변경 반영 검증
✅ Docker Volume을 활용한 데이터 영속성 검증
⬜ Git 사용자 설정 및 GitHub 연동 (마지막에 진행 예정)

---

## 4. 터미널 조작 및 권한 실습

### 4.1 작업 디렉토리 및 파일 생성
터미널을 이용해 과제용 디렉토리를 생성하고 빈 파일을 만들었습니다.
```bash
kty@gimtaeyun-ui-MacBookAir ~ % mkdir codyssey-mission
kty@gimtaeyun-ui-MacBookAir ~ % cd codyssey-mission
kty@gimtaeyun-ui-MacBookAir codyssey-mission % touch memo.txt
kty@gimtaeyun-ui-MacBookAir codyssey-mission % echo "Hello" > memo.txt
kty@gimtaeyun-ui-MacBookAir codyssey-mission % cat memo.txt
Hello
```
### 4.2 파일 권한 변경 (chmod)
생성된 memo.txt 파일의 권한을 644에서 755로 변경하여 소유자에게 실행(x) 권한을 부여하는 실습을 진행했습니다.
```bash
# 권한 변경 전 확인
kty@kty codyssey-mission % ls -l memo.txt
-rw-r--r--  1 kty  staff  6  3월 31 14:16 memo.txt

# 권한 변경 수행 (755)
kty@kty codyssey-mission % chmod 755 memo.txt

# 권한 변경 후 확인 (x 권한 추가됨)
kty@kty codyssey-mission % ls -l memo.txt
-rwxr-xr-x  1 kty  staff  6  3월 31 14:16 memo.txt
```
검증 결과: 소유자(User) 권한이 -rw-r--r--에서 -rwxr-xr-x로 정상 변경됨을 확인했습니다.

---
## 5. Docker 설치 및 기본 점검
sudo 권한 사용이 제한되는 환경을 고려하여 OrbStack을 활용해 Docker 데몬을 구동했습니다.
```bash
# 도커 버전 확인
kty@kty codyssey-mission % docker --version
Docker version 28.5.2, build ecc6942

# 도커 데몬 동작 여부 및 환경 확인
kty@kty codyssey-mission % docker info
Client:
 Version:    28.5.2
 Context:    orbstack
... (중략) ...
Server:
 Containers: 0
 Images: 0
 Server Version: 28.5.2
 Operating System: OrbStack
 Architecture: aarch64
```

---

## 6. Docker 기본 운영 명령 수행

### 6.1 `hello-world` 컨테이너 실행
Docker Hub로부터 이미지를 Pull 받아 실행하는 흐름을 확인했습니다.
```bash
kty@kty codyssey-mission % docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```
### 6.2 ubuntu 컨테이너 접속 (격리 환경 확인)
-it 옵션을 통해 컨테이너 내부에 접속하여 독립된 파일 시스템을 관찰했습니다.
```bash
kty@kty codyssey-mission % docker run -it ubuntu bash
root@6090259bc7c9:/# ls
bin   dev  home  media  opt   root  sbin  sys  usr
boot  etc  lib   mnt    proc  run   srv   tmp  var
root@6090259bc7c9:/# exit
```
### 6.3 이미지 및 컨테이너 상태 점검
```bash
# 다운로드된 이미지 목록 확인
kty@kty codyssey-mission % docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    eb84fdc6f2a3   7 days ago    5.2kB
ubuntu        latest    e3847ac055b4   5 weeks ago   101MB

# 컨테이너 실행 이력 확인
kty@kty codyssey-mission % docker ps -a
CONTAINER ID   IMAGE         COMMAND    STATUS                     PORTS   NAMES
6090259bc7c9   ubuntu        "bash"     Exited (0) 34 seconds ago          gallant_moser
114f93ba582f   hello-world   "/hello"   Exited (0) 37 minutes ago          zealous_carver
```

---

## 7. 커스텀 이미지 제작 및 포트 매핑
### 7.1 Dockerfile 작성 및 이미지 빌드
Nginx 베이스 이미지를 활용하여 정적 웹 서버 커스텀 이미지를 빌드했습니다.
```bash
kty@kty my-web-server % docker build -t kty-web .
[+] Building 6.1s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => => naming to docker.io/library/kty-web
```
### 7.2 포트 매핑(-p) 실행 및 접속 확인
호스트의 8080 포트를 컨테이너의 80 포트와 연결했습니다.
```bash
kty@kty my-web-server % docker run -d -p 8080:80 --name kty-site kty-web
432e602a112f54ad97121efd078d6ddda72cd2794809f7da578bb495998a3fd7

kty@kty my-web-server % docker ps
CONTAINER ID   IMAGE     STATUS         PORTS                                     NAMES
432e602a112f   kty-web   Up 36 seconds  0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   kty-site
```
검증 방법: 브라우저에서 localhost:8080에 접속하여 정상 출력됨을 확인했습니다.

---
## 8. 바인드 마운트 및 볼륨 영속성 검증
### 8.1 바인드 마운트 (Bind Mount)
호스트의 디렉토리를 컨테이너 내부에 직접 연결하여 코드 수정 시 즉시 반영되는지 확인했습니다.
```bash
kty@kty my-web-server % docker run -d -p 8081:80 --name kty-mount -v "$(pwd)/index.html:/usr/share/nginx/html/index.html" nginx:alpine
```
검증 방법: 호스트에서 index.html을 수정한 뒤, 브라우저에서 localhost:8081 새로고침 시 변경 사항이 컨테이너 재시작 없이 실시간으로 반영됨을 확인했습니다.
### 8.2 Docker 볼륨을 이용한 데이터 영속성 검증
컨테이너 삭제 후에도 데이터가 유지되는지 검증했습니다.
```bash
# 1. 볼륨 생성 및 컨테이너 연결
kty@kty my-web-server % docker volume create kty-data
kty@kty my-web-server % docker run -d -p 8082:80 --name kty-vol-test -v kty-data:/usr/share/nginx/html nginx:alpine

# 2. 컨테이너 내부에 데이터 파일 생성
kty@kty my-web-server % docker exec -it kty-vol-test sh -c "echo 'Persisted Data' > /usr/share/nginx/html/save.txt"

# 3. 컨테이너 강제 삭제 (데이터 유실 환경 시뮬레이션)
kty@kty my-web-server % docker stop kty-vol-test
kty@kty my-web-server % docker rm kty-vol-test

# 4. 동일한 볼륨으로 새 컨테이너 실행 및 데이터 생존 확인
kty@kty my-web-server % docker run -d --name kty-vol-recovery -v kty-data:/usr/share/nginx/html nginx:alpine
kty@kty my-web-server % docker exec -it kty-vol-recovery cat /usr/share/nginx/html/save.txt
Persisted Data
```
검증 결과: 컨테이너가 삭제(rm)되었음에도, 별도 생성한 kty-data 볼륨에 기록된 save.txt 내용이 새로운 컨테이너에서 정상적으로 유지/조회되었습니다.

---
## 9. 트러블슈팅
```bash
🚨 Issue 1: 큰따옴표 미처리로 인한 dquote> 무한 대기
문제 상황: 터미널에서 echo "Hello, Codyssey!"memo.txt 입력 시, 프롬프트가 dquote>로 변경되며 명령이 종료되지 않았습니다.

원인: 닫히지 않은 큰따옴표(") 또는 띄어쓰기 구문 오류로 인해, 터미널(zsh)이 문자열 입력이 끝나지 않은 것으로 간주하고 추가 입력을 대기했습니다.

해결 방법: Ctrl + C를 입력하여 현재 명령을 강제 취소하고, echo "Hello" > memo.txt로 문법에 맞게 재입력하여 해결했습니다.

🚨 Issue 2: 오타 및 잘못된 플래그 입력으로 인한 실행 실패
문제 상황: ls =al 입력 시 zsh: al not found 에러 발생, dokcer images 입력 시 zsh: command not found 에러가 발생했습니다.

원인: CLI 환경의 엄격한 문법 규칙(띄어쓰기, 하이픈 - 사용)을 지키지 않거나 단순 철자 오류가 발생했습니다.

해결 방법: 옵션 플래그인 =를 -로 수정하고(ls -al), 오타를 정정(docker images)하여 정상 수행했습니다.
```

---
## 10. 학습 내용 요약 (과제 목표 달성도)
```bash
디렉토리 경로 및 권한 관리: 절대/상대 경로의 개념을 숙지하고, chmod 755를 통해 소유자(rwx)와 그 외 사용자(r-x)의 권한 차이를 직접 확인했습니다.

Docker 포트 매핑: 8080:80 설정을 통해 호스트의 특정 포트와 격리된 컨테이너 내부 포트를 연결하는 네트워크 통신 원리를 파악했습니다.

데이터 영속성 보장: 컨테이너의 휘발성(Stateless) 특징을 이해하고, 바인드 마운트와 Docker Volume을 사용하여 안전하게 데이터를 외부(호스트)에 보관하는 방법을 완벽히 검증했습니다.
```