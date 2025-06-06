# 📝 Site-to-Site VPN 실습 상세 단계

## ✅ 1. VPC 생성

### VPC A (Seoul Region)
- Name: vpc-01
- CIDR: 10.1.0.0/16
- 서브넷: 10.0.1.0/24 (public)
- 인터넷 게이트웨이 연결: vpc-01-igw
- Enable DNS hostnames 

### VPC B (Tokyo Region)
- Name: vpc-04
- CIDR: 10.4.0.0/16
- 서브넷: 192.168.1.0/24 (public)
- 인터넷 게이트웨이 연결: vpc-04-igw
- Enable DNS hostnames 

---
## ✅ 2. Internet Gateway 생성 및 연결 
### IGW (Seoul Region)
- Name: vpc-01-igw

### IGW (Tokyo Region)
- Name: vpc-04-igw

---
## ✅ 3. Subnet 생성 
### Subnet (Seoul Region)
- Name: vpc-01-public-subnet-a
- CIDR: 10.1.1.0/24

### Subnet (Tokyo Region) 
- Name: vpc-04-public-subnet-a
- CIDR: 10.4.1.0/24

---
## ✅ Security Group 생성 및 수정
### Security Group (Seoul Region)
- Name: vpc-01-private-ec2-sg
- Inbound rule
  - Type Custom ICMP: Source 10.4.0.0/16: Description allow icmp for vpc-04

### Security Group (Tokyo Region)
- Name: vpc-04-public-ec2-a1
- Inbound rule
  - Type Custom ICMP: Source 10.1.0.0/16: Description allow icmp for vpc-01



---

## ✅ 4. EC2 인스턴스 생성
### EC2 (Seoul Region)
- Name: vpc-01-public-ec2-a1
- AMI: Amazon Linux 2
- Instance type: t2.micro
- Key pair: ec2-public-seoul
  - Key pair type: RSA
  - Private key file format: .pem
- Security Group:
  - Create security Group
    - Name: vpc-01-public-ec2-sg
    - Description: security group for vpc-01-public-ec2
- Advanced details
  - Metadata version: V1 and V2 (token optional)
  - Allow tags in metadata: Enable

- User data
```bash
#!/bin/bash
dnf update -y
dnf install -y httpd wget php-fpm php-mysqli php-json php php-devel
dnf install -y mariadb105-server
systemctl start httpd
systemctl enable httpd
usermod -a -G apache ec2-user
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;
```

### EC2 (Tokyo Region)
- Name: vpc-04-public-ec2-a1
- AMI: Amazon Linux 2
- Instance type: t2.micro
- Key pair: ec2-public-tokyo
  - Key pair type: RSA
  - Private key file format: .pem
- Security Group:
  - Create security Group
    - Name: vpc-04-public-ec2-sg
    - Description: security group for vpc-04-public-ec2
- Advanced details
  - Metadata version: V1 and V2 (token optional)
  - Allow tags in metadata: Enable



---
## ✅ 5. Elastic IP 생성 및 할당 
- Name: eip-vpc-01-public-ec2-a
Actions > Associate Elastic IP address > Instance(vpc-01-public-ec2-a1)

--- 
## ✅ 6. Route Table 생성 
### RT Seoul
- Name: vpc-01-public-subnet-rt
- VPC: vpc-01
Subnet associations > Edit subnet associations > vpc-01-subnet-a
Routes > Edit routes > Destination 0.0.0.0/0: Target vpc-01-igw

### RT Tokyo
- Name: vpc-04-public-subnet-rt






---
## ✅ 3. Virtual Private Gateway 설정

- Name: vpc-01-vgw
Attach to VPC

---




## ✅ 4. Customer Gateway 설정

- Name: vpc-04-cgw (Region: Seoul)
- IP 주소: 57.180.39.205 
- Routing: Static
- BGP: 사용 안함

---

## ✅ 5. Site-to-Site VPN Connection 설정

- Name: vpc-0104-vpn (Region: Seoul)
- Virtual Private Gateway: vpc-01-vgw
- Customer Gateway: vpc-04-cgw
- Static IP prefixes: 10.4.0.0/16
Download configuration > Openswan (Libereswan의 베이스) 


## Tokyo에서 Libreswan 설정하기 
```bash
dnf install libreswan -y
vi /etc/sysctl.conf

---
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.default.accept_source_route = 0
---
vi /etc/ipsec.conf
---
include /etc/ipsec.d/*.conf
---
vi /etc/ipsec.d/aws.conf

```
![image](https://github.com/user-attachments/assets/df4d909d-49f2-4d68-a5aa-c65968fc73c8)

```bash
vi /etc/ipsec.d/aws.secrets
```
![image](https://github.com/user-attachments/assets/c84600e1-9fa6-4803-9f9a-565480a06d59)
```bash
service ipserc start
chkconfig ipsec on
service ipsec status
```

## vpc-01의 Route table 수정
- Edit routes
  - Destination 10.4.0.0/16: Target vpc-01-vgw

# 실습 결과
![image](https://github.com/user-attachments/assets/ec3e39a4-532b-47c6-aa32-e5771c5e000e)


![image](https://github.com/user-attachments/assets/6a259dc2-6c4d-48ee-9bbb-f3940d4824ca)
