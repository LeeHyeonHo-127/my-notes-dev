---
tags:
  - aws
date: 2024-03-25
title: Security Groups
---
# Introduction to Security Groups

Security Group(보안그룹)은 AWS에서의 기본적인 네트워크 보안입니다. 
Security Group은 EC2에서 나가는, 혹은 EC2로 들어오는 Traffic 중 어떤 Traffic을 허용할지를 결정합니다.
![[Pasted image 20240325120028.png]]
Security Group은 Allow Rules만 허용합니다. 보안그룹 규칙은 IP 혹은 security group를 참조하여 rule을 정합니다.

# Security Groups Deeper Dive

Securiy Groups은 "방화벽"처럼 EC2에서 동작합니다.
Security Groups은 아래 요소들을 규제하여 보안을 유지합니다.
- Access to Ports
- Authorised IP ranges - IPv4 and IPv6
- Control of inbound network (from other to the instance)
- Control of outbound network (from other to the instance)
![[Pasted image 20240325120946.png]]
![[Pasted image 20240325121025.png]]

# Security Groups Good to know

- 하나의 Security Group을 여러 Instance에 사용할 수 있습니다
- region/VPC 조합에 종속됩니다
- ssh access를 위해 별도의 보안 그룹을 유지하는게 좋습니다
- 시간 초과로 어플리케이션에 접근을 못할 경우 대부분 보안 그룹 문제입니다
- default로 inbound는 blocked 되어 있습니다.
- default로 outbound는 authorised 되어 있습니다.

## Referencing other security groups Diagram

Security Group은 다른 Security Group을 참조해서 규칙을 정할 수 있습니다.
![[Pasted image 20240325141656.png]]

![[Pasted image 20240325142909.png]]
(다른 보안 그룹을 이용해 rule을 지정하는 실습입니다. 위 다이어그램의 보안 그룹과는 무관합니다. )

---
관련 글
[[[[Udemy] - Amazone EC2(1)]]
