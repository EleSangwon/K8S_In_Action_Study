# 서비스: 클라이언트가 포드를 검색하고 통신을 가능하게함 P200~252

```
특정 포드는 외부 자극과는 별도로 작업을 수행할 수 있지만 대다수의 애플리케이션은
외부 요청에 응답하기 위한 것이다.

예를 들어, 마이크로서비스의 경우 포드는 대게 클러스터 내의 다른 포드의 요청이나
클러스터 외부의 클라이언트로부터 오는 HTTP 요청에 응답한다.
```

## 서비스 소개
```
쿠버네티스 서비스는 동일한 서비스를 제공하는 포드 그룹에 단일 진입 점을 만들기 위해
생성하는 리소스다.

각 서비스에는 서비스가 존재하는 동안 절대로 변경되지 않는 IP주소와 포트가 있다.

클라이언트는 해당 IP 및 포트에 연결할 수 있고,이 연결은 해당 서비스를
지원하는 포드 중 하나로 라우팅된다.

이렇게하면 서비스 클라이언트는 서비스를 제공하는 포드의 개별 위치와 무관해지므로
포드는 언제든지 클러스터 주변에서 이동할 수 있다
```

## 예제를 통한 서비스 설명
```
프론트엔드 웹 서버와 백엔드 데이터베이스 서버가 있다고 하자.

프론트 엔드로 동작하는 여러 개의 포드가 있을 수 있지만 하나의 백엔드 DB 포드만이
있을 수 있다. 시스템 기능을 만들기 위해 두 가지 문제를 해결해야 한다

- 외부 클라이언트는 단일 웹 서버든 수백 개의 서버든 관계없이 프론트엔드 포드로 
연결할 수 있어야 한다

- 프론트엔드 포드는 백엔드 DB에 연결할 수 있어야 한다.
DB는 포드 안에서 실행되기 때문에 포드는 시간에 지남에 따라 IP가 변경되면서 클러스터 상에서
이동할 수 있다. 백엔드 DB가 옮겨질 때마다 프론트엔드 포드를 재설정하지 않길 바란다.

프론트 포드를 위한 서비스를 생성하고 클러스터 외부로부터 액세스가 가능하게 설정해
클라이언트가 포드로 접속을 가능케 하는 고정 IP주소를 노출할 수 있다.

마찬가지로 백엔드 포드의 서비스를 작성하여 백엔드 포드에 안정적인 주소를 생성한다.
포드의 IP주소가 변경 되더라도 서비스의 IP주소는 변경되지 않는다.
```
![캡처1](https://user-images.githubusercontent.com/50174803/128696706-18f6bc90-a9e4-4994-b615-cb5ab92d7ca0.jpg)


## 서비스 생성
```
어떤 포드가 서비스의 일부인지 어떻게 식별할 수 있을까?

같은 라벨 셀렉터 세트를 사용한다.
```
![캡처2](https://user-images.githubusercontent.com/50174803/128721390-6741c169-0e7a-4169-9c3f-28477a78b7a0.jpg)

## 클러스터 안에서 서비스 테스트
```
몇 가지 방법을 통해 클러스터 내부의 서비스에게 요청을 전송할 수 있다.

- 서비스의 클러스터 IP주소로 요청을 보내고 응답을 로그로 남기는 포드를 생성하는 것
어떤 서비스가 응답했는지 포드의 로그를 통해서 확인할 수 있다.

- 쿠버네티스의 노드 중 하나로 SSH 접속을 수행하고 curl 명령을 사용할 수 있다.

- kubectl exec 명령어를 통해 이미 존재하는 포드들 중 하나에서 curl 명령을 수행

kubectl exec 명령을 통해 특정한 컨테이너의 포드 안에서 원격으로 임의의 명령어를 실행할 수 있다.

kubectl exec <특정포드이름> -- curl -s <서비스의IP주소>
```

![캡처3](https://user-images.githubusercontent.com/50174803/128722150-87ef8b3e-f89f-4d64-914b-fb09dfbb5088.jpg)

## 동일한 서비스에서 여러 개의 포트 노출

![캡처4](https://user-images.githubusercontent.com/50174803/128722522-8421e7be-0bd7-456e-b997-dcd08729b0a2.jpg)

## 클러스터 외부 서비스에 연결
```
서비스를 외부로 노출이 필요한 경우가 있다. 클러스터 내의 포드로 연결을 리다이렉트하는 서비스 대신에
외부의 IP와 포트로 리다이렉트해야 하는 경우가 있다.

이 경우 서비스 로드 밸런싱이나 서비스 디스커버리 수행 시 이점이 있다.
클러스터 내에 실행 중인 클라이언트 포드는 내부 서비스에 연결하는 것처럼 외부 서비스에 연결
할 수 있다.

서비스 엔드포인트 리소스는 서비스에 의해 노출되는 IP주소와 포트의 목록이다.

서비스의 엔드포인트를 서비스와 분리하면 엔드포인트를 수동으로 구성하고 업데이트할 수 있다.
포드 셀렉터 없이 서비스를 만들었다면 쿠버네티스는 엔드포인트 리소스조차 만들지 못한다.

수동으로 관리되는 엔드포인트를 사용해 서비스를 만들려면 서비스 및 엔드포인트
리소스를 모두 만들어야 한다.
```
![캡처5](https://user-images.githubusercontent.com/50174803/128724112-ae28c581-e2f4-4f7d-911b-8b85e3a016f3.jpg)
![캡처6](https://user-images.githubusercontent.com/50174803/128724127-20c5b742-eeeb-40f1-8f1c-33f463582ba6.jpg)
![캡처7](https://user-images.githubusercontent.com/50174803/128724222-bf8c3906-5c73-4434-b426-6292b4073347.jpg)

## 외부 서비스에서 외부 클라이언트로
```
서비스가 외부에서 액세스 가능하게 하는 방법

- NodePort 서비스 타입으로 설정하기

NodePort 서비스의 각 클러스터 노드는 노드 자체의 이름을 통해 포트를 열고 포트에서 발생한
트래픽을 서비스로 리다이렉트한다.

- LoadBalancer 서비스 타입으로 설정하기

쿠버네티스가 실행 중인 클라우드 인프라에 프로비저닝된 지정된 로드밸런서를 통해
서비스 액세스 가능하게 된다.
로드밸런서는 발생한 트래픽을 모든 노드에서 노드 포트로 리다이렉트한다. 
클라이언트는 로드 밸런서의 IP를 통해 서비스에 접속한다.

-하나의 IP 주소를 통해 여러 서비스를 제공하는 인그레스 리소스 생성하기

HTTP 레벨(네트워크 7계층) 수준에서 동작하기 때문에 4계층 서비스보다 
좀 더 기능을 제공한다.
```
![캡처8](https://user-images.githubusercontent.com/50174803/128724967-f56cf1c5-4b63-41da-a2a0-fccaeb923463.jpg)

## NodePort 서비스 사용
![캡처9](https://user-images.githubusercontent.com/50174803/128725242-f98b975b-20e8-46a3-b7f9-cfbd19d02118.jpg)

## LoadBalancer 를 이용한 서비스 노출
![캡처10](https://user-images.githubusercontent.com/50174803/128725474-a4e23ff7-c246-4418-bb08-2a1c18e3b884.jpg)

## 인그레스 리소스를 이용해 외부로 서비스 노출
```
- 인그레스가 필요한 이유

각 로드밸런서 서비스는 그 자신만의 외부 IP를 갖는 자체 로드 밸런서를 필요로 하는 반면,
인그레스는 수십 개의 서비스에 대한 엑세스를 제공할 떄에도 하나의 IP만 필요로 하기 떄문이다.

다수의 서비스는 단일 인그레스로 노출할 수 있다.

로드밸런서 타입으로 노출시키면 다수의 서비스는 고유의 외부 IP를 갖는 다수의 로드밸런서가 생성됨

```
![캡처11](https://user-images.githubusercontent.com/50174803/128725944-89ec2add-75e1-4867-918a-27e8b4c0b6a2.jpg)
![캡처12](https://user-images.githubusercontent.com/50174803/128726245-3afa2a17-b236-4c1d-95e1-22f5727b06ef.jpg)

## 포드가 연결을 수락할 준비가 됐을 떄 신호 보내기
```
적절한 라벨의 새로운 포드가 생성되자 마자 서비스의 일부가 되고 요청이 포드로 리다이렉트
되기 시작한다.

그러나 포드가 요청을 처리할 준비가 되어 있자 않다면 어떻게 될까?

포드는 구성 또는 데이터를 로드하는 데 시간이 필요하거나, 첫 번쨰 사용자 요청이 
너무 오래 걸리고 사용자 환경에 영향을 미치지 않도록 예열 절차를 수행해야할 수 있다.
그런 경우 수신한 요청을 포드가 바로 시작하길 원치 않을 수 있다.

프로세스가 완전히 준비될 때까지 시작 중에 요청을 포드로 전달하지 않는 것은
당연한 일이다.

그래서 쿠버네티스는 포드를 위한 레디네스 프로브(readiness probe)를 제공한다.
레디네스 프로브는 주기적으로 호출되고 특정 포드가 클라이언트 요청의 수락 여부를 결정한다.

컨테이너의 레디니스 프로브가 성공을 반환한다면 컨테이너가 요청을 받아들일 준비가 됐다는 신호다.
```

## 레디네스 프로브의 동작이해
```
컨테이너가 시작되면 쿠버네티스는 첫 번째 준비 확인을 수행하기 전에 구성 가능한 시간이
경과할 때까지 대기하도록 구성할 수 있다.
그리고 주기적으로 프로브를 호출하고 레디네스 프로브의 결과를 기반으로 수행된다.

라이브니스 프로브와 달리 컨테이너가 준비 확인에 실패한다면 종료되거나 다시 시작하지 않는다.

라이브니스 프로브는 상태가 좋지 않은 컨테이너를 종료하고 새로운 것으로 교체함으로써 
포드의 상태를 좋게 유지한다.

반면에 레디네스 프로브는 오직 포드가 요청을 수신할 수 있는 환경이 됐을 떄 수신한다.
```

## 레디네스 프로브가 중요한 이유
```
애플리케이션 서버를 실행하는 포드그룹이 백엔드 데이터베이스 포드에서 제공하는 서비스에
의존한다고 가정해보자.

프론트엔드 포드 중 하나에 문제가 발생해 더 이상 데이터베이스에 연결할 수 없는 경우
포드가 요청을 처리할 준비가 되지 않았다는 신호를 레디네스 프로브가 쿠버네티스에
알려주는 것이 현명한 방법이다.

다른 포드 인스턴스에서 동일한 유형의 연결 문제가 발생하지 않으면 요청을 정상적으로 
처리할 수 있다.
레디네스 프로브는 클라이언트가 정상 상태인 포드하고만 통신하게 하고 
시스템에 문제가 있다는 것을 알아차리지 못하게 한다.
```



















