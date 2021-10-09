# Chapter 11 - 쿠버네티스 내부

## 컨트롤러의 상호 협력 방식
```
쿠버네티스의 동작 원리에 대한 이해를 위해 포드의 리소스가 생성될 때
무슨 일이 발생하는 지 알아본다.
```

## 관련된 컴포넌트의 이해
```
전체 프로세스를 시작하기 전에도 컨트롤러, 스케줄러, Kubelet 은 각 리소스 유형의
변경 사항을 API 서버에서 감시하고 있다. 
그림에 etcd가 없는 이유는 API 서버 뒤에 숨겨져 있기 때문이다.
그래서 API 서버를 객체가 저장되는 공간으로 생각할 수 있다.
```
![image](https://user-images.githubusercontent.com/50174803/136661740-1db8286b-204c-4a2f-8f9b-5f6bb218934a.png)

## 이벤트 체인
```
배포 매니패스트가 포함하는 YAML 파일을 준비하고 이를 kubectl 을 통해서 쿠버네티스에 
제출한다고 상상해보자.

kubectl 은 쿠버네티스 API 서버로 HTTP POST 요청으로 매니패스트를 보낸다.
API 서버는 배포 명세를 확인하고 etcd에 저장한다.
그리고 kubectl 에 응답한다.
```
![image](https://user-images.githubusercontent.com/50174803/136661876-01f63ca6-3ab2-4c98-a919-4f93f110f82b.png)

## 포드의 컨테이너를 실행하는 Kubelet 
```
특정 노드에 스케줄되는 포드와 함께 노드의 Kubelet은 일을 시작한다.
API 서버에서 포드의 변화를 감시하는 Kubelet은 노드로 스케줄되는 새로운 포드를 살펴본다.
그래서 포드의 정의를 조사하고 포드의 컨테이너를 시작하기 위해 도커나 컨테이너 런타임 등에게 알려준다.
그런 후에 컨테이너 런타임은 컨테이너를 실행한다.
```

## 실행 중인 포드의 이해
```
kubectl run nginx --image=nginx 명령어를 통해 단일 컨테이너 포드를 실행한다.
그 이후 docker ps 를 통해 모든 컨테이너를 나열할 수 있다.
nginx 컨테이너 뿐 아니라 추가 컨테이너를 볼 수 있다. 
일시 정지된 컨테이너는 포드의 모든 컨테이너를 함께 담고 있는 컨테이너다.

일시 정지된 컨테이너는 인프라스트럭처 컨테이너로써 모든 네임스페이스를 보유하는 것이
고유한 목적이다. 포드의 모든 사용자 정의 컨테이너는 포드 인프라스트럭처 컨테이너의
네임스페이스를 사용한다.
```
![image](https://user-images.githubusercontent.com/50174803/136662534-d5e1a6a6-03ca-4c66-90a8-71233a381ceb.png)
![image](https://user-images.githubusercontent.com/50174803/136662661-c90770a4-7016-430b-9d79-81f4e60d00d2.png)

## 인터-포드 네트워킹 495 