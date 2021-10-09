# Chapter 11 - 쿠버네티스 내부


## 아키텍처 이해
```
쿠버네티스 클러스터는 아래와 같이 두 부분으로 나뉜다.
- 쿠버네티스 컨트롤 플레인
- 워커노드

- 컨트롤 플레인 컴포넌트
역할 : 전체 클러스터 기능을 제어하고 만드는 역할을 한다.
* ETCD 분산 스토리지
* API 서버
* 스케줄러
* 컨트롤 매니저

- 워커노드 컴포넌트
역할 : 컨테이너를 실행하는 작업은 워커노드에서 진행
* Kubelet
* 쿠버네티스 서비스 프록시 (kube-proxy)
* 컨테이너 런타임

```
![image](https://user-images.githubusercontent.com/50174803/135829795-3c3ee058-f6db-4902-a0a0-a8f1f5a2a8a8.png)

## 컴포넌트와 커뮤니케이션하는 방법
```
쿠버네티스 시스템 컴포넌트는 오직 API 서버와 통신한다.
서로 직접 통신하지 않는다.

API서버는 etcd와 통신할 수 있는 유일한 컴포넌트다.

API 서버와 다른 컴포넌트 간의 연결은 거의 항상 컴포넌트에 의해 먼저 시작된다.
그러나 로그를 가져오기 위해, 실행 중인 컨테이너에 연결하기 위해, port-forward 할 때는
API서버가 Kubelet에 먼저 연결을 시도한다.
```

## 컴포넌트의 실행 방법
```
Kubelet은 정규 시스템 컴포넌트로서 항상 실행할 수 있는 유일한 컴포넌트다.
또한 Kubelet은 포드로서 모든 컴포넌트를 실행한다.
포드로서 컨트롤 플레인 컴포넌트를 실행하기 위해 Kubelet 또한 마스터에서 배포된다.
```
![image](https://user-images.githubusercontent.com/50174803/135830678-b1182b0e-8fd8-47e1-9930-ae8488b772ec.png)

## 쿠버네티스가 etcd를 사용하는 방법
```
모든 오프젝트(포드,서비스,시크릿 등)는 API 서버가 다시 시작하거나 실패하더라도 
생존하기 위해 어딘가에 영구적으로 저장해야 한다.
이를 위해 쿠버네티스는 빠르고, 분산되며, 일관된 키 값 스토리지인 etcd를 사용한다.

분산돼 있으므로 둘 이상의 인스턴스를 실행해 고가용성과 우수한 성능을 제공한다.

etcd와 직접 통신하는 유일한 컴포넌트는 쿠버네티스 API 서버이다.
다른 모든 컴포넌트는 API서버를 통해 간접적으로 etcd에 데이터를 읽고 쓴다.

여기에 몇 가지 장점이 있는데, 강력한 낙관적 잠금 시스템과 유효성 검사가 있다.

etcd는 쿠버네티스가 클러스터 상태 및 메타 데이터를 저장하는 유일한 장소이다.
```

## etcd에 리소스를 저장하는 방법 

![image](https://user-images.githubusercontent.com/50174803/135831704-1109ac21-23e8-4774-98b4-9c890326b072.png)
![image](https://user-images.githubusercontent.com/50174803/135831731-c79a0c2f-c75e-4f07-a6f3-c62cd9d2d842.png)
![image](https://user-images.githubusercontent.com/50174803/135831746-21ab8b75-e49a-4d2a-9065-20e3e7abcae3.png)

## etcd가 클러스터가 될 때 일관성 보장

```
고가용성을 보장하기 위해 일반적으로 etcd인스턴스를 하나 이상 실행한다. 
여러 개의 etcd 인스턴스가 일관성을 유지할 필요가 있다. 
```
![image](https://user-images.githubusercontent.com/50174803/135832187-8782df3d-bff4-455e-b844-e2324d0cd46c.png)

## etcd 인스턴스의 개수가 홀수여야 하는 이유

```
etcd인스턴스는 일반적으로 홀수로 배포된다. 인스턴스가 두 개 있다면
두 인스턴스가 모두 있어야 과반수가 된다. 두 개 중 하나에 장애가 발생한 경우
다수가 없으므로 etcd 클러스터를 새 상태로 전활할 수 없다.
```

## API 서버가 하는 일 
 
```
쿠버네티스 API 서버는 모든 컴포넌트와 kubectl 같은 클라이언트가 사용하는 중앙 컴포넌트다.
클러스터의 상태를 쿼리 및 수정할 수 있는 CRUD 인터페이스를 제공한다. 그 상태를 etcd 안에 저장한다.
```

![image](https://user-images.githubusercontent.com/50174803/135833245-f543b7b1-f02f-4fff-a9c9-4b27fcfc40ba.png)
![image](https://user-images.githubusercontent.com/50174803/135833471-b44f71e2-11af-42fd-bb41-05f392bf80fd.png)

## 스케줄러의 이해

```
API 서버의 감시 매커니즘을 통해 새롭게 생성된 포드를 기다리고 노드에 아직 할당되지 않은
새로운 포드를 노드에 할당하는 일을 한다.

스케줄러는 선택된 노드에 포드를 실행하도록 지시하지 않는다.
스케줄러가 하는 것은 API 서버를  통해 포드의 정의를 업데이트하는 일이다.

그리고 나서 API서버는 포드에 스케줄됐다고 kubelet에게 통지한다.
대상 노드의 Kubelet이 포드가 해당 노드에 스케줄됐음을 확인하자마자 
포드의 컨테이너를 생성하고 실행한다. 
```

![image](https://user-images.githubusercontent.com/50174803/135834167-c82b6219-a275-4631-951b-2ede517b5f95.png)

## 컨트롤러 매니저에서 실행되는 컨트롤러

```
API서버는 etcd에 리소스를 저장하고 변경 사항을 클라이언트에 통지하는 것 외에 아무것도 하지 않는다.

따라서 API 서버를 통해 배포된 리소스에 지정된 대로 시스템의 실제 상태가 원하는 상태로 수렴되는 지
확인하려면 다른 활성화된 컴포넌트가 필요하다.

이것은 컨트롤러 매니저에서 실행되는 컨트롤러에 의해 수행된다.

* 레플리카셋, 데몬셋 잡 컨트롤러
* 디플로이먼트, 스테이트풀셋,노드,서비스 컨트롤러
* 앤드포인트, 네임스페이스, 영구볼륨 컨트롤러 등

리소스는 클러스터에서 무엇을 실행해야하는 지 설명하는 반면, 
컨트롤러는 배치된 리소스의 결과로 실제 작업을 수행하는 활성화된 쿠버네티스 컴포넌트다.
```

## 컨트롤러가 하는 일

```
컨트롤러는 리소스(디플로이먼트,서비스 등) 변경을 위해 API 서버 감시를 수행하고 새로운 객체를 생성하거나 
기존의 객체의 삭제 또는 업데이트 등의 작업을 수행한다.
```

## 컨트롤러 정리

```
모든 컨트롤러는 API서버를 통해 API 오브젝트를 가지고 동작한다. 컨트롤러는 Kubelet과 직접 통신하거나
어떤 종류의 명령도 내리지 않는다. 
```

## Kubelet이 하는 일

```
마스터 노드에서 실행되는 모든 컨트롤러와 대조적으로 Kubelet과 서비스 프록시는
워커노드에서 실행된다. 

Kubelet은 워커노드에서 실행되는 모든 것에 책임을 가지는 컴포넌트다.

Kubelet의 초기 작업은 API 서버에서 노드 리소스를 생성해 실행하고 있는 노드를 등록하는 것이다.
그러고 나서 노드에 스케줄됐던 포드에 대해 API 서버를 모니터해야 한다. 
그리고는 포드의 컨테이너를 시작한다.

설정된 컨테이너 런타임에 특정 컨테이너 이미지에서 컨테이너를 실행하도록 
지시함으로써 이런 작업을 수행한다. 
그런 후 Kubelet은 지속적으로 실행 중인 컨테이너를 모니터링하고 상태와 이벤트, 리소스의 소모를
API 서버에게 보고한다.

Kubelet도 컨테이너 라이브니스 프로브를 실행하는 컴포넌트이며 프로브가 오류가 발생했을 때 컨테이너를
다시 시작한다. 
```
![image](https://user-images.githubusercontent.com/50174803/135836759-a3deca4c-a521-4f8b-bbe9-38e906030547.png)

## 쿠버네티스 서비스 프록시의 역할 

```
Kubelet 외에도 모든 워커노드는 kube-proxy를 실행하는 데, 쿠버네티스 API를 통해
정의한 서비스에 클라이언트가 연결할 수 있도록 하는 것이 목적이다.
kube-proxy는 서비스 IP 및 포트 연결을 보장한다. 

서비스가 둘 이상의 포드로 백업되면 프록시는 해당 포드에서 로드밸런싱을 수행한다.

[ 프록시로 불리는 이유 ]
초기에는 실제 서버 프로세스가 연결을 수락하고 이를 포드로 전달했다.
서비스 IP로 향하는 연결을 가로채기 위하여 프록시는 iptables 규칙을 다음과 같이 구성한다.

현재는 훨씬 더 나은 성능 구현을 위해 iptables 규칙을 사용해 패킷을 실제 프록시 서버를
통과시키지 않고 무작위로 선택한 백엔드 포드로 리다이렉션한다.
이 모드를 iptables 프록시 모드라 부른다.

* iptables 란?
리눅스 커널의 패킷 필터링 기능을 관리하기 위한 도구입니다.
```
![image](https://user-images.githubusercontent.com/50174803/136660798-8730dbca-6faa-49dc-9f28-e53d970e4a56.png)
![image](https://user-images.githubusercontent.com/50174803/136660929-c6a20073-3580-4f70-8272-fbaa43d85dc8.png)

## 두 가지 프록시 모드의 차이
```
Userspace 프록시 모드 : 프록시 서버를 통해 리다이렉트
Iptables  프록시 모드 : 포드로 직접 리다이렉트 (사이에 어떤 프록시 서버도 없음) 

두 가지 모드 간의 주요 차이는 패킷을 쿠버 프록시를 거쳐 유저스페이스에서
처리하느냐 아니면 오직 커널에서만 처리하느냐이다. 이것은 성능상의 주요 쟁점이다.

유저스페이스 프록시 모드는 단순한 라운드 로빈 방식을 사용해 연결이 포드에 걸쳐
균형을 맞춘다.

iptables 프록시 모드는 포드를 랜덤하게 선택한다.
```

## 쿠버네티스 애드온 DNS 서버의 동작 원리
```
클러스터의 모든 포드는 클러스터 내부 DNS 서버를 사용하도록 구성하는 것이 기본이다.
DNS 서버 포드는 kube-dns 서비스를 통해 익스포트하고 다른 포드들처럼 포드가 클러스터상에서 
이동할 수 있게 한다. 
서비스의 IP 주소는 클러스터에 배포되는 매 컨테이너 내부의 /etc/resolv.conf 파일의 nameserver에 지정된다.

kube-dns 포드는 API 서버의 서비스와 앤드포인트로의 변화를 관찰하기 위해 감시 매커니즘을 사용하고 변화할 때마다
DNS 레코드를 갱신한다.
클라이언트가 항상 최신의 DNS 정보를 얻을 수 있게 한다.

```