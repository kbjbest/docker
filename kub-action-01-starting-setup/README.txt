docker build -t kub-first-app .

kubectl create deployment first-app --image=kub-first-app
->deployment.apps/first-app created

kubectl get deployments
->  NAME        READY   UP-TO-DATE  AVAILABLE   AGE
    first-app   0/1     1           0           27s
    READY 0/1 의미 : 한 개의 deployment 중 하나가 실패하였음을 의미

kubectl get pods : deployment에서 생성된 모든 것을 보자.
->  NAME                        READY   STATUS          RESTARTS    AGE
    first-app-85df859fb4-5zp2k  0/1     ErrImagePull    0           60s
    READY의미 : 타깃상태라 할 수 있고 Current State : 0, Target State : 1, 포드 한 개가 생성되지 않았다고
    STATUS : 이유는 ErrImagePull
    최종원인 : 로컬에만 image 존재하고 kubernetes클러스터 내부? 에는 image가 존재하지 않으므로 docker hub같은곳에 올려줘야 됨

kubectl delete deployment first-app
->deployment.apps "first-app" deleted

kubectl get pods
->No resources found in default namespace.

docker hub에서 저장소 새로 생성
저장소 이름 : kub-first-app

docker tag kub-first-app kbjbest/kub-first-app

docker push kbjbest/kub-first-app

다시 시작!

kubectl create deployment first-app --image=kbjbest/kub-first-app
->deployment.apps/first-app created

kubectl get deployments
->  NAME        READY   UP-TO-DATE  AVAILABLE   AGE
    first-app   1/1     1           1           5s

kubectl get pods
->  NAME                        READY   STATUS  RESTARTS    AGE
    first-app-69b8b97f95-8gc45  1/1     RUNNING 0           22s

minikube dashboard -> 이걸로 확인해보자.

Service 객체
pod는 만들어질떄 자동으로 내부IP를 생성하는데 이게 고정이 아니라 매번 바뀜
그래서 service객체를 사용하여 외부와 연결할 수 있는 고정 IP를 제공함
그러므로 service객체는 필요

kubectl expose deployment first-app --type=LoadBalancer --port=8080 : expose를 사용하여 service를 생성할 수 있다. LoadBalancer는 자동으로 여유있는 pod에 분배해줌
-> service/first-app expoed

kubectl get services
->  NAME        TYPE            CLUSTER-IP      EXTERNAL-IP     PORT(S)         AGE
    first-app   LoadBalancer    10.98.181.101   <pending>       8080:31321/TCP  10s
    kubernetes  ClusterIP       10.96.0.1       <none>          443/TCP         23h
    맨 밑에 kubernets는 자동으로 생성된 default service

minikube service first-app
->  NAMESPACE   NAME        TARGET PORT     URL
    default     first-app   8080            http://192.168.99.100:31321
    Opening service default/first-app in default browser...
    URL을 통해 브라우저에서 접근할 수 있으며 이 어플리케이션은 우리가 생성하여 쿠버네티스 클러스터로 보낸 deployment를 기반으로 쿠버네티스에 의해 생성한 pod에서 실행

kubectl scale deployment/first-app --replicas=3
->deployment.apps/first-app scaled

kubectl get pods
->  NAME                        READY   STATUS  RESTARTS    AGE
    first-app-69b8b97f95-8gc45  1/1     Running 2           20m
    first-app-69b8b97f95-hj5xf  1/1     Running 2           6s
    first-app-69b8b97f95-xdwtc  1/1     Running 2           6s

deployment 업데이트
docker build -t kbjbest/kbu-first-app:2 .
docker push kbjbest/kub-first-app
kubectl set image deployment/first-app kub-first-app=kbjbest/kub-first-app:2 : deployment 업데이트
kubectl rollout status deployment/first-app : 업데이트 상태
kubectl get deployments

deployment 롤백&히스토리
kubectl set image deployment/first-app kub-first-app=kbjbest/kub-first-app:3 : 이건 실패할꺼 이미지를 tag3에 대한 이미지를 안만들었기 때문에
->deployment.apps/first-app image updated
kubectl rollout status deployment/first-app : deployment에서 현재 진행중인 작업을 알려줌
->Waiting for deployment "first-app" rollout to finish: 1 old replicas are pending termination... : 완료되기를 기다림. 그리고 오래된 레플리카, 현재 실행중인 pod가 종료되고 있다.
kubernetes에서는 새로운 것이 실패하기 때문에(tag3) 전에 pod를 제거할 수 없는 상황임(정상)
그래서 롤백을 해야함
kubecdtl get pods
->  NAME                        READY   STATUS              RESTARTS    AGE
    first-app-5bf786b767-scx4r  1/1     Running             0           5m29s
    first-app-64f44b744d-xgg6f  0/1     ImagePullBackOff    0           2m31s : 이미지를 찾지 못하여 실패
kubectl rollout undo deployment/first-app
->deployment.apps/first-app rolled back
kubecdtl get pods
->  NAME                        READY   STATUS              RESTARTS    AGE
    first-app-5bf786b767-scx4r  1/1     Running             0           6m11s
kubectl rollout status deployment/first-app
->deployment "first-app" successfully rolled out
여기까지가 롤백
지금부터 히스토리
kubectl rollout history deployment/first-app
->  deployment.apps/first-app
    REVISION    CHANGE-CAUSE
    1           <none>
    3           <none>
    4           <none>
kubectl rollout history deployment/first-app --revision=3 : REVISON 3에 대한 상세정보
kubectl rollout history deployment/first-app --revision=1 : 위에꺼랑 비교하면 다른게 있음
kubectl rollout undo deployment/first-app --to-revision=1 : 첫번째 deployment로 돌아감
->deployment.apps/first-app rolled back

명령적 접근방식 vs 선언적 접근방식
하기 전 다 삭제
kubectl delete service first-app
->service "first-app" deleted
kubectl delete deployment first-app
deployment.apps "first-app" deleted
minikube dashboard에도 다 삭제된걸 확인할 수 있음
명령적 접근방식은 docker run 처럼 CLI에다가 명령어를 입력해서 실행하는 궤
선언적 접근방식은 docker-compose처럼 파일에 기록하고 실행하는 궤
kubernetes에도 이와같은게 있음

배포 구성 파일 생성하기(선언적 접근방식)
kubernetes 공식문서를 꼭 참조하자
apiVersion은 최신꺼를 사용하면 된다.(공식 docs 참조)
kind도 공식문서 참고(정해져있음)
metadata-name : deployment 이름으로써 사용자지정

Pod와 컨테이너 사양(Specs)추가
spec은 매우 중요
spec-replicas : pod의 갯수라 보면 됨. 안쓰면 default 1. 0으로 지정해서 pod를 생성안할수도 있음.
kubectl apply -f=deployment.yaml
->error: error validating "deployment.yaml": error validating data: Validation Error(Deployment.spec): missing required field "selector" in io.k8s.api.apps.v1.DeploymentSpec; if you choose to ignore these errors, turn Validation off with --validate=false
에러가 발생하는데 selector의 개념이 중요

Label 및 Selector로 작업하기
deployment.yaml파일처럼 deployment에게 관리해야할 pod의 key, value를 알려줘야함 Selector:matchLables 참고
kubectl apply -f=deployment.yaml
->deployment.apps/second-app-deployment created
kubectl get deployment
->  NAME                    READY   UP-TO-DATE  AVAILABLE   AGE
    second-app-deployment   1/1     1           1           7s
kubectl get pods
->  NAME                                    READY   STATUS  RESTARTS    AGE
    second-app-deployment-7f794d896f-vrvpr  1/1     Running 0           14s

선언적으로 Service 만들기
service.yaml 참고
kubectl apply -f service.yaml : 이 시점에서 어플리케이션이(서비스) 실행됨
->service/backend created
kubectl get services
minikube service backend

리소스 업데이트 & 삭제
예를들어 deployment.yaml에서 replicas 3으로 고친 후 
kubectl apply -f=deployment.yaml
->deployment.apps/second-app-deployment configured
kubectl get pods
->pod 3개 출력
kubectl delete -f=deployment.yaml, service.yaml 또는 kubectl delete -f=deployment.yaml -f=service.yaml

다중 vs 단일 Config 파일
master-deployment.yaml 참조 
---   대시 3개로 구분

Label & Selector에 대한 추가 정보
deployment.yaml 참고 matchExpressions로 보다 더 파워풀하고 유연하게 사용할 수 있고 간단할 때는 matchLabels를 사용
label로도 삭제가능
kubectl delete deployments,services -l group=example
->deployment.apps "second-app-deployment" deleted
  service "backend" deleted

  활성 프로브(Liveness Probes)
  deployment.yaml 참조