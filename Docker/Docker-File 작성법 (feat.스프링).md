## 도커파일 포맷
- 스프링 부트의 공식 사이트 가이드에서 스프링 부트와 Docker 를 연동하기 위해 다음과 같은 Dockerfile 포맷을 올려놓았다.
```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
  - 링크 : https://spring.io/guides/gs/spring-boot-docker/

- Gradle의 경우

``` $ docker build --build-args JAR_FILE=build/libs/*.jar -t springio/gs-spring-boot-docker . ```
  - 명령어를 입력하여 도커 이미지를 빌드한다.

- 그럼 도커 파일이 뭔지 자세하게 뜯어보자.

## Dockerfile
### Dockerfile 이란?

- Dockerfile은 **나만의 도커 이미지를 만들기 위해** 사용되는 파일이다.
- 우리가 만든 스프링 부트와 같은 웹 어플리케이션은 이미 도커 이미지들이 저장되어 있는 DockerHub에 등록되어 있지 않아서 Docker pull 명령어로 이미지를 받아올 수 없다. 
- 그래서 이미지를 직접 만들어줘야한다. 결국 이미지를 어떻게 만들건지 정의해놓은 파일이 바로 Dockerfile 이다.

### Dockerfile 의 내부
- 도커 파일의 내부는 이미지를 만들기 위해 사용되는 명령어들로 구성되어 있고, 순서대로 실행되어 이미지를 만들게 된다.
- 따라서 이 Dockerfile 명령어들을 알게되면 이미지가 어떻게 만들어 졌는지 이해할 수 있는 장점이 있다.

### 스프링 부트 가이드의 Dockerfile 명령어

``` 
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

---

- 스프링 부트 가이드에 기본적으로 도커파일 명령어들이 있고 하나하나 살펴보자.

#### 1. FROM
- Dockerfile 로 어떤 이미지를 만드려면 기반이 되는 이미지 레이어가 필요하다.
- 이때 `FROM <이미지 이름>:<태그>` 형식으로 작성하여 기반으로 하는 이미지를 설정할 수 있다. 
- ex) `FROM ubuntu:14.04`

- 우리는 자바 기반의 스프링 부트 어플리케이션을 실행해야 하기 때문에 자바 이미지를 기반으로 이미지를 만들어야 한다. 
- 따라서 `FROM bellsoft/liberica-openjdk-alpine:17` 명렁어는 '이 이미지는 알파인 리눅스에 사용되는 OpenJDK 17버전을 기반으로 만듭니다'라는 의미이다.
```dotenv
# Base 이미지 설정 (Alpine 기반 OpenJDK 17 사용)
FROM bellsoft/liberica-openjdk-alpine:17
```

#### 2. VOLUME
- Docker 에서 컨테이너를 삭제할 때, 컨테이너 `내부`의 데이터가 삭제되는데, `VOLUME 명령어를 사용할 경우` 컨테이너 외부와 컨테이너 내부를 연결시켜 저장된 데이터의 수명과 데이터를 생성한 컨테이너의 수명을 분리할 수 있다. 
- 디렉토리 하나를 사용할 경우, `VOLUME <컨테이너 디렉토리>` , 여러 디렉토리를 설정할 경우 `VOLUME ["컨테이너 디렉토리1", "컨테이너 디렉토리2"]` 와 같이 사용할 수 있다.
- `VOLUME /tmp` 로 설정할 경우, 컨테이너 내부의 /tmp 디렉토리가 호스트의 로컬 컴퓨터의 /var/lib/docker/volumes/${volume_name}/_data에 마운트 된다. (볼륨 이름은 임의의 해쉬값으로 생성)
- 하지만, mac이나 윈도우의 경우 /var/lib/docker/volumes 디렉토리가 없는데, 이는 mac과 윈도우의 경우 docker 를 바로 실행할 수 없어서 xhyve라는 가상머신을 하나 띄운 후 도커를 실행하기 때문에, /var/lib/docker/volumes 디렉토리가 가상머신 내에 감춰져 있기 때문이다.
- 참고 : https://joont92.github.io/docker/volume-container-%EC%B6%94%EA%B0%80%ED%95%98%EA%B8%B0/

#### 3. ARG
- ARG 명령어는 build 시에 사용되는 변수이다. 
- ARG 명령어는 `ARG <변수명>` 또는 `ARG <변수 명=값>` 형태로 설정해줄 수 있고, docker build 명령어로 사용할 경우 --build-arg 옵션으로 오버라이딩 할 수 있다. 
- `ARG JAR_FILE=target/*.jar` 명령어는 'target/*.jar 이라는 값을 가진 JAR_FILE 변수를 사용하겠다'라는 것을 나타낸다. 
- target/*.jar 는 Maven 빌드 환경에서 빌드된 JAR 파일이 저장되는 디렉토리 이다.

#### 4. COPY
- COPY 명령어를 사용하면 파일이나 디렉토리를 복사하여 컨테이너 디렉토리에 추가해준다. 
- `COPY <복사할 경로> <이미지에 추가할 경로>` 형식으로 사용된다. 
- `COPY ${JAR_FILE} app.jar` 명령어는 'JAR_FILE 변수에 저장된 JAR 파일을 컨테이너의 app.jar 경로에 추가한다' 라는 의미이다.

#### 5. ENTRYPOINT
- ENTRYPOINT 는 해당 컨테이너가 시작되었을 때 수행할 실행 명령을 정의하는 명령어이다. Dockerfile 파일 내에서 1번만 정의가 가능하다.
- `ENTRYPOINT <실행할 명령어>` 혹은 `ENTRYPOINT ["실행 파일", "매개 변수 1", "매개 변수 2", ...]` 형식으로 사용할 수 있다. 
``` ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"] ```

- 명령어를 실행하면 컨테이너가 시작되면 java -Djava.security.egd=file:/dev/./urandom -jar /app.jar 쉘 명령어를 실행 한다. 
- 자바에서 DB 연결 시에 JDBC Driver 에서 암호화 작업을 하는데, 이 과정에서 난수 발생이 일어난다. 
- 자바는 기본적으로 /dev/random 을 사용하는데, /dev/random 을 사용할 경우 랜덤 값을 만들어 낼 때 엔트로피가 충분해질때까지 Blocking 이 있기 때문에 간헐적으로 지연되어서 에러가 발생할 수 있다. 
- 이 과정에서 /dev/random 대신 /dev/urandom 을 사용하면 현재 존재하는 엔트로피로만 사용하기 때문에 Blocking 을 하지 않아 지연 현상이 발생하지 않는다. 
- 따라서 Djava.security.egd=file:/dev/./urandom 을 매개 변수로 입력해준다. 
- 참고 : http://blog.naver.com/PostView.nhn?blogId=hanccii&logNo=220725686080
- 참고 : https://waspro.tistory.com/254

### 도커 이미지 빌드
`$ docker build --build-args JAR_FILE=build/libs/*.jar -t springio/gs-spring-boot-docker .`
- 위 명령어는 도커 이미지 빌드 명령어 이다.
- `docker build [OPTION] <Dockerfile 이 있는 PATH>`
  - 위와 같은 형태이며 현재 도커 이미지 빌드 명령어를 실행하는 디렉토리에 Dockerfile 이 있는 경우 경로로 "." 를 입력하면 된다.

#### 옵션
1. `--build-args`
- 이전 Dockerfile 파일의 ARG 명령어로 선언했었다. (바로 위 참고)
- Dockerfile 내의 JAR_FILE 경로가 Maven 빌드 환경의 경로이므로 Gradle 빌드 환경에서 JAR 파일 경로를 입력해줘야 한다.

- `build/libs/*.jar` 을 입력하여 Dockerfile 내에서 JAR_FILE 변수의 값을 오버라이딩해주는 역할을 한다.

2. `-t`
- 생성될 도커 이미지의 이름을 지정해주는 옵션이다. 
- 빌드를 하면 springio/gs-spring-boot-docker 라는 이름의 이미지를 생성해준다.