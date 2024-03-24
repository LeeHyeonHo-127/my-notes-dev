---
tags:
  - aws
date: 2024-03-22
title: AWS IAM (Identity and Access Management)
---
# IAM 이란 무엇일까?

AWS IAM(Identity and Access Management)는 AWS 리소스에 대한 액세스를 안전하게 제어할 수 있는 글로벌 서비스 입니다.

처음 AWS 계정을 생성하면, AWS 서비스와 리소스에 대한 모든 권한이 있는 단일 로그인 ID로 실행됩니다. 
이 ID는 AWS 계정 루트 사용자라고 합니다. AWS 에서는 일상적인 작업에 루트 계정을 사용하지 말라고 권고합니다. 루트 계정은 루트 계정으로만 할 수 있는 작업에만 사용을 권합니다. 
[루트 사용자 보안 인증이 필요한 작업](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/root-user-tasks.html)


# IAM: User & Group

IAM에는 User, Group 이라는 개념이 있습니다. IAM으로 필요에 따라 User, Group에 최소한의 권한을 부여해서 권한을 관리할 수 있습니다.

- Root Account : 기본적으로 생성되는 계정입니다. 공유하거나 일상적인 작업에 사용하지 않는 것을 추천합니다.
- User : 사람에 해당하는 개념입니다. 그룹안에 속할 수 있습니다.
- Groups : User들을 포함하는 개념입니다. Group안에 다른 Group은 속할 수 없습니다. 한 User가 여러 그룹에 속할 수는 있습니다.
![[Pasted image 20240322150011.png]]


# IAM: Permissions

User 또는 Group 그리고 Group에 **Policy**라고 불리우는 JSON Document를 할당합니다. Policy는 사용자의 권한을 정의합니다.
AWS는 최소 권한의 법칙을 적용합니다. 

![[Pasted image 20240322150509.png | 400]]

User와 Group에 Policy를 부여하면 아래와 같습니다.

![[Pasted image 20240322150651.png|500]]



# IAM Policies Structure

![[Pasted image 20240322154653.png]]
### IAM Policies Consist Of
- Version: 정책 언어의 버전입니다.
- ID: 정책의 id입니다.(Optional)
- Statement: 하나 이상의 개별 상태입니다(필수)

### Statements Consists of
- Sid: Statement의 id입니다.
- Effect: Statement를 Allow인지 Deny인지 선택합니다.
- Principal: 이 정책이 적용된 account, user, role을 정의합니다. Group은 포함되지 않습니다.
	Principal은 리소스 기반 정책에 사용됩니다. 리소스 기반 정책은 리소스에 연결되는 정책입니다.
- Action: 이 정책이 Allow(or Deny)할 action들의 list입니다. 즉 뭘 하게(못하게) 할 것인지 정의합니다.
- Resource: 이 정책이 적용될 resource들의 리스트를 정의합니다.
- Condition: 정책이 적용될 조건을 정의합니다.(Optional)

# IAM - Password Policy

AWS 계정에서 사용자 지정 암호 정책을 설정하여 IAM 사용자 암호의 복잡성 요건과 의무적인 교체 주기를 지정할 수 있습니다.

Password의 최소길이, 구체적인 character type, 암료 만료 활성화, 암호 재사용 재한 등을 설정할 수 있습니다.


# Multi Factor Authentication - MFA

MFA(Multi Factor Authentication)을 활용하여 보안을 강화할 수 있습니다. MFA를 사용하면 password가 분실되거나 해킹되어도 보안을 유지할 수 있습니다.

MFA = password you know + security device you own
![[Pasted image 20240322173239.png]]
### MFA device options in AWS
- virtual MFA Device
- Universal 2nd Factor Security Key
- Hardware Key Fob MFA Device
- Hardware Key Fob MFA Device for AWS GovCloud(US)

# How can users access AWS?

AWS에 접근하는 방법은 3가지 입니다.
1. AWS Management Console: protected by password + MFA
2. AWS Command Line Interface (CLI) : protected by access keys
3. AWS Software Developer Kit(SDK) - for code : protected by access keys

### Access Keys 뭐지?
User가 관리하는, AWS 계정에 접근할 수 있게 하는 Key입니다.
Access Key ID, Secret Access Key로 이루어져 있습니다. 일종의 ID, PW라고 이해해도 됩니다.


# IAM Roles for Services

특정 AWS Service들을 사용자를 대신하여 작업을 수행해야 합니다. (자동화 등의 이유로). 그러기 위해서 IAM Roles를 사용해 AWS Service에게 권한을 부여해야 합니다.

![[Pasted image 20240322174246.png]]
Common roles:
- EC2 Instance Roles
- Lambda Function Roles
- Roles for CloudFormation

# IAM Security Tools

### IAM Credentials Report (account-level)
- 계정의 모든 사용자와 자격증명을 나열하는 보고서 입니다.

### IAM Access Advisor (user-level)
- Access Advisor는 사용자에게 부여된 권한과, 언제 마지막으로 서비스 접근했는지를 보여줍니다.
- 이 정보를 바탕으로 정책을 수정할 수 있습니다.

# IAM Guidelines & Best Practice

- AWS 계정 설정 외에는 root 계정을 사용하지 않기
- One physical user = One AWS user
- User를 Group에 할당하고, Group에 권한을 부여하기
- Strong password policy 만들기
- MFA 사용하기
- Roles을 만들어 AWS Service에 권한주기
- CLI, SDK 같은 Programmatic Access에 Access Key 사용하기
- IAM Credentials Report나 IAM Access Advidor를 사용해서 계정 권한을 감시하기
- IAM Users, Access Keys 공유하지 않기

---
참고자료
https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/introduction.html
https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html