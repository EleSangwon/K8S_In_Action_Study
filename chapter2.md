# Chapter2 - 도커와 쿠버네티스의 첫걸음

## Hello World 컨테이너 실행
```
docker run busybox echo "Hello world" 명령어를 수행하면,
도커는 busybox 를 통해 최신 이미지 busybox:latest 가 로컬 시스템에 제공되는 지 확인한다.

최신 이미지가 아니므로, 도커는 도커 허브 레지스트리에서 최신 이미지를 가져온다.

이미지를 가져와서 다운로드한 후, 도커는 해당 이미지에서 컨테이너를 생성하고 
그 안에서 명령을 실행한다.

echo 명령은 텍스트를 STDOUT에 출력한 다음 프로세스가 종료되고 컨테이너가 중지된다.
```
![KakaoTalk_20210803_144607949](https://user-images.githubusercontent.com/50174803/127964383-b9864794-fc3f-48f1-98fc-80ce8e27793c.jpg)

## 이미지 실행 & 이미지 버전
```
docker run <image>

docker run <image>:<tag>
```

## 간단한 Node.js 앱 만들기
![챕터22](https://user-images.githubusercontent.com/50174803/127967862-6ec288d8-4605-4ae9-b363-123aa27a15fd.PNG)

## 이미지 레이어의 이해
```
이미지는 크기가 큰 단일 바이너리 덩어리가 아니라, 여러 개의 레이어로 구성된다. 
여러 이미지에서 레이어를 공유할 수 있으므로 이미지 저장 및 전송이 훨씬 효율적이다.
```

## 컨테이너 이미지 실행
```
docker run --name sangwon-container -p 8080:8080 -d sangwon

도커가 sangwon 이라는 이미지에서 sangwon-container 라는 새 컨테이너를 실행하도록 명령한다.

-d 플래그는 컨테이너가 콘솔에서 분리됨을 말하며 이는 백그라운드에서 실행됨을 의미한다.

로컬 컴퓨터의 포트 8080은 컨테이너 내부의 포트 8080에 매핑되므로
localhost:8080 을 통해 애플리케이션에 액세스할 수 있다.
```
![캡처33](https://user-images.githubusercontent.com/50174803/127968352-b05d57a3-6f34-4e47-bd48-eccd7eb3c8e9.PNG)

## 컨테이너 관한 추가 정보 얻기
```
docker inspect <container이름>
```

## 실행 중인 컨테이너 내부 탐색
```
docker exec -it sangwon-container bash 

내부에서 컨테이너를 탐색 시, 컨테이너에서 실행 중인 프로세스만 확인가능하고 호스트 OS의 다른 프로세스는
볼 수 없다.
```
![챕터44](https://user-images.githubusercontent.com/50174803/127968696-6fb30a2c-d6e2-411f-a068-93b9be0e4bf1.png)

## 컨테이너 중지 및 제거
![챕터55](https://user-images.githubusercontent.com/50174803/127969125-7083267a-6abb-466e-be17-3cbf80aeb260.PNG)

## 이미지 레지스트리로 이미지 푸시
```
추가 태그로 이미지 태그 지정
docker tag sangwon sangwondockerhub/sangwon

도커로그인
docker login

도커허브에 이미지 푸시 
docker push sangwondockerhub/sangwon

다른 머신에서 이미지 실행
docker run -p 8080:8080 -d sangwondockerhub/sangwon
```
![캡처66](https://user-images.githubusercontent.com/50174803/127969623-ce652051-d3d6-4df3-a759-0210efbb7a37.png)
![캡처77](https://user-images.githubusercontent.com/50174803/127970070-ae150510-30f7-4c02-aaae-ed3fe61352dd.PNG)

## 쿠버네티스 클러스터와 상호작용 하는 방법
![캡처88](https://user-images.githubusercontent.com/50174803/127970945-f49ff5a3-aad1-459b-9245-d3c90f9f7791.jpg)

## 쿠버네티스로 앱 배포
```
Pod 생성 시, 계속 Pending 이여서 describe 로 확인해보니 한 개의 노드에 할당가능한 파드 개수를 초과해서
오토 스케일링으로 노드를 2개 생성하여 Pending - > Running 되도록 만듬
```
![캡처99](https://user-images.githubusercontent.com/50174803/127971833-d3ae2715-de38-4c52-807b-b5a266254c73.png)
![캡처10](https://user-images.githubusercontent.com/50174803/127971842-d921f04f-6cd9-4275-b67a-c2618f681c7e.png)

## 파드
```
파드는 하나 이상의 밀접하게 관련된 컨테이너로 구성된 그룹을 의미한다.
동일한 리눅스 네임스페이스와 동일한 워커 노드에서 항상 함꼐 실행된다.

각 파드는 애플리케이션을 실행하는 자체 IP, 호스트 이름, 프로세스 등이 있는
별도의 논리적 시스템이다. 

애플리케이션은 단일 컨테이너로 실행되는 단일 프로세스일 수도 있고 
주요 애플리케이션 프로세스 및 각각의 자체 컨테이너에서 실행되는 지원 프로세스 일 수도 있다.
```
## 컨테이너, 포드, 워커노드 간 관계
![캡처11](https://user-images.githubusercontent.com/50174803/127973036-bd08941e-32e1-4532-8c2b-069ac71f6ded.jpg)

## 화면 뒤에 숨겨진 내용 이해
```
- 쿠버네티스 내부에서 컨테이너 이미지를 실행하려면 거쳐야 하는 단계

먼저 이미지를 제작해 도커허브에 푸시한다 (혹은 aws ecr 등의 이미지 리포지토리 사용)

kubectl 명령어를 실행해 쿠버네티스 API 서버에 REST HTTP 요청을 전송해 클러스터에
새로운 래플리카 객체를 만든다.

래플리카 객체는 새로운 파드를 생성하고 스케줄러에 의해 워커 노드 중 하나에
스케줄링된다.

마스터 노드로부터 kubelet 은 통지를 받고 kubelet이 도커에게 이미지를
실행하도록 한다.

도커는 이미지를 풀하고 실행한다.
```
![캡처12](https://user-images.githubusercontent.com/50174803/127974301-17606fda-20f2-425a-bfe3-ab21dc86abbd.jpg)

## 애플리케이션 액세스
```
실행 중인 파드를 어떻게 액세스할 수 있을까? 
각 파드에는 자체 IP주소가 있지만 이 주소는 클러스터 내부에 있으며 외부에서 액세스 할 수 없다.

외부에서 액세스할 수 있도록 하려면 서비스 객체를 통해(Kind: Service) IP를 노출해야 한다.

파드와 같은 일반 서비스(ClusterIP 서비스)를 만들면 클러스터 내부에서만 액세스할 수 있기 떄문에
LoadBalancer 형태의 서비스를 만든다.

LB 형태의 서비스를 만들면 외부 로드 밸런서가 생성되므로 로드 밸런서의 공용 IP를 통해
파드에 연결할 수 있다.

+ 내가 알고 있는 지식 더하기

Kind : Service에는 여러 가지 type이 있는 데 NodePort 으로 노출시켜 외부에서 액세스할 수 있다.

AWS 를 사용한다고 가정한다.
예를 들어, 워커노드 A에 프론트앤드 파드가 있다.

이 프론트 앤드 파드를 서비스 객체 + NodePort 타입으로 노출시키고
노드포트를 3000으로 설정한다.

그리고 워커노드 A (EC2) 에 보안그룹 (SG) 규칙 inbound 에 3000번 포트를 추가하면
워커노드A의 퍼블릭 IP:3000 으로 서비스 객체에 액세스할 수 있다.

그러나, 이 방법은 보안상 취약하기 떄문에 테스트 시에만 주로 사용한다.
```

## 서비스 객체의 생성
```
k8saction 이라는 네임스페이스에 있는 리소스를 확인한다.
kubectl -n k8saction get all

해당 네임스페이스에 존재하는 파드를 로드밸런서 타입으로 sangwon-service라는 이름으로 생성한다
kubectl -n k8saction expose pod sangwon-container --type=LoadBalancer --name sangwon-service

k8saction 이라는 네임스페이스에 있는 리소스를 확인한다.
서비스가 추가됨을 확인할 수 있다
kubectl -n k8saction get all

describe를 사용하여 자세히 들여다본다.
kubectl -n k8saction describe svc sangwon-service

Endpoints 가 192.168.2.221:8080 이다 

파드의 IP는 192.168.2.221 이다.

curl <외부 IP>:포트로 결과를 확인한다.
```
![캡처13](https://user-images.githubusercontent.com/50174803/127976462-666df294-8a77-4db6-9d7f-10aec131466b.PNG)

## 서비스에 관한 테스트 - 파드
```
sangwon-container 라는 파드를 expose 시켜 sangwon-service 라는 서비스를 만들었고 
그 서비스는 curl <외부 IP>:포트로 액세스 가능한 상태이다.

이제 sangwon-container 라는 파드를 지우고 남아있는 서비스 객체로 curl 명령어를 날리면
결과를 받아올 수 없다.

동일한 이미지의 파드를 재생성하고 curl 명령어를 날려도 역시 결과를 받아올 수 없다.

describe로 service를 확인해보니, Endpoints가 정의되어 있지 않아 동일한 파드를 재생성해도
결과를 확인할 수 없다.

만약 서비스로 결과를 받아오려면 재생성한 파드를 서비스 객체로 노출시켜야 한다.

```
![캡처14](https://user-images.githubusercontent.com/50174803/127978439-30c54510-c093-4f5e-bc8f-9961c743631d.PNG)

## 서비스에 관한 테스트 - 디플로이먼트
```
kind : deployment로 아래 yaml을 배포한다.
```
![캡처16](https://user-images.githubusercontent.com/50174803/127979745-03fa5558-9861-40f2-af20-0fce2f750a8e.jpg)

```
deployment를 service로 노출시킨다. 
```
![캡처15](https://user-images.githubusercontent.com/50174803/127979892-2f5881be-2fc2-4fdc-9000-1c3e0060f739.PNG)

```
describe 로 service의 엔드포인트를 확인해보자. 
192.168.9.162로 nginx-deployment 의 IP와 동일하다 .

deployment로 배포된 파드를 삭제하면 replica가 1이기 때문에 롤링 업데이트 방식으로

기존의 파드를 삭제하고 새로운 파드가 생성된다.
다시 service의 엔드포인트를 확인해보자.
192.168.2.221 로 바뀌었고 이 IP는 새로 생성된 nginx-deployment 의 IP와 동일하다.
```
![캡처17](https://user-images.githubusercontent.com/50174803/127980295-12c8896c-d6ec-4eec-8c53-4efba9308b4f.PNG)

## 두 가지 테스트를 통해 얻을 수 있는 결론
```
Kind: Pod를 통해 서비스 객체를 만들면, 해당 파드가 사라지면 서비스의 엔드포인트가 정의되지 않아
외부에서 액세스할 수 없다.

그러나, Kind: Deployment 를 통해 서비스 객체를 만들면, 해당 파드가 사라지더라도 정의된 레플리카의 수 만큼
파드가 생성되고 새로운 파드의 IP로 서비스의 앤드포인트가 변화하여 중단없이 외부에서 액세스 가능하다.
```
