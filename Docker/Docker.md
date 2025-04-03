### Docker

#### Docker Desktop
- 도커를 윈도우나 맥에서도 쉽게 쓸 수 있게 해주는 앱이자 도커 데몬을 실행해주는 도구
- 역할
  - 윈도우에는 리눅스 커널이 없어서 도커가 바로 실행 안됨

### Docker 구조
1. `which docker`
   - 도커가 실제로 어디에 있는지 출력하면 아래와 같은 위치에 도커가 존재하고 있다.
   - `/usr/bin/docker`
   - 즉, docker 명령어는 위 파일을 통해 사용되고 있다는 의미
   - 그렇다면 이번에는 실행중인 도커 프로세스를 아래 명령어로 확인하면
   - `ps aux | grep docker`
   ```
   root       307  0.0  0.5 2803304 84180 ?       Ssl  10:55   0:04 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
   root       500  0.0  0.1 1247540 21436 pts/4   Ssl+ 12:56   0:00 /mnt/wsl/docker-desktop/docker-desktop-user-distro proxy --distro-name Ubuntu --docker-desktop-root /mnt/wsl/docker-desktop C:\Program Files\Docker\Docker\resources
   ks       55980  0.0  0.0   4092  2020 pts/9    S+   16:30   0:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox --exclude-dir=.venv --exclude-dir=venv docker
    ```
   - 위에서 도커가 위치된 `/usr/bin/docker`가 실행되고 있어야 하는데 `/usr/bin/dockerd` 가 실행 중이다.

### Docker daemon
   - api 요청을 수신하고 이미지, 컨테이너, 네트워크 및 볼륨과 같은 docker 객체를 관리한다.

### Docker client
   - Docker client 는 많은 Docker 사용자가 Docker와 상호작용하는 기본 방법이다.
   - `docker run` 같은 명령어를 사용하면 클라이언트가 이러한 명령을 API로서 ***dockered*** 로 보낸다.
   - 이때 Docker client는 /var/run/docker.sock에 있는 Unix Socket을 통해 Docker daemon의 API를 호출한다.
   - Docker client가 사용하는 Unix Socket은 같은 호스트 내에 있는 Docker daemon에게 명령을 전달할 때 사용된다.
   - tcp로 원격에 있는 Docker daemon을 제어하는 방법도 있다.

### Docker registries
   - Docker registries는 Docker image 를 저장한다.
   - Docker Hub는 누구나 사용할 수 잇는 공용 레지스트리고 Docker는 기본적으로 Docker Hub에서 Docker image를 찾는다.
   - Docker는 클라이언트-서버 구조를 가진다. Docker client는 도커 컨테이너를 빌드, 실행 및 배포에 대한 무거운 작업을 수행하는 Docker daemon과 통신을 한다.
   - Docker client와 Docker Daemon은 동일한 시스템에서 실행되거나 Docker client를 원격 Docker daemon에 연결할 수 있다.
   - 이런 Docker Client 와 Docker daemon은 Unix Socket or Network interface를 통해 REST API를 사용하여 통신한다.