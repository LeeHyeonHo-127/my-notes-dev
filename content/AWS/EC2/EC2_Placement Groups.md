---
tags:
  - aws
title: Placement Groups
date: 2024-03-28
---
# EC2_Placement Groups 이란?

EC2를 사용하는 중 EC2 Instance의 배치 전략이 필요할 수 있습니다. 
예를 들어 가용성이 중요한 EC2라면 EC2 끼리 떨어뜨리거나. 속도가 중요하다면 최대한 가깝게 배치하는 등으로 말입니다.

AWS에서는 Placement Groups로 EC2 배치 전략을 수립할 수 있습니다.

배치전략은 3가지 입니다.

- Cluster 
- Partition
- Spread

# Cluster

![[Pasted image 20240328192824.png]]
같은 AZ 같은 Rack에 EC2를 배치합니다.

장점: 
	네트워크가 매우 빠릅니다(Enhance Network를 가능하게 한 EC2 간 10Gbps의 bandwidth)
단점: 
	Rack이 손상된다면 모든 Instance가 손상됩니다.
Usecase:
- 매우 빠른 네트워크가 필요한 Application
- 빠른 완료를 위한 Big Data 작업

# Spread

![[Pasted image 20240328193305.png]]
장점:
	서로 다른 Hardware기기에 EC2를 배치합니다. 또한 across AZ하게 배치합니다. 
	Risk를 낮출 수 있습니다.
단점:
	하지만, AZ당 7개의 Instance만을 배치할 수 있습니다.

Usecase: 
	고가용성이 필요한 Application
	서로 장애로 부터 격리되어야 하는 중요한 Application

# Partition Group

![[Pasted image 20240328193601.png]]

AZ당 7개의 Partiion을 나누고 Partition 내부에 EC2를 배치합니다. 이는 across AZ하게 배치할 수 있습니다.
Partition 끼리는 Rack을 공유하지 않습니다. Partition 내부에 장애가 발생해도 Partition외부로 전파되지 않습니다.

EC2 Instance는 metadata로 Partion 정보를 볼 수 있습니다

Usecase:
	HDFS, HBase, Cassandra, Kafka