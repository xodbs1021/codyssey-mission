# 미션 1: 개발 워크스테이션 구축

## 1. 프로젝트 개요
본 프로젝트는 특정 로컬 머신에 종속되지 않고, 팀원 누구나 동일한 방식으로 실행 및 배포할 수 있는 **재현 가능한 개발 워크스테이션 환경**을 구축하는 것을 목표로 합니다. 리눅스 CLI 조작, Docker 컨테이너 격리 및 실행, 볼륨을 통한 데이터 영속성 검증 과정을 수행했습니다.

---

## 2. 실행 환경
* **OS**: macOS (Apple Silicon / aarch64)
* **Shell / Terminal**: zsh
* **Docker**: Docker version 28.5.2, build ecc6942 (OrbStack 환경)
* **Git**: [여기에 본인의 Git 버전을 적어주세요. 예: git version 2.39.3]

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