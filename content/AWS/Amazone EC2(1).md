---
tags:
  - aws
relate: "[[[Udemy] - Amazone EC2(2)]]"
date: 2024-03-25
title: Amazone EC2(1)
---

![[Pasted image 20240325104749.png | 600]]

# EC2란?

EC2란 Amazone Elastic Compute Cloud의 줄임말입니다. 안전하고 크기 조정이 가능한 컴퓨팅 파워를 클라우드에서 제공하는 웹 서비스입니다. AWS에서 제공하는 서비스 중 가장 자주 쓰이는 서비스 중 하나입니다.  
(출처 : [Amazone 공식 문서](https://aws.amazon.com/ko/pm/ec2/?gclid=CjwKCAjwnv-vBhBdEiwABCYQA5NebTCR6PunQjVTyIbACGOOsDrYFOX-TdUzK8smGoxjF5oxtsDX-xoCtYwQAvD_BwE&trk=4c74fd91-5632-4f18-ac76-a6c66c92e185&sc_channel=ps&ef_id=CjwKCAjwnv-vBhBdEiwABCYQA5NebTCR6PunQjVTyIbACGOOsDrYFOX-TdUzK8smGoxjF5oxtsDX-xoCtYwQAvD_BwE:G:s&s_kwcid=AL!4422!3!477203497843!e!!g!!ec2!11549843702!111422708806) )

EC2는 Elastic Compute Cloud로서 컴퓨팅 자원을 클라우드로 제공하는 서비스 이므로, IaaS(Infrastructure as a Service)입니다. 

# EC2 sizing & configuration options

- OS(Operating System) : Linux, Window, Mac OS
- CPU
- RAM
- Stroage Space : Network-attach(EBS & EFS, Hardware(EC2 Instance Store))
- Network Card: speed of the card, Public IP address
- Firewall rules
- Bootstrap script(EC2가 처음 시작 될 때의 설정): EC2 User Data

# EC2 User Data

EC2 User data script를 사용하면 Instance에 bootstrap할 수 있습니다. Bootstrapping이란, machine(EC2)가 처음 동작할 때, 명령을 실행하는 것을 뜻합니다. 이 script는 Instance가 처음 동작할 때만, 즉 한 번만 동작합니다.

EC2 user data는 아래와 같은 부팅 작업에 사용됩니다.
- Installing Updatas
- Installing Software
- Downloading common files from the internet

EC2 User Data Scripts는 root user로 동작합니다.

# EC2 Instance Types 

다양한 Use Case에 따라 EC2 Instance Type을 지정할 수 있습니다. 
![[Pasted image 20240325105357.png]]

위 그림처럼 Instance Type - 세대 - 속성 - 크기의 Convention이 있습니다.

AWS에서는 목적에 따른 Instance Type들을 추천합니다. 아래에서 좀 더 자세히 살펴보겠습니다.

## General Purpose
T, M, A1 타입이 여기에 속합니다.

![[Pasted image 20240325112748.png|600]]

웹 서버 또는 Code Repository와 같은 다양한 워크로드를 위한 Instance Type입니다.
Compute, Memory, Networking의 Balance가 잘 맞추어져 있습니다.

각 Instance Type의 디테일한 사양은 아래 링크에서 볼 수 있습니다.
https://aws.amazon.com/ko/ec2/instance-types/

## Compute Optimized
C 타입이 여기에 속합니다.
고성능의 Processor가 필요한, 계산집약적인 task에 적합한 Instance Type들 입니다. 

![[Pasted image 20240325112825.png|600]]

## Momory Optimized
R,X 타입이 여기에 속합니다.
메모리에서 대규모 데이터를 처리하는 워크로드에서 높은 성능이 필요할 때 사용합니다

![[Pasted image 20240325113203.png|600]]

## Storage Optimized
I,D,H 타입이 일반적으로 속합니다.
로컬 스토리지에 대규모 데이터셋의 순차적인 읽기/쓰기 작업에 사용됩니다.

OLTP systems(고빈도의 온라인 거래 시스템), 관계형 DB, NoSQL DB, Cache for in-momory DB(Redis),  Data warehousing Application 등에 사용됩니다.

![[Pasted image 20240325114443.png|600]]



---
관련 글
[[Amazone EC2(2)]]


참조 
https://aws.amazon.com/ko/pm/ec2/?gclid=CjwKCAjwnv-vBhBdEiwABCYQA5NebTCR6PunQjVTyIbACGOOsDrYFOX-TdUzK8smGoxjF5oxtsDX-xoCtYwQAvD_BwE&trk=4c74fd91-5632-4f18-ac76-a6c66c92e185&sc_channel=ps&ef_id=CjwKCAjwnv-vBhBdEiwABCYQA5NebTCR6PunQjVTyIbACGOOsDrYFOX-TdUzK8smGoxjF5oxtsDX-xoCtYwQAvD_BwE:G:s&s_kwcid=AL!4422!3!477203497843!e!!g!!ec2!11549843702!111422708806

https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ri-convertible-exchange.html