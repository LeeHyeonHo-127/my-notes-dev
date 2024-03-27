---
tags:
  - 네트워크
  - 기술부채
date: 2024-03-27
---

> [!NOTE]
> L2 -> L3 라우터 기능 활성화
> 192.168.200.0 / 192.168.210.0 통신 확인
> 192.168.100.0/24 정적 라우팅 이용해서 통신

![[Pasted image 20240327173019.png | 500]]
위 그림처럼 L2 스위치를 이용해 서로 다른 네트워크 대역의 통신을 확인하고,
정적 라우팅을 이용해 router에 연결된 기기와 통신해 볼 것입니다.

# 1. PC IP Setting

![[Pasted image 20240327174254.png|500]]
![[Pasted image 20240327174124.png|500]]
![[Pasted image 20240327174141.png|500]]


# 2. L3 Switch로 변환

### 물리적 포트에 IP주소 할당
L2 의 물리적인 포트에는 IP주소를 할당할 수 없습니다. 
아래 스크린샷을 보면 IP 주소 할당이 안 되는 것을 볼 수 있습니다.
![[Pasted image 20240327161812.png]]

### 물리적 포트에 L3역할 할당
L3 Switch의 물리적 포트는 L2, L3역할 줄 하나를 관리자가 선택하여 사용할 수 있습니다. 

switch에서 아래 명령어를 통해 L3 Switch로 변환 후 IP Routing 기능을 동작하게 합니다.
이제 물리적인 포트는 L2(SwitchPort)가 아닌 L3(Routed Port)역할을 합니다. 
```
enable
conf t
int range f0/1 - f0/3
no switchport // L3 Switch Mode로 전환하기
ip routing // 스위치가 IP 라우팅 기능을 한다
```

### 물리적 포트에 IP주소 할당하기

아래 코드로 각 포트(인터페이스)에 IP를 할당해줍니다.
IP는 연결되어있는 PC의 네트워크 ID와 같아야합니다.

저는 Gateway를 .1 로 설정하였으므로, 포트의 IP에도 .1로 설정해 주었습니다.
```
Switch(config)#int f0/3
Switch(config-if)#ip addr 192.168.210.1 255.255.255.0
```

```
Switch(config-if)#int f0/2
Switch(config-if)#ip addr 192.168.200.1 255.255.255.0
```

### PC1 - PC2 통신

이제 통신을 해봅시다. 
이론적으로는 안 될 이유가 없습니다.

L3 Switch가 Router의 역할을 해 줄 것입니다.


```
ping 192.168.210.10
```

다행히 잘 됩니다.
![[Pasted image 20240327174658.png]]

tracert 명령어로 경로를 확인해봅시다.
```
tracert 192.168.210.10
```
![[Pasted image 20240327174732.png]]

Gateway(이자 L3 Switch의 포트)를 거쳐서 PC2로 전달됨을 볼 수 있습니다.

# 3. 정적 라우팅을 이용한 통신

L3 Switch와 직접적으로 연결된 기기들의 통신을 해 보았습니다.

이제 정적 라우팅을 통해서 PC0과 통신해봅시다.

### Setting

우선 Switch와 Router의 Interface에 IP를 설정해줍니다.

Switch의 포트에 IP를 설정해줍니다
```
Switch(config)#int f0/1
Switch(config-if)#ip addr 10.10.10.2 255.255.255.0
```

마찬가지로 Router의 Interface에 IP를 설정해줍니다
```
enable
conf t
Router(config)#int f1/0
Router(config-if)#ip addr 10.10.10.1 255.255.255.0
Router(config-if)#no shutdown
```

Router에 연결된 기기와 연결된 Router Interface에도 설정을 해줍니다.
```
Router(config)#int f0/0
Router(config-if)#ip addr 192.168.100.1 255.255.255.0
Router(config-if)#no shutdown
```

### 정적 라우팅

Setting을 통해 IP 할당은 끝이 났습니다.
이제 정적 라우팅을 통해 라우터가 경로를 알 수 있게 해줍시다.

![[Pasted image 20240327173019.png | 400]]

전체 구성도를 다시 한 번 봅시다.
PC1, PC2와는 직접적인 연결이 없으므로, 해당하는 경로를 테이블에 넣어주어야 합니다.

ip route 명령어를 통해 각 PC의 네트워크 대역대로 가기위해서는 스위치(10.10.10.2)에 물어 보라고 전달합니다.
```
Router(config)#ip route 192.168.200.0 255.255.255.0 10.10.10.2
Router(config)#ip route 192.168.210.0 255.255.255.0 10.10.10.2
```


### 통신 해보기 (PC0 -> PC2)

자 이제 세팅은 다 끝났습니다.

1. L2 스위치를 L3로 바꾸어 주어 다른 네트워크 대역대 끼리 통신하게 해주었고,
2. 라우터와 스위치를 연결해주고, 라우팅 테이블에 값도 넣어 주었습니다.

이제 통신해 봅시다.

PC0에서 PC2로 통신을 시도합니다.

```
ping 192.168.210.10
```
![[Pasted image 20240327184902.png]]

..연결이 안 됩니다.


### Routing이 안돼요

tracert 명령어로 어디서 막혔는지 확인해 봅시다
```
tracert 192.168.210.10
```

![[Pasted image 20240327185038.png]]
Gateway에서 Switch로 넘어가지 않습니다.


우선 라우터의 인터페이스를  확인해봅니다.
문제 없는 것 같습니다. IP할당은 잘 되어있고, Status도 up입니다.
```
show ip interface breif
```
![[Pasted image 20240327185227.png]]

routing table도 확인해 봅니다.
문제 없는 것 같습니다. 각 PC의 네트워크 대역대를 위해서 Switch에게 물어보도록 되어있습니다.
```
show ip route
```

![[Pasted image 20240327185758.png]]


아래는 Ping을 통해 통신이 안되는 부분을 확인한 그림입니다.


![[Pasted image 20240327192327.png | 600]]
Routing이 제대로 안 되고 있습니다.

불현듯 깨달았습니다. 
Switch에서 Routing Table을 설정하지 않았던 것을.

Switch에서 Routing Table을 확인해봅니다.
설정이 안 되어 있습니다.

```
show ip route
```
![[Pasted image 20240327192752.png]]

Routing이 되려면 Router 두 기기 모두 테이블에 경로가 지정되어야 한다는 것을 잊었었습니다.


기쁜 마음으로 경로를 알려줍니다.

```
ip route 192.168.100.0 255.255.255.0 10.10.10.1
```

### Routing이 잘돼요

PC0 에서 PC1로 연결을 시도합니다

```
ping 192.168.200.10
```
![[Pasted image 20240327193140.png]]

잘 됩니다.
각 Router는 서로의 주소를 알아야 통신이 가능합니다.
각 Router Table에 서로의 경로가 있어야 합니다.
이제는 진짜 안 잊을 것 같습니다.
