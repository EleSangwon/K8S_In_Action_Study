# Chapter8 : 애플리케이션에서 포드 메타 데이터와 그 외의 리소스에 접근하기 P343~376

```
애플리케이션에는 자신과 클러스터의 다른 구성 요소에 대한 세부 정보를 포함해 실행 중인 환경에 대한
정보가 필요한 경우가 있다.

특정 포드 및 컨테이너 메타 데이터를 컨테이너에 전달하는 방법과 컨테이너 내부에서 실행되는 애플리케이션이
쿠버네티스 API 서버와 통신하여 클러스터에 배포된 리소스 정보를 얻는 방법과 더 나아가 리소스를 생성하거나
수정하는 방법을 알아보도록 한다.
```

## 쿠버네티스 API 서버와 통신하기
```
애플리케이션이 다른 리소스에 대한 데이터를 필요로 하거나 가능한 한 최신 정보에 접근이 필요한 경우
API서버와 직접 통신해야 한다.
```
![캡처1](https://user-images.githubusercontent.com/50174803/130325900-260f4787-8798-418a-b23d-2a3132d759ea.jpg)

## 쿠버네티스 REST API 탐색
```
kubectl cluster-info 를 실행해 API 서버에 URL을 얻을 수 있다.

서버는 HTTPS를 사용하고 인증이 필요하기 떄문에 직접 통신하기가 쉽지 않다.
curl 을 사용해 엑세스하고, curl 의 --insecure 옵션을 사용해 서버 인증서 검사를 건너뛸 수는 있지만
멀리까지 전달되지는 못한다.

다행히 인증을 직접 처리하는 대신 kubectl proxy 명령을 실행해 프록시를 통해 서버와 통신할 수 있다.

< kubectl proxy를 통한 API 서버 접근 >
kubectl proxy 명령은 로컬 시스템에서 HTTP 연결을 허용하고 인증을 처리하는 동안 API 서버로 프록시하는 프록시 서버를
실행하므로 요청할 때마다 인증 토큰을 전달할 필요가 없다.

kubectl proxy 명령을 통해 , localhost:8001 을 얻고 curl 명령어를 통해 요청을 프록시로 보냈고
API 서버에 요청을 보낸 다음 프록시 서버가 무엇가를 반환했다.

```
![캡처2](https://user-images.githubusercontent.com/50174803/130326053-286397de-7fe9-48b3-9916-8c40db20c17d.jpg)
![캡처3](https://user-images.githubusercontent.com/50174803/130326054-c9366650-f2f3-47c4-92b6-a1b65004966e.jpg)
![캡처4](https://user-images.githubusercontent.com/50174803/130326138-5b465c6d-c073-4d0c-a86f-c78956fb73a0.jpg)

## 배치 API 그룹의 REST 엔드포인트 나열
![캡처5](https://user-images.githubusercontent.com/50174803/130326175-c415bcdb-7072-4d1f-bb4c-d313ecfe9aad.jpg)

## 포드 내에서 API 서버와 통신
```
kubectl proxy 를 사용해 로컬 시스템에서 API 서버와 통신하는 방법을 배웠다.
이제 kubectl 을 가지고 있지 않은 보통의 포드 내에서 API 서버와 통신하는 방법을 보자.
포드 내부에서 API 서버와 통신하려면 다음 세 가지를 처리해야 한다.

- API 서버의 위치를 찾아야 한다.
- API 서버를 가장한 것이 아닌 API서버에게 이야기해야 한다.
- 서버와 인증한다.

< API 서버와의 통신을 시도해서 포드 실행하기 >
먼저 API와 통신할 수 있는 포드가 필요하다. 아무것도 하지 않는 포드를 실행한 다음 kubectl exec로 컨테이너에서
쉘을 실행한다. 그 뒤 curl 을 사용해 쉘 내에서 API 서버에 액세스를 시도한다. 
그래서 curl 바이너리를 포함하는 컨테이너 이미지를 사용해야 한다.

kubectl exec -it curl bash 명령어를 통해 컨테이너에서 쉘을 실행시키면
API 서버와 통신을 준비를 마쳤다.

API 서버의 주소를 찾아야 하기 때문에 , kubectl get svc 를 통해 쿠버네티스 API 서버의 IP와 포트를 찾는다.

```

![캡처6](https://user-images.githubusercontent.com/50174803/130326359-f979f8bd-3181-4fbf-9432-f36d2f5d8b57.jpg)
![캡처7](https://user-images.githubusercontent.com/50174803/130326363-b31d4bed-c17a-41d5-bf16-30cbe90c44ce.jpg)
![캡처8](https://user-images.githubusercontent.com/50174803/130326366-cebb9a90-81f6-4b3c-ad04-f9371cf016e6.jpg)

## 포드가 쿠버네티스와 통신하는 방법 정리
```
- API 서버의 인증서가 ca.crt 파일에 있는 certificate 기관에 의해 서명됐는지 여부를 확인

- 애플리케이션은 token 파일에서 무기명 토큰과 함께 Authorization 헤더를 보내 자신을 인증

- namespace 파일은 포드의 네임스페이스 안에 있는 API 객체에 대해 CRUD 작업을 수행할 떄 네임스페이스를
API 서버로 전달하는 데 사용해야 한다.
```

![캡처9](https://user-images.githubusercontent.com/50174803/130326619-23ad5899-e433-48f5-93c3-0d084bc937e4.jpg)

## 클라이언트 라이브러리를 사용해 API 서버와 통신
```
애플리케이션에서 API 서버에 몇 가지 간단한 작업만 수행하는 경우 일반 HTTP 클라이언트 라이브러리를 사용해
간단한 HTTP 요청을 수행할 수 있다. 

간단한 API 요청을 수행하려면 쿠버네티스 API 라이브러리를 사용하는 것이 좋다.
```

## 요약
```
포드 내부에서 실행 중인 애플리케이션이 자체, 다른 포드 및 클러스터에 배포된 다른 구성 요소에 대한
데이터를 얻을 수 있는 방법에 대해 살펴봤다.

- kubectl proxy 를 통해 쿠버네티스 REST API 를 검색하는 방법

- 쿠버네티스에 정의된 다른 서비스와 마찬가지로 환경 변수 또는 DNS를 통해 포드가 API 서버의
위치를 찾는 방법

- 포드에서 실행 중인 애플리케이션이 API 서버와 통신하고 있으며 자신을 인증하는 방법을 확인하는 방법

- 클라이언트 라이브러리를 통해 쿠버네티스와 상호작용할 수 있는 방법
```











