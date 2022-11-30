# docker

# How To Use

Dockerfile → build → image → docker run → container

docker ps

docker ps -a

docker images

docker rm {container name}

docker rmi (image name}

docker build -t {image name}:{version} .

ex)docker build -t feedback-node:volumes .

docker run -d -p 3000:80 —rm —name {container name} -v {volume name}:/app/feedback {image name}:{version}

ex)docker run -d -p 3000:80 —rm —name feedback-app -v feedback:/app/feedback feedback-node:volumes

Dockerfile : FROM node:14, WORKDIR /app, COPY package.json ., RUN npm install, COPY . ., EXPOSE 80, CMD [”node”, “server.js”]

-v : :앞에 이름이 들어가면 바운드 마운트가 되고 없으면 볼륨으로 취급되어 명명된 볼륨이 된다.

Dockerfile에 VOLUME [ "/app/node_modules" ]와 같은 방식으로 volume을 추가할 수도 있지만 docker run 할때 -v /app/node_modules와 같은 방식도 취할 수 있다.

docker run -d --rm -p 3000:80 --name feedback-app -v feedback:/app/feedback -v "C:\docker\DOCKER-COMPLETE:/app" -v /app/node_modules feedback-node:volumes

# **EXPOSE & 약간의 유틸리티 기능**

지난 강의에서는 포트(포트 80)를 **노출**하는 컨테이너로 시작했습니다.

결국 Dockerfile의 '`EXPOSE 80`'은 선택 사항이라는 점을 다시 한 번 명확히 하고 싶습니다. 그것은 컨테이너의 프로세스가 이 포트를 노출할 것임을 **문서화**하는 겁니다. 하지만 '`docker run`'을 실행할 때 '`-p`'를 사용하여 실제로 포트를 노출해야 합니다. 따라서 기술적으로 '`-p`'는 포트에서 수신 대기할 때 **유일하게 필요한 부분**입니다. 하지만 Dockerfile에 'EXPOSE'를 추가하여 이 동작을 문서화하는 것이 **모범적인 사용법**입니다.

추가 **참고 사항**: **ID**를 사용할 수 있는 모든 **docker 명령**의 경우, **항상 전체 ID를 복사**/기록할 필요는 **없습니다**.

첫 번째(몇 개) 문자를 사용할 수도 있습니다. 고유 식별자를 갖는 것만으로도 충분합니다.

그래서

`1. docker run abcdefg`

이 명령 대신

`1. docker run abc`

이 명령을 실행할 수도 있습니다.

또는 "a"로 시작하는 다른 이미지 ID가 없으면, 다음과 같은 명령을 실행할 수도 있습니다.

`1. docker run a`

**이는 ID를 필요로 하는 모든 Docker 명령어에 적용됩니다.**

# **이미 실행 중인 컨테이너에 연결하기**

디폴트로 '`-d`' 없이 컨테이너를 실행하면, "attached모드"로 실행됩니다.

detached 모드(예: `-d`)로 컨테이너를 시작한 경우에는 다음 명령을 사용하여 컨테이너를 다시 시작하지 않고도 컨테이너에 연결할 수 있습니다.

`1. docker attach CONTAINER`

이는 `CONTAINER`라는 ID 또는 이름으로 실행 중인 컨테이너에 연결합니다.

# **익명 볼륨 제거하기**

컨테이너가 제거되면, 익명 볼륨이 **자동으로 제거**되는 것을 보았습니다.

이는 '`--rm`' 옵션으로 컨테이너를 시작/실행할 때 발생하게 됩니다.

**그 옵션 없이** 컨테이너를 시작하면, 컨테이너를 (`docker rm ...` 으로) 제거하더라도 익명 볼륨은 **제거되지 않습니다.**

그래도 컨테이너를 다시 만들어, 다시 실행하면(즉, docker run ... 다시 실행), **새 익명 볼륨이 생성됩니다.** 즉, 익명 볼륨이 자동으로 제거되지 않았지만, 다음에 컨테이너가 시작될 때, 다른 익명 볼륨이 연결되기 때문에, 이전 컨테이너를 제거하고 새 컨테이너를 실행하는데 **도움이 되지 않습니다.**

이제 **사용하지 않는 익명 볼륨이 쌓이기** 시작합니다. '`docker volume rm VOL_NAME`' 또는 '`docker volume prune`'을 통해 그러한 **볼륨을 삭제**할 수 있습니다.

# **바인드 마운트 – 바로 가기(Shortcuts)**

간단한 참고 사항: 항상 전체 경로를 복사하여 사용하고 싶지 않은 경우, 다음 바로 가기를 사용할 수 있습니다.

macOS / Linux: `-v $(pwd):/app`

Windows: `-v "%cd%":/app`

모든 분께 적용되는 접근 방식을 보여드리고자 하기 때문에, 강의에서는 이 바로가기를 사용하지 않습니다(매번 둘 사이를 전환하고 싶지는 않음). 하지만 작업 중인 OS에 따라, 타이핑을 절약하기 위해 이 바로 가기를 사용할 수 있습니다.

# Volume & Bind Mount

docker run -v /app/data … → Anonymous Volume

특징 : 컨테이너가 사라지면 익명의 볼륨도 제거됨. 그러므로 컨테이너 간에 데이터를 공유할 수 없다. 하지만 컨테이너에 이미 존재하는 특정 데이터를 잠그는 것에는 유용. 예를들어 /app/node_moduls와 같은 데이터가 다른 모듈에 의해 덮어쓰여지는 것을 방지. 이것은 Dockerfile에서 “VOLUME”라는 키워드로도 생성 가능. 이건 컨테이너 내부에 존재하지 않고 호스트에 관리됨.

docker run -v data:/app/data … → Named Volume

특징 : 명명된 볼륨은 Dockerfile에서 생성 불가. 어느 컨테이너와도 연결X. 컨테이너를 종료하고 제거해도 살아남아있음. docker cli로 제거는 가능. 여러 컨테이너간 데이터 공유 가능. 다수의 다양한 컨테이너에 동일하게 명명된 볼륨 하나를 마운트 가능.

docker run -v /path/to/code:/app/code … → Bind Mount

특징 : 컨테이너 종료 및 제거후에도 남아있음. bind mount데이터를 지우려면 호스트머신에서 삭세해야함. 컨테이너간 데이터 공유 가능. 재빌드안해도 라이브 데이터 제공 가능(리빌드 필요X)

# COPY사용 vs bind mount 사용

개발 중일때는 Dockerfile에 COPY를 빼서 사용할 수 있다. bind mount가 자동으로 해주니까

근데 product단계에서는 bind mount를 할리가 없으니 COPY가 필요한 것

# **.dockerignore 파일에 더 추가하기**

`.dockerignore` 파일에 "무시할" 파일 및 폴더를 더 추가할 수 있습니다.

예를 들어, 항목에 다음을 추가하는 것을 고려하세요.

`1. Dockerfile
2. .git`

이것은 잠재적으로 존재하는 `.git` 폴더와 `Dockerfile` 자체를 무시합니다(프로젝트에 Git을 사용하는 경우).

일반적으로 애플리케이션이 올바르게 실행되는데 필요없는 모든 것을 추가하고자 할 겁니다.

# .dockerignore & .env & arg

.dockerignore : .gitignore와 비슷함(ex. node_modules)

.env : 환경변수 (PORT=8000) docker run -p 3000:8000 —env-file ./.env, Dockerfile에 ENV PORT 8000과 같이 입력

arg : Dockerfile에 ARG DEFAULT_PORT=80이라 입력 후 이미지 만들 때 docker build -t feedback-node:dev --build-arg DEFAULT_PORT=8000 .

# 컨테이너와 호스트머신과의 통신

ex) mongodb://localhost:27017/swfavorites → mongodb://host.docker.internal:27017/swfavorites

# 컨테이너와 컨테이너의 통신

docker network create {networkName}

docker run —network {networkName}

docker run 할때 -p를 안해도 된다. -p를 할때는 컨테이너와 호스트 연결이나 컨테이너와 외부연결할 때 필요함

# **Docker 네트워크 드라이버**

Docker Networks는 실제로 네트워크 동작에 영향을 미치는 다양한 종류의 '**드라이버**'를 지원합니다.

디폴트 드라이버는 '**bridge**' 드라이버입니다. 이 드라이버는 모듈에 나타난 동작을 제공합니다 (즉, 컨테이너가 동일한 네트워크에 있는 경우, 이름으로 서로를 찾을 수 있음).

드라이버는 네트워크 생성 시 '`--driver`' 옵션을 추가하여 간단히 설정할 수 있습니다.

`1. docker network create --driver bridge my-net`

물론 'bridge' 드라이버를 사용하고자 하는 경우, 'bridge'가 디폴트이므로, 전체 옵션을 생략하면 됩니다.

Docker는 아래의 대체 드라이버도 지원하지만 대부분의 경우 'bridge' 드라이버를 사용합니다.

- **host**: 스탠드얼론 컨테이너의 경우, 컨테이너와 호스트 시스템 간의 격리가 제거됩니다 (즉, localhost를 네트워크로 공유함).
- **overlay**: 여러 Docker 데몬 (즉, 서로 다른 머신에서 실행되는 Docker)이 서로 연결될 수 있습니다. 여러 컨테이너를 연결하는 구식의 / 거의 사용되지 않는 방법인 'Swarm' 모드에서만 작동합니다.
- **macvlan**: 컨테이너에 커스텀 MAC 주소를 설정할 수 있습니다. 그러면 이 주소를 해당 컨테이너와 통신하는데 사용할 수 있습니다.
- **none**: 모든 네트워킹이 비활성화됩니다.
- **써드파티 플러그인**: 모든 종류의 동작과 기능을 추가할 수 있는 타사 플러그인을 설치할 수 있습니다.

언급했듯이 '**bridge**' 드라이버는 대부분의 시나리오에 가장 적합합니다.

# mongodb 사용시 “데이터유지”와 “인증 보안” 관련

docker run --name mongodb -v data:/data/db --rm -d --network goals-net -e MONGO_INITDB_ROOT_USERNAME=max -e MONGO_INITDB_ROOT_PASSWORD=secret mongo

# Windows에서 live source update

nodemon : { “scripts”: “nodemon -L server.js” }

react : docker run -e CHOKIDAR_USEPOLLING=true ...other-options <react-image-id>

# MULTI-01-STARTING-SETUP 프로젝트 가이드

docker run --name mongodb -v data:/data/db --rm -d --network goals-net -e MONGO_INITDB_ROOT_USERNAME=max -e MONGO_INITDB_ROOT_PASSWORD=secret mongo

docker build -t goals-node .

docker run --name goals-backend -v C:\docker\multi-01-starting-setup\backend:/app -v logs:/app/logs -v /app/node_modules -e MONGODB_USERNAME=max -d --rm -p 80:80 --network goals-net goals-node

docker build -t goals-react .

docker run -e CHOKIDAR_USEPOLLING=true -v C:\docker\multi-01-starting-setup\frontend/src:/app/src --name goals-frontend --rm -p 3000:3000 -it goals-react

# **Linux에 Docker Compose 설치하기**

macOS 및 Windows에서는 Docker Compose가 이미 설치되어 있을 겁니다. Docker와 함께 설정되기 때문이죠.

Linux 시스템에서는 별도로 설치해야 합니다.

설치를 위해 다음 단계를 수행하세요.

1. `sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

2. `sudo chmod +x /usr/local/bin/docker-compose`

3. `sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

4. to verify: `docker-compose --version`

참조: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

# Docker Compose

1. docker-compose.yaml과 같이 파일명.yaml 형태로 파일 생성
2. yam파일안에 내용 구성
3. docker-compose up, docker-compose down과 같은 형태로 서비스를 올리고 내릴 수 있다.
4. docker-compose up -d : detach 모드로 서비스 올림
5. docker-compose down -v : volume도 내림(보통 안씀)

# Utility Container

예를들어 빈프로젝트에서 nodejs의 package.json을 구성해야 하는데 이건 “npm install”로 가능하다. 근데 만약 호스트PC에서 nodejs가 설치가 안되어있다면 package.json을 직접 타이핑하여 구성할 수 밖에 없다. 그러한 문제점을 해결하기 위해 Utility Container를 사용한다.

ex) docker run -it -v C:\docker\utility-container:/app node-util npm init

위와 같이 바운드마운트를 사용하여 호스트PC에 package.json을 생성할 수 있다.

Dockerfile : ENTRYPOINT [ "npm" ]을 사용하면 docker run -it -v C:\docker\utility-container:/app node-util init 처럼 init앞에 npm 제거 가능(그다지 필요 없을 거 같은데?)

종속성 추가 : docker run -it -v C:\docker\utility-container:/app mynpm install express --save(express 설치)

docker compose사용가능하다. (docker-compose run --rm npm init)