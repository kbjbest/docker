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