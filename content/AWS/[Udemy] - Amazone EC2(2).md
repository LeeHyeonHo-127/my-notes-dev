---
tags:
  - aws
relate: "[[Saving Plan VS Reserved Plan]]"
date: 2024-03-25
title: Amazone EC2(2)
---
![[Pasted image 20240325104749.png | 600]]

# EC2 Instances Purchasing Options

- On-Demand-Instances : 짧은 작업량, 예측 가능한 가격 책정, 초당 비용 지불
- Reserved(1 & 3 years):
	- 긴 작업량일 때(Long workloads) 
	- Convertible Reserved Instances : 예약폭은 표준 예약 인스턴스보다 적지만, EC2의 인스턴스 유형을 바꿀 수 있습니다. EC2 인스턴스 유형은 가치가 같거나 큰 것으로만 변경할 수 있습니다. [참고 자료](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ri-convertible-exchange.html)
- Saving Plans (1 & 3 years): 긴 워크로드일때 사용합니다. 사용량이 약속되어있습니다.
- Spot Instances: 짧은 워크로드일 때 사용합니다. 가격이 싸지만, 낮은 가용성을 보장합니다.
- Dedicated Hosts: 전체 물리 서버를 예약합니다. Instance의 배치를 조정할 수 있습니다.
- Dedicated Instances: 다른 어떤 사람들과 본인의 hardware를 공유하지 않습니다.
- Capacity Reservations: 특정 기간동안 특정 AZ의 용량을 예약합니다.

## EC2 On Demand
- Pay for What you use: 
	OS마다 가격 책정이 다릅니다.
	Linux or Windows: Billing Per second, after the first minute
	All other OS: Billing Per hour
- 다른 옵션에 비해 비싸지만, 선불 금액은 없습니다
- Application이 어떻게 동작할지 예측이 안될 때, 끊기지 않는 Application이 필요할 때, Shot-term workload일 때 추천합니다.

## EC2 Reserved Instance
- On-Demand 대비 최대 72%를 할인받을 수 있습니다.
- 구체적인 Attribute(Instance Type, Region, Tenancy(공유 여부), OS)의 Instance를 예약합니다.
- 1년 혹은 3년으로 예약할 수 있습니다. 3년 예약이 1년 예약보다 더 많은 할인이 됩니다.
- 선결제, 부분 선결제, 선결제 없음으로 세 가지의 payment option이 있습니다. 선결제가 가장 할인이 많이됩니다.
- Regional 혹은 AZ 단위로 Instance를 예약할 수 있습니다.
- DB처럼 꾸준하게 사용하는 Application에 추천합니다.
- Marketplace에서 Reserved Instance를 사거나 팔 수 있습니다.
- Reserved Instance의 Instance Type, Instance Family, OS, Scope와 Tenancy를 변경할 수 있습니다. 이 때 가치가 같거나 큰 유형으로만 변경할 수 있습니다.

## EC2 Saving Plans
- EC2 Saving Plan은 1 or 3 년동안 일정 사용량을 약정하여 저렴한 요금을 제공하는 요금입니다.
- 장기 사용에 따른 할인(Reserved Instance로 최대 72%)
- 특정 유형의 사용량을 약정하기 (10$/hour for 1 or 3 years)
- EC2 Saving Plan을 넘어가는 사용은 On-Demand 가격으로 청구됩니다.

AWS는 2가지의 Saving Plan을 제공합니다. 
- Compute Savings Plans : 최대 66프로 할인. 인스턴스, Region 변경 가능
- EC2 Instance Savings Plans : 최대 72프로 할인. 리전의 개별 인스턴스 패밀리에 대한 약정. 리전 내에서 인스턴스 패밀리를 수정할 수 있다.

## EC2 Spot Instances
- On-Demand 대비 90%의 할인을 받을 수 있습니다.
- 현재 Spot Price보다 본인의 max price가 낮다면, Instance를 'lose'할 수 있습니다. 즉 책정한 가격이 낮다면 Instance가 뺏길 수 있습니다.
- AWS에서 가장 싼 Instance입니다.

- 장애에 대해서 복원력이 있는 워크로드에 적합합니다
	- Batch 작업
	- 데이터 분석
	- Image 처리
	- 분산 workload
	- 시작과 끝이 유연한 workload

## EC2 Dedicated Hosts
- EC2 Instance를 위한 물리적인 서버 전체를 사용합니다.
- 기존 서버 기반 SW license를(VM Software licenses..etc)를 사용할 수 있습니다.
- 규정 준수 요구사항을 충족할 수 있습니다.
- 구매 옵션:
	- On-Demand : 동작하는 Dedicated Host에 대해 초당 지불
	- Reserved : 1 or 3년을 예약(전체 선결제)
- EC2 Dedicated Hosts는 가장 비싼 옵션입니다.

- 강력한 규제가 있는 기업이거나, 복잡한 라이센스가 있는 소프트웨어를 사용할 때 유용한 옵션입니다.

## EC2 Dedicated Instances
- EC2 Dedicated Hosts와 유사하나 다른 옵션입니다.
- 선택한 Hardware에서 Instance가 동작합니다.
- 같은 계정의 Instance와 Hardware를 공유합니다.
- Instance 배치를 조절할 수 없습니다. (중지/시작후 Hardware를 이동할 수는 있습니다.)
![[Pasted image 20240325184831.png|400]]

## EC2 Capacity Reservations
- On-Demand Instance의 용량을 특정 AZ 특정 기간동안 예약합니다.
- 필요할 때 언제든지 EC2 용량에 접근할 수 있습니다.
- 시간 약정 없음(언제든 create/cancel 가능), 할인 없음
- Regional Reserved Instance와 Saving Plan을 결합하여 할인을 받으세요
- 인스턴스 실행 여부에 상관없이 On-Demand의 요금이 청구됩니다.
- 특정 AZ에 있어야 하는 중단 없는 단기 워크로드에 적합합니다.

- EC2 Capacity Reservations을 통해 On-demand로 필요한 컴퓨팅 용량을 확보하지 못 할 경우를 예방할 수 있습니다.
### ![[Pasted image 20240325185903.png | 700]]

## 정리
리조트에 비유하여 정리해 보겠습니다.
(개인적으로 잘 와 닿았습니다.)

- On Demand: 원할 때 리조트 사용, 사용한 만큼 전체 가격 지불
- Reserved: 미리 계획하거나, 오래 머물경우 할인을 받을 수 있다
- Saving Plans: 일정 기간동안 시간당 비용을 지불하기로 약속. 모든 타입의 방 사용 가능
- Spot Instances: 최고 입찰자가 빈 방을 사용. 쫒겨날 수 있음
- Dedicated Hosts: 리조트의 전체 건물을 예약
- Capacity Reservations: 숙박하지 않더라도 방을 특정 기간동안 예약

### Price Comparison(m4.large - us-east-1)
![[Pasted image 20240325190831.png]]

# EC2 Spot Instance Requests

- On-Demand 대비 최고 90% 할인을 받을 수 있습니다.
- 현재 최고가 보다 더 높은 가격을 제시할 경우 instance를 할당받습니다.
	- 시간당 spot price는 구매 옵션과, 리소스에 따라 다릅니다.
	- 현재 spot price가 본인의 가격보다 높을경우 2분의 유예시간을 두고 stop or terminate를 지정할 수 있습니다.
- Spot Instance Requests 에는 두 가지 방법이 있습니다.
	- 일회성 요청 : 한 번 요청 후에는 스팟 요청을 할 수 없다
	- 지속적인 요청 : 지속적으로 요청할 수 있다.(Instance가 종료 되거나 Interrupt되어도)

# How to terminate Spot Instances?

![[Pasted image 20240325193938.png|600]]
위 그림은 Spot Instance 요청의 두 가지 방법에 대한 도식도입니다.(일회성/지속적인 요청)

일회성 요청인 경우 Instance 실행후 Interrupt될 경우 프로세스가 끝납니다. Spot Request 또한 Instance가 시작되는 순간 사라집니다.
지속적인 요청일 경우 Instance가 종료 되면 다시 시작합니다. 혹은 Interrupt 된다면 다시 Spot Instance를 요청하여 자동으로 Instance를 실행합니다.


![[Pasted image 20240325193952.png|400]]
지속적인 요청으로 Spot Request를 생성했다면, Open, Active, Disabled 상태일 때만 Spot Instance Request를 취소할 수 있습니다.
Spot request를 취소해도 Instance는 종료되지 않습니다. 때문에, Spot Request를 취소하고, Spot Instance를 종료해야 합니다.

# Spot Fleets(스팟 집합 전략)

Spot Fleets은 자동으로 Spot Instance를 최저의 가격에 요청합니다.

- Spot Fleets = Spot Instance의 집합 + (Optional) On-Demand Instances
- Spot Fleets 정한 가격 제약안에서 목표 용량을 충족하기 위해 최선을 다합니다.
	- Launch Pools을 정의 합니다(Instance Type, OS, AZ)
	- Fleets이 최적의 Pool을 선택합니다.
	- 목표 용량에 도달하거나 max cost에 도달할 경우 Instance 시작을 종료합니다.

- Spot Fleet에 Instance를 할당하는 전략을 세워야합니다.
	- LowestPrice: 가장 낮은 가격의 pool에서 시작합니다.(비용 최적화, shot workload)
	- diversified: 모든 Pool에 분산해서 Spot Instance를 가져옵니다
		- 가용성 및 하나의 풀에서 발생하는 가격 상승에 덜 민감해집니다
		- On-Demand 가격보다 비싼 풀에서는 시작하지 않습니다.
	 - CapacityOptimized: 원하는 인스턴스 수에 맞는 최적의 용량을 가진 풀을 갖게됩니다.
	 - priceCapacityOptimized(추천) : 가장 용량이 큰 Pool을 선택 후 가격이 낮은 pool들을 순차적으로 선택합니다. 대부분의 workload에 적합한 선택입니다.
