---
tags:
  - 네트워크
  - 기술부채
date: 2024-03-28
title: Routing Protocol(RIP)
---
1. RIP란?
2. RIP의 장단점
3. RIP v1, v2
4. RIP 실습해보기
---
# 1. RIP란?

Routing Information Protocol의 약자로 디스턴스 벡터 알고리즘을 기반으로 한 동적 라우팅 알고리즘입니다.
인접한 라우터의 라우팅 테이블을 수집해 저장하고, 저장한 데이터 중 Hop Count가 가장 작은 경로로 라우팅합니다.

(Hop Count : Router를 하나 지날 때 마다 Hop Count가 1 증가합니다.)

### RIP의 특징
- RIP는 v1, v2가 있습니다.
- AD값은 120 입니다
	(AD란 일종의 Routing Protocol의 우선순위 입니다. AD의 값이 작을수록 우선순위가 높습니다.)
-  작은 규모의 네트워크나 대형 네트워크의 말단에 사용하기 좋습니다. (RIP에는 홉 수 제한16이 있어 큰 네트워크에는 경로 계산의 오류가 발생할 수 있다고 합니다.)
- Routing 정보 전송을 위해 UDP 포트 520번을 사용합니다
- Convergence(수렴) Time이 3초 입니다. Convergence Time이란 네트워크 변화를 인식하고 수정하는 시간입니다. 
	30초는 느린편입니다. 때문에 Routing Loop 같은 문제가 발생할 수 있습니다.

### Routing Loop란?

인접한 라우터 끼리만 정보를 교환하는 Distance Vector Algorithm의 문제입니다.

A - B - C 라는 라우터가 서로의 테이블 정보를 교환했었다고 가정합시다. 
A에 연결된 pc a 가 문제가 발생했습니다. 때문에 A는 a로 가지 못한다고 인식합니다. 

하지만 B는 a 로 가는 경로를 아직 테이블에 저장하고 있습니다. 지정된 시간(RIP의 경우 30초)이 되면 B는 A에게 a로 가능 경로를 가르쳐 주며 갈 수 있다고 알려줍니다.
A는 이를 확인하고 a로 갈 수 있다고 테이블에 저장합니다.

a는 다운되었지만 서로 갈 수 있다고 정보를 갖고 있으니 패킷이 빙글빙글 돕니다. 이를 Routing Loop라고 부릅니다.


# 2. RIP의 장/단점

### 장점
- 설정이 쉽습니다
- 표준 Routing Protocol이므로 모든 Router에서 사용이 가능합니다

### 단점
- Metric을 Hop-Count로 계산합니다. 때문에 속도에 상관없이 hop수가 적은 쪽으로 전달되어 비효율적일 수 있습니다.
- 최대 Hop-count가 15입니다.
- Routing 전송 방식이 비효율적입니다.
	Topology(네트워크 구성도) 변화에 상관없이 30초 마다 인접 Router에게 Routing Table 내용 전체를 전송합니다.

# 3. RIP version

RIP 에는 1, 2 두 가지 버전이 있습니다.

### RIP Version 1
- Classful(SubnetMask가 없습니다)
- 정보 전송 시 Broadcast주소를(255.255.255.255) 사용합니다. 때문에 RIP가 설정 안된 다른 장비에도 불필요한 부하를 줍니다.

### RIP  Version2
- Classess 
- 정보 전송 시 Multicast(224.0.0.9)를 사용합니다.
	(Multicast란 하나의 메시지를 여러 개의 목적지로 동시에 전송하는 방식입니다)
- 각 Router에서 네트워크 경로 정보에 대한 인증을 할 수 있습니다. 
- Auto Summary(Classful하게 라우팅 정보를 요약)
- Manual Summary(Classless하게 라우팅 정보를 요약)
	RIP 설정시 Summary 방법을 설정해 줘 원하는 정보 요약법을 정할 수 있습니다.
	Default : Auto Summary


# RIP 실습

![[Pasted image 20240328170933.png]]

![[Pasted image 20240328161310.png]]![[Pasted image 20240328161344.png]]



### Router 1

![[Pasted image 20240328161434.png]]
![[Pasted image 20240328161559.png]]


router rip
network 100.100.100.100
network 1.1.1.1
network 10.10.10.0

### Router2

int lo0
ip addr 200.200.200.200 255.255.255.0

```
Router(config)#router rip

Router(config-router)#network 200.200.200.0

Router(config-router)#network

% Incomplete command.

Router(config-router)#network 172.16.10.0

Router(config-router)#network 1.1.1.0

Router(config-router)#
```


```
Router#show ip route

  
1.0.0.0/24 is subnetted, 1 subnets
C 1.1.1.0 is directly connected, Serial2/0
R 10.0.0.0/8 [120/1] via 1.1.1.1, 00:00:12, Serial2/0 # AD=120, hop수 1
R 100.0.0.0/8 [120/1] via 1.1.1.1, 00:00:12, Serial2/0
172.16.0.0/24 is subnetted, 1 subnets
C 172.16.10.0 is directly connected, FastEthernet0/0
C 200.200.200.0/24 is directly connected, Loopback0
```

본인것만 넣으면 통신이 된다

![[Pasted image 20240328162326.png]]



```
show ip router // 사용중인 Router Protocol 보는 명령어
```


```
debug ip rip // 로그 확인 (테이블 주고받기 확인)
no debug ip rip // 끄기
```
![[Pasted image 20240328162655.png]]

255.255.255.255(Broad Cast)로 주고받느다

# RIP V2

![[Pasted image 20240328170928.png]]

Router2
```
router rip
version 2
no auto-summary //subneting 되게 설정
SecureRouter(config-router)#network 1.1.1.0

SecureRouter(config-router)#network 100.100.100.0

SecureRouter(config-router)#network 10.10.10.

^

% Invalid input detected at '^' marker.

SecureRouter(config-router)#network 10.10.10.0
```

Router3

```
router rip
no auto-summary
network 200.200.200.0
network 1.1.1.0
network 172.16.10.0
```

서브넷 마스크가 Classless하게. 즉. 내가 지정한대로 routing table이 지정되었다.
![[Pasted image 20240328172222.png]]


비슷하게 Router4도 setting

ping을 날려보면 잘 날아갑니다
PC2 -> PC0
![[Pasted image 20240328172621.png]]



```
show ip protocols 
```
둘 다 version2로 전송 수신하는 것
![[Pasted image 20240328172757.png]]


```
debug ip rip
```
![[Pasted image 20240328172905.png]]

224.0.0.9로 전송한다
Version 1은 255.255.255.0 으로 보냄