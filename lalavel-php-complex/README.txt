1. 라라벨 프로젝트 구성
docker-compose run --rm composer create-project --prefer-dist laravel/laravel .
2. Docker Compose로 각각의 컴포넌트 구동
docker-compose up -d server php mysql
3. depends_on
docker-compose에 depends_on이 있다면 "docker-compose up -d server"만 하여 server 컨테이너만 실행하여도 의존하기 때문에 depends_on에 있는 서비스(컨테이너)들도 실행
4. --build 옵션
docer-compose up은 이미지가 이미 존재한다면 Dockerfile자체를 수정하여도 전에 이미지를 그대로 사용하므로 리빌드(docker build와 같은)하면서 up을 하고싶으면 
docker-compose up -d --build server와 같은 방법을 사용
5. artisan
docker-compose run --rm artisan migrate