멀티스테이지 빌드 -> ./frontend/Dockerfile.prod 참조
멀티스테이지빌드란 FROM을 2번 이상 사용하여 최종적인 이미지를 얻기 위한 과정이라고 생각하면 됨.

docker build -f frontend/Dockerfile.prod -t kbjbest/goals-react ./frontend

docker push kbjbest/goals-react

같은 작업(task)안에 같은 포트(ex. 80)가진 컨테이너를 2개 이상 만드는 것은 불가(backend container port 80, frontend container port 80)
그러므로 아예 새 작업을 만들어야 한다.
A작업 -> backend container
B작업 -> frontend container

멀티스테이지에서 빌드 target
docker build --target build -f frontend/Dockerfile.prod ./frontend
앞에껏만 빌드