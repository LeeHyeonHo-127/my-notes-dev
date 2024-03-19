---
tags:
  - 리눅스
date: 2024-03-19
title: Bastion Host 실습
---
# Bastion Host란?

> [!NOTE]
> Bastion Host는 외부 네트워크에서 내부 혹은 private network 로의 접근을 관리하기위한 서버입니다. 
> (Jump box 혹은 jump server 라고 불리우기도 합니다.)  최근의 Bastion Host 들은 Internet에 존재하기 때문에 attack surface를 줄이기 위해 최소한의 서비스만 동작시킵니다. 일반적으로 proxy나 log communication(ssh session 같은)에 사용됩니다.
> 출처 : https://www.strongdm.com/what-is/bastion-host
> 

보안을 위해 사용하는 내부 네트워크의 출입구 정도라고 이해하고 있습니다.
# Bastion Host실습

Bastion Host 실습을 해봅시다.
아래 그림과 같은 구조를 만들어 봅니다.

1. 내부 네트워크에 웹서버와 DB 서버를 동작하고 Bastion Host를 통해서 들어올 수 있게 합니다.
2. 각 DB와 Web서버는 Haproxy를 통해 로드밸런싱 됩니다.
3. DB는 Master, Slave가 존재합니다 Slave는 자동으로 Master의 내용을 백업합니다.
4. Bastion에는 자체 서명된 서명서가 있어 외부에서 Https로 접근합니다

![[KakaoTalk_20240319_150341518 1.jpg|600]]

총 4대의 서버가 필요합니다. 
실습을 위해서 하나의 기기가 여러 기능을 수행하기도 합니다.

각 기기는 아래 기능을 수행합니다
CentOS 7 : NAT Gateway, HAProxy, 방화벽 -> Bastion Host
CentOS 8 : WEB 서버 01, DB-Master
CentOS 8-2 : WEB 서버 02, DB-Slave
Window2003-1 : WEB 서버 03, DNS 서버

## Setting 하기


```
1. 실습 VM 
NAT / HAproxy CENTOS 7
GW 10.10.10.1/24
공인 IP 192.168.1.111

2. WEB01, DB 서버(Maser) => CENTOS 8
10.10.10.10/24 GW 10.10.10.1 DNS 8.8.8.8

3. WEB02, NFS 서버, DB서버(Slave) => CENTOS 8-2
10.10.10.20/24 GW 10.10.10.1 DNS 8.8.8.8

4. WEB03, DNS, 원격데스크톱 연결 TCP 3389 => WIN2003-1 
10.10.10.30 255.255.255.0 GW 10.10.10.1 DNS 8.8.8.8
```
#### Centos7 (Bastion Host)

Lan 카드 하나 추가 후 실행합니다.
Lan 카드를 하나 추가하는 이유는 Bastion Host가 NAT Gateway 역할을 수행하기 위해서입니다. 
외부 IP를 Private IP로 변경하기 위해서는 외부 네트워크, 내부 네트워크를 위한 각각의 네트워크 카드가 필요합니다.  (디테일한 부분은 잘 모르겠습니다..)

저는 VM를 사용하고 있어 간단하게 추가할 수 있습니다.

아래 명령어를 통해 확인하면 ens33, ens37이라는 두 개의 네트워크 카드가 있음을 알 수 있습니다.
ens는 (Ethernet Network Interface Card의 약자입니다.)

```
ip addr
```
![[Pasted image 20240319103341 1.png]]

저는 ens37을 Private Network로 사용합니다.

NetworkManager를 사용하여 실습을 위한 네트워크를 세팅해줘야합니다.
명령어는 nmtui를 입력하면 아래와 같은 창이 나타납니다.

```
nmtui
```

![[Pasted image 20240319103420 1.png|300]]
![[Pasted image 20240319103434 1.png|500]]

ens37 이라는 networkcard는 내부 IP용으로 사용할 것이기 때문에
10.10.10.1/24로 설정합니다.
![[Pasted image 20240319103535 1.png|500]]

아래 명령어로 ens37 네트워크 연결을 활성화 합니다.
```
nmcli con up ens37
```

Bastion Host의 Setting은 끝났습니다.  


#### CentOS 8

CentOS8의 세팅을 해줍니다.
ens160이라는 네트워크 카드의 IP를 10.10.10.10으로 지정해줍니다
Gateway는 10.10.10.1로 지정해줍니다.
(Bastion Host의 네트워크 카드의 IP가 10.10.10.1입니다)
![[Pasted image 20240319104501 1.png|500]]


#### Centos 8-2

Centos 8-2도 마찬가지로 세팅해줍니다.
![[Pasted image 20240319104706 1.png|500]]

#### Window2003-1

Window2003-1도 세팅합니다.
사설 IP는 10.10.10.30으로, Gateway는 10.10.10.1로 세팅합니다.
![[Pasted image 20240319105007 1.png]]

#### (Optional) IP를 도메인으로 매핑

Centos7/8/8-2 에서 각 IP와 도메인을 매핑해줍니다.

vi /etc/hosts
10.10.10.10 WEB01
10.10.10.20 WEB02
10.10.10.30 WEB03

## 키인증하기

저는 VMWare를 사용하고있습니다.
VMWare는 불편한 점이 많아 XSHELL을 사용할 것입니다. Bastion Host인 CentOS 7을 제외한 나머지 기기들은 Private Subnet에 존재하므로 외부에서 (가상 머신 밖에서) 접근할 수 없습니다.

때문에 CentOS7에서 Key를 만들어 SSH로 Private Subnet의 기기에 접속할 수 있게 하여 XSHELL에서 CentOS7/8/8-2 를 접속할 수 있게 하겠습니다.

#### CentOS7

ssh-keygen으로 ssh 접근을 위한 key를 만들어줍니다.
```
ssh-keygen
```
![[Pasted image 20240319105552 1.png]]

키가 잘 생성되었습니다.
![[Pasted image 20240319105608 1.png]]

ssh-copy-id 명령어를 사용해 ssh 공개키를 각 서버에 복사합니다.
```
ssh-copy-id 10.10.10.10 // 공개키 복제
ssh-copy-id 10.10.10.20 // 공개키 복제
```


이제 ssh로 CentOS8/8-2 에 각각 접근해 보겠습니다.
![[Pasted image 20240319105924 1.png]]
![[Pasted image 20240319105959 1.png]]
잘 됩니다. 

## 마스커레이드(Masquerate) 

#### IP Masquerade란?

내부 컴퓨터들이 리눅스 서버를 통해서 인터넷 등 다른 네트워크에 접속할 수 있도록 해주는 기능입니다.
저희 실습에서는 Private Subenet에 있는 CentOS 8/8-2 Win2003-1 이 CentOS7(Bastion Host)를 통해서 외부 네트워크로 나갈 수 있게 합니다.

#### Centos7

아래 명령어로 firewalld를 실행시켜줍니다.
```
yum -y install bash-completion // 자동완성 패키지
systemctl enable --now firewalld 
```


아래 명령어로 현재 활성화된 방화벽 영역을 확인합니다.

현재 ens33, ens37 네트워크 카드에 대해서 활성화 되어있음을 알 수 있습니다.
```
firewall-cmd --get-active-zones
```
![[Pasted image 20240319111654 1.png]]

저희는 하나는 내부, 하나는 외부 네트워크를 위해 지정해 주어 NAT Gateway 역할을 수행해야 합니다.
아래 명령어로 ens33은 외부 네트워크를 위한 모드로, ens36은 내부 네트워크를 위한 모드로 설정합니다.
```
nmcli c mode ens33 connection.zone external
nmcli c mode ens36 connection.zone internal
```
![[Pasted image 20240319111743 1.png]]


/etc/sysctl.conf 파일은 리눅스 커널의 다양한 설정을 제어하는 데 사용되는 구성 파일입니다.
해당 파일을 수정하여 Centos 7이 IP 패킷 포워딩 기능을 활성화하게 합니다.
IP 패킷 포워딩 기능을 활성화하여, NAT 기능을 수행할 수 있게 합니다.


```
vi /etc/sysctl.conf

net.ipv4.up_forward=1 <- 추가
```
![[Pasted image 20240319111825 1.png]]

아래 명령어로 CentOS7을 리부트합니다.
```
reboot
```

마스컬레이딩이 됐으므로 외부망으로 ping이 보내집니다.
![[Pasted image 20240319112002 1.png]]



## 포트포워딩

마스컬레이드를 통해 Private Subnet에서 외부 네트워크로 나갈 수 있게 설정해주었습니다. 
이를 SNAT라고 부릅니다.

이제 외부에서 Private Subnet으로 찾아들어올 수 있게(DNAT) 설정해줍니다.
이를 포트포워딩을 통해 구현합니다.

포트포워드를 구현후, private subnet 외부에서 private subenet 내부 PC(Win2003)에 접속해 보겠습니다.

#### win2003-1

win2003-1에서 원격 데스크톱을 사용할 수 있게 설정해줍니다.

내 컴퓨터 -> 속성
![[Pasted image 20240319112127 1.png]]

#### Centos07

아래 명령어를 통해서 3389 port(원격 데스크톱 프로토콜)로 들어오는 요청은 10.10.10.30:3389로 보내도록 포트포워딩합니다.
```
firewall-cmd --permanent --zone=external --add-forward-port=port=3389:proto=tcp:toport=3389:toaddr=10.10.10.30

firewall-cmd -reload
```
![[Pasted image 20240319112445 1.png]]

#### Win10(외부 기기)

외부 기기에서 192.168.1.111(Bastion Host의 External 네트워크 카드의 IP) 로 접속을 합니다.
![[Pasted image 20240319112540 1.png]]

잘 들어가지는 것을 볼 수 있습니다.
![[Pasted image 20240319112809 1.png]]


## 웹 서버 설치

이제 웹 서버를 설치해봅니다.
Centos8/8-2에 Apatch 웹 서버를 설치한 후,
구분을 위한 기본 html을 생성할 것입니다.

Win 2003에서도 web 서버를 구축해 줄 것입니다.
#### Centos8

```
yum -y install httpd
cd /var/www/html
echo WEB01 > index.html
```

#### Centos8-2

```
yum -y install httpd
cd /var/www/html
echo WEB02 > index.html
```

잘 보입니다.
![[Pasted image 20240319113412 1.png]]

#### Win 2003-1 (WEB03)

아래 설정을 통해 web03을 구성합니다.
![[Pasted image 20240319113557 1.png]]


#### Centos8

확인해보면 잘 보입니다.
![[Pasted image 20240319113707 1.png]]


## HAProxy

로드밸런싱을 위해 HAProxy를 구성해봅시다.
저는 HAProxy를 통해 웹 서버와 DB를 로드밸런실 할 것입니다.

#### CentOS 7

HAProxy를 설치합니다.
```
yum -y install haproxy
```


haproxy.cfg 파일을 수정해줍니다.
어디로 들어오는 요청을(front) 어디로 로드밸런싱 할지(backend) 구성해줍니다.
```
 vi /etc/haproxy/haproxy.cfg
```

아래 설정을 통해 80 포트로 들어오는 모든 IP는 roundrobin 방식으로 웹 서버 3개로 로드밸런싱 됩니다.
![[Pasted image 20240319114146 1.png]]
![[Pasted image 20240319114123 1.png]]
![[Pasted image 20240319114245 1.png]]

haproxy를 실행하고 확인해봅니다.
안될 이유가 없습니다.
![[Pasted image 20240319114314 1.png]]
![[Pasted image 20240319114402 1.png]]

안 들어가집니다.
안될 이유가 있었습니다.
![[Pasted image 20240319114447 1.png]]

방화벽에 막혔었습니다.

Bastion Host가 80요청을 방화벽을 통해 막았으니, HAProxy는 로드밸런싱 할 수 없었습니다.

아래 명령어로 http를 방화벽에서 열어줍니다.
```
firewall-cmd --permanent --add-service=http --zone=external
firewall-cmd --reload
```

외부로 열린 방화벽을 확인해 봅니다.
```
firewall-cmb --zone=external --list-all
```
http가 잘 열려있습니다.
이제는 진짜 안 될 이유가 없습니다.
![[Pasted image 20240319114704 1.png]]

잘 넘어가집니다
로드밸런싱도 잘 됩니다.
![[Pasted image 20240319114759 1.png]]
![[Pasted image 20240319114821 1.png]]

## 웹 서버에 방화벽 올리기

현재 Bastion Host에는 방화벽이 동작하고 있습니다.
하지만 내부 Private Subnet의 웹서버에는 방화벽이 없습니다. 방화벽을 설치해줍시다.

#### Centos8, 8-2

Centos8. 8-2에 방화벽을 동작시킵니다.
```
 systemctl enable --now firewalld
```

이제 웹 서버 01, 02는 방화벽에 막혀서 WEB03만 보입니다.
![[Pasted image 20240319115410 1.png]]

다시 열어봅시다.


```
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-all
```
![[Pasted image 20240319115631 1.png]]

다시 잘 보입니다
![[Pasted image 20240319115756 1.png]]

Bastion Host 뿐만이 아니라 웹 서버에서도 방화벽을 통해 이중으로 보안을 강화할 수 있다는 것을 알 수 있습니다.

## DNS Server

IP로는 웹 서버에 들어갈 수 있습니다. 하지만 저희는 도메인을 통해 웹 서버에 접속합니다.
DNS 서버를 구축해서 도메인으로 접속해 봅시다.

#### Win-2003

Win-2003에 DNS 서버를 구축합니다.

![[Pasted image 20240319121137 1.png]]

web서버와 ip를 적절하게 매핑합니다.
![[Pasted image 20240319121219 1.png]]![[Pasted image 20240319121259 1.png]]

이제 이 DNS 서버를 통해 IP를 찾도록 설정하면 도메인을 통해 웹서버에 접근할 수 있습니다.

#### CentOS 8-2

DNS 서버(10.10.10.30)을 resolv.conf에 추가
resolv.conf는 DNS 서버 설정을 담당하는 텍스트 파일입니다.


```
vi /etc/resolv.conf
```
![[Pasted image 20240319121351 1.png]]

이제 도메인을 통해서 웹서버에 접근할 수 있습니다.
훨씬 인간친화적입니다.
![[Pasted image 20240319121446 1.png]]

## 인증서를 사용한 보안

현재는 HTTP를 통해서 웹서버에 접근합니다. 이는 보안상 취약한 방법입니다.
자체 인증서를 생성하여 https로 접근하게 설정합니다.

Bastion Host에서 https로 인증서를 확인하고, haproxy를 통해서 http로 웹 서버에 접속하게 설정할 것입니다.

아래 명령어를 통해 인증서를 생성합니다.
```
cd /etc/pki/tls/certs 
openssl req -x509 -nodes -newkey rsa:2048 -keyout /etc/pki/tls/certs/haproxy.pem -out /etc/pki/tls/certs/haproxy.pem -days 365 
```

haproxy.pem은 개인키와 인증서가 들어있는 파일입니다.
![[Pasted image 20240319133348 1.png]]

소유자만 읽고 쓸 수 있도록 권한을 변환합니다.
```
chmod 600 haproxy.pem 
```

HAProxy에서 443 포트로 요청을 받고, 인증서를 검증하도록 수정합니다.
```
vi /etc/haproxy/haproxy.cfg
```
![[Pasted image 20240319133535 1.png]]

인증서를 넣어줬으니, 방화벽에서 443 port를 열어줍시다.
```
firewall-cmd --permanent --add-service=https --zone=external
firewall-cmd --reload
```


https로 들어가집니다.
![[Pasted image 20240319133925 1.png]]

하지만 loadbalancing이 안 됩니다.


```
vi /etc/haproxy/haproxy.cfg
```
![[Pasted image 20240319134050 1.png]]
haproxy 설정 변경후 잘 됩니다.

## MySQL(MariaDB) Replication - DB 실시간 이중화

웹 서버의 로드밸런싱이 완료 되었으니, 이제 DB의 로드벨런싱을 구현합니다.
DB의 로드밸런싱을 위해서는 DB 끼리의 동기화를 구현해야합니다.
즉, DB들의 정합성을 보장해야합니다. 

우선 DB를 설치해 줍시다
CentOS 8/8-2에 DB를 설치합니다
```
yum -y install mariadb-server
```

#### web01 (Centos8 10.10.10.10) - master

mariadb-server.cnf는 MariaDB 서버 설정 파일입니다. 
들어가 줍니다.
```
vi /etc/my.cnf.d/mariadb-server.cnf
```

맨 마지막줄에 추가
```
log-bin = mysql-bin // 바이너리 로그 파일의 이름을 mysql-bin으로 지정
server-id = 1  // 서버 ID는 1로 지정, 복제 환경에서 식별을 위해 지정 
binlog_format = row //바이너리 로그 형식을 row로 지정. 각 행의 변경 내용을 기록
expire_logs_days = 2 //바이너리 로그는 2일 후 사라지게 설정
```
![[Pasted image 20240319135848 1.png]]

위 설정을 통해서 Master DB의 내용은 mysql-bin이라는 바이너리 로그파일에 저장됩니다.
slave-db에서 해당 로그를 읽어서 DB의 Sync를 맞춥니다.

mysql에 들어간 후 아래 명령어를 입력합니다.
```
grant replication slave on *.* to 'slave_db'@'%' identified by '12345678';
```
grant는 권한을 부여하는 SQL 키워드입니다.
slave_db 라는 사용자에게 모든 데이터베이스의 모든 테이블에 대한 복제 슬레이브 권한을 부여합니다.
이제 slave_db 라는 사용자는 slave-db 로서 master db의 내용을 복제할 수 있습니다.


아래 명령어를 통해 master db의 상태를 확인합니다.
```
show master status

+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 |      342 |              |                  |

```


#### web02 (Centos8-2 10.10.10.20) - slave

Master DB의 설정이 끝났으니, Slave DB의 설정을 해줍시다.

마찬가지로 mariadb-server.cnt에 들어갑니다.
```
vi /etc/my.cnf.d/mariadb-server.cnf
```

맨 마지막 줄에 아래 내용을 추가합니다.
```
log-bin = mysql-bin 
server-id = 2 
binlog_format = row 
expire_logs_days = 2
```


mysql로 들어갑니다.
master status를 참조해서 아래 값 작성 후 mysql에 넣습니다.
master db의 host와 user, pw, logfile, log_position을 기입하여 어떤 master_db의 값을 복제할 것인지 지정합니다.
```
CHANGE MASTER TO 
        MASTER_HOST="10.10.10.10", 
        MASTER_USER="slave_db", 
        MASTER_PASSWORD="12345678", 
        MASTER_PORT=3306, 
        MASTER_LOG_FILE="mysql-bin.000002", 
        MASTER_LOG_POS= 342
```
![[Pasted image 20240319140717 1.png]]


```
show slave status; 
```
를 하면 연결 상태를 볼 수 있습니다.
10.10.10.10과 연결되었음을 볼 수 있습니다.
![[Pasted image 20240319140808 1.png]]


이제 한 번 확인해 봅시다

확인을 위해master에서 db를 만듭니다.
![[Pasted image 20240319140903 1.png]]

slave에서 확인합니다.
안 될 이유가 없습니다.
![[Pasted image 20240319140946 1.png]]

어
안 보입니다.

이번에도 방화벽이 문제였습니다.

방화벽을 멈추고 다시 DB 세팅을 해줍니다.
```
systemctl stop firewalld
```

다시 세팅합니다.
```
// mysql에 들어간 뒤

stop slave;

CHANGE MASTER TO 
        MASTER_HOST="10.10.10.10", 
        MASTER_USER="slave_db", 
        MASTER_PASSWORD="12345678", 
        MASTER_PORT=3306, 
        MASTER_LOG_FILE="mysql-bin.000002", 
        MASTER_LOG_POS= 342
        
start slave;
```


Slave DB에서 database를 확인해 봅니다.
이제 잘 보입니다.
![[Pasted image 20240319141225 1.png]]


## LoadBalancer로 DB 연결하기

거의 다 왔습니다.
DB의 정합성을 충족하였으니, Loadbalancing을 할 준비는 다 되었습니다.
HAProxy를 통해 로드밸런싱 해줍니다.


다시 haproxy.cfg에 들어가서 haproxy 설정을 수정해줍시다.
```
vi /etc/haproxy/haproxy.cfg
```

파일 아래에 아래 코드 내용을 붙여넣어 줍니다.
```
frontend  mysql-in
    bind *:3306
    default_backend    db_servers

backend db_servers
    balance            roundrobin
    server             db01 10.10.10.10:3306 check
    server             db02 10.10.10.20:3306 check
```
![[Pasted image 20240319143726 1.png]]

설정후 haproxy를 재시작 해줍니다.
```
systemctl restart haproxy
```


아까와 같은 실수를 반복하지 않습니다.
방화벽을 열어줍니다.


```
firewall-cmd --permanent --add-service=mysql --zone=external
firewall-cmd --reload
```


확인 해봅니다.

```
 firewall-cmd --zone=external --list-all
```

시원하게 잘 열렸습니다
![[Pasted image 20240319143903 1.png]]

로드밸런싱이 잘 되는지 확인해봅시다.

Centos7 (Bastion Host에서) 아래 명령어로 접속합니다.
안 될 이유가 없습니다.
```
mysql -h 192.168.1.111 -u slave_db -p
```

아

안됩니다.

접속은 되지만 master db가 안 보입니다.
![[Pasted image 20240319144421 1.png]]


grant로 복제에 대한 권한은 줬지만 원격 접속은 주지 않았습니다.
master와 slave에 다시 grant로 원격 접속의 권한을 부여합니다.

```
 grant all privileges on *.* to  root@'%' identified by '4321';
```

Bastion이 아니라 새로운 기기에서 접속해봅시다.
#### Linus01 (새로운 리눅스)


```
mysql -h 192.168.1.111 -u root -p
```

연결이 잘 됩니다.
![[Pasted image 20240319145115 1.png]]


로드밸런싱이 되는지 확인해봅시다.
slave에 database를 하나 만듭니다.

![[Pasted image 20240319145201 1.png]]
slavedb라는 db를 만들었습니다.
(slave의 내용은 master로 백업되지 않는다)


하나씩 들어가봅니다.
같은 192.168.1.111로 들어가면 하나는 slave로 하는 master db로 들어갑니다.
(slavedb를 보고 알 수 있습니다.)
![[Pasted image 20240319145526 1.png]]


이것으로 Bastion Host를 통해서 내부 네트워크를 외부로 부터 격리하고,
HAProxy를 사용해 로드밸런싱 해보았습니다.
