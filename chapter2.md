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
