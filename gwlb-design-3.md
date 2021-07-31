---
description: 'Update : 2021-07-01'
---

# GWLB Design 3

### 목표 구성 개요

2개의 각 워크로드 VPC \(VPC01,02\)은 Account내에 구성된 GWLB 기반의 보안 VPC를 통해서 내, 외부 트래픽을 처리하는 구성입니다. GWLB 기반의 보안 VPC는 2개의 AZ에 4개의 가상 Appliance가 로드밸런싱을 통해 처리 됩니다.

이러한 구성은 VPC Endpoint를 각 VPC에 분산하고, GWLB에 VPC Endpoint Service를 연결하는 분산형 구조입니다. 각 VPC01,02는 외부에 서비스를 제공하기 위해 ALB를 통해 웹 서비스를 제공하고 있으며, Private Subnet의 인스턴스는 타겟 그룹으로 연결되어 있습니다. 해당 인스턴스들을 패치 관리를 위해서 NAT Gateway를 통해 Source NAT 서비스를 받게 됩니다.

앞선 [GWLB Design1](gwlb-design1.md) 구성보다 , 외부 서비스에 더 중점을 둔 디자인입니다.

아래 그림은 목표 구성도 입니다.

![](.gitbook/assets/image%20%28100%29.png)

## Cloudformation기반 VPC 배포

### 1.VPC yaml 파일 다운로드

Cloud9 콘솔에서 아래 github로 부터 VPC yaml 파일을 다운로드 합니다. \(앞선 LAB에서 다운로드 받은 경우에는 실행하지 않습니다.\)

```text
git clone https://github.com/whchoi98/gwlb.git

```

Cloud9에서 로컬로 파일을 다운로드 받습니다.

![](.gitbook/assets/image%20%28105%29.png)

### 2.AWS 관리콘솔에서 VPC 배포

AWS 관리콘솔에서 Cloudformation을 선택합니다.

![](.gitbook/assets/image%20%28115%29.png)

앞서 다운로드 해둔 yaml 파일 중에서, 아래 그림과 같이 GWLBVPC.yml 파일을 선택합니다.

![](.gitbook/assets/image%20%28111%29.png)

스택 세부 정보 지정에서 , 스택이름과 VPC Parameters를 지정합니다. 대부분 기본값을 사용하면 됩니다.

* 스택이름 : GWLBVPC
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.254.0.0/16
* PublicSubnetABlock: 10.254.11.0/24
* PublicSubnetBBlock: 10.254.12.0/24
* InstanceTyep: t3.small
* KeyPair : 미리 만들어 둔 keyPair를 사용합니다.

![](.gitbook/assets/image%20%28107%29.png)

다음 단계를 계속 진행하고, 아래와 같이 "AWS CloudFormation에서 IAM 리소스를 생성할 수 있음을 승인합니다."를 선택하고, 스택을 생성합니다.

![](.gitbook/assets/image%20%28116%29.png)

3~4분 후에 GWLBVPC가 완성됩니다.

AWS 관리콘솔 - VPC - 가상 프라이빗 클라우드 - 엔드포인트 서비스 를 선택합니다. Cloudformation을 통해서 VPC Endpoint 서비스가 이미 생성되어 있습니다. 이것을 선택하고 세부 정보를 확인합니다.

서비스 이름을 복사해 둡니다. 뒤에서 생성할 VPC들의 Cloudformation에서 사용할 것입니다.

![](.gitbook/assets/image%20%28109%29.png)

VPC01,02 2개의 VPC를 Cloudformation에서 앞서 과정과 동일하게 생성합니다. 다운로드 받은 Yaml 파일들 중에 VPC01.yml, VPC02,yml을 차례로 선택해서 생성합니다.

![](.gitbook/assets/image%20%28104%29.png)

스택 이름을 생성하고, GWLBVPC의 VPC Endpoint 서비스 이름을 "VPCEndpointServiceName" 에 입력합니다. 또한 나머지 파라미터들도 입력합니다. 대부분 기본값을 사용합니다.

* 스택이름 : VPC01, VPC02
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.1.0.0/16 \(VPC01\), 10.2.0.0/16 \(VPC02\)
* GWLB SubnetABlock: 10.1.1.0/24\(VPC01\),10.2.1.0/24\(VPC02\)
* GWLB SubnetBBlock: 10.1.2.0/24 \(VPC01\), 10.2.2.0/24 \(VPC02\)
* PublicSubnetABlock: 10.1.11.0/24 \(VPC01\), 10.2.11.0/24 \(VPC02\)
* PublicSubnetBBlock: 10.1.12.0/24 \(VPC01\), 10.2.12.0/24 \(VPC02\)
* VPCEndpointServiceName : 앞서 복사해둔 GWLBVPC의 VPC endpoint service name을 입력합니다.
* PrivateToGWLB : 0.0.0.0/0 \(Private Subnet이 외부로 가는 목적지에 대한 라우팅 경로 설정입니다.\)
* InstanceTyep: t3.small
* KeyPair : 미 만들어 둔 keyPair를 사용합니다.

![](.gitbook/assets/image%20%28118%29.png)

아래와 같이 VPC가 모두 정상적으로 설정되었는지 확인해 봅니다.

AWS 관리콘솔 - VPC

![](.gitbook/assets/image%20%28114%29.png)

## GWLB 구성 확인

GWLBVPC 구성을 확인해 봅니다.

1. GWLB 구성
2. GWLB Target Group 구성
3. VPC Endpoint 와 Service 확인
4. Appliance 확인

![](.gitbook/assets/image%20%28110%29.png)

### 3.GWLB 구성

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 로드밸런서 메뉴를 선택합니다. Gateway LoadBalancer 구성을 확인할 수 있습니다. ELB 유형이 "gateway"로 구성된 것을 확인 할 수 있습니다.

![](.gitbook/assets/image%20%28113%29.png)



### 4.GWLB Target Group 구성

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 대상 그룹을 선택합니다. GWLB가 로드밸런싱을 하게 되는 대상그룹\(Target Group\)을 확인 할 수 있습니다.

* 프로토콜 : GENEVE 6081 \(포트 6081의 GENGEVE 프로토콜을 사용하여 모든 IP 패킷을 수신하고 리스너 규칙에 지정된 대상 그룹에 트래픽을 전달합니다.\)
* 등록된 대상 : GWLB가 로드밸런싱을 하고 있는 Target 장비를 확인합니다.

![](.gitbook/assets/image%20%28117%29.png)

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 대상 그룹 - 상태검사 메뉴를 확인합니다.

ELB와 동일하게 대상그룹\(Target Group\)에 상태를 검사할 수 있습니다. 이 랩에서는 HTTP Path / 를 통해서 Health Check를 하도록 구성했습니다.

![](.gitbook/assets/image%20%28101%29.png)

### 5. VPC Endpoint Service 확인

Workload VPC\(VPC01,02\)들과 Private link로 연결하기 위해, GWLB VPC에 Endpoint Service를 구성하였습니다. 이를 확인해 봅니다.

AWS 관리 콘솔 - VPC - 엔드포인트 서비스를 선택합니다. 생성된 VPC Endpoint Service를 확인할 수 있습니다.

* 서비스 이름 - 예 com.amazonaws.vpce.ap-northeast-2.vpce-svc-03f01aa9fbb85beb4
* 유형 : GatewayLoadBalancer
* 가용영역 : ap-northeast-2a, ap-northeast-2b

2개 영역에 걸쳐서 GWLB에 대해 VPC Endpoint Service를 구성하고 있습니다.

![](.gitbook/assets/image%20%28126%29.png)

AWS 관리 콘솔 - VPC - 엔드포인트 서비스-엔드포인트 연결를 선택합니다.

Workload VPC \(VPC01,02\)의 각 가용영역들과 연결된 것을 확인 할 수 있습니다. 각 VPC별 2개의 가용영역을 구성하였기 때문에 VPC별 2개의 Endpoint가 연결됩니다. \(VPC 2개를 생성해서 VPC Endpoint를 각 리전별로 구성하기 때문에 이 랩에서는 4개가 보이게 됩니다.\)

![](.gitbook/assets/image%20%28125%29.png)

### 6. Appliance 확인

AWS 관리 콘솔 - EC2 - 인스턴스 메뉴를 선택하고, "appliance" 키워드로 필터링 해 봅니다. 4개의 리눅스 기반의 appliance가 설치되어 있습니다.

![](.gitbook/assets/image%20%28102%29.png)

Appliance 구성 정보를 확인해 봅니다.

AWS 관리콘솔 - Cloudformation - 스택을 선택하면, 앞서 배포했던 Cloudformation 스택들을 확인 할 수 있습니다. "GWLBVPC"를 선택합니다. 그리고 출력을 선택합니다. 값을 확인해 보면 공인 IP 주소를 확인 할 수 있습니다.

![](.gitbook/assets/image%20%28108%29.png)

앞서 사전 준비에서 생성한 Cloud9에서 Appliance로 직접 접속해 봅니다.

```text
export Appliance1={Appliance1ip address}
export Appliance2={Appliance2ip address}
export Appliance3={Appliance3ip address}
export Appliance4={Appliance4ip address}
```

아래와 같이 구성합니다.

```text
export Appliance1=3.35.55.51
export Appliance2=3.35.53.210
export Appliance3=3.35.5.188
export Appliance4=3.34.28.238
echo "export Appliance1=$Appliance1" | tee -a ~/.bash_profile
echo "export Appliance2=$Appliance2" | tee -a ~/.bash_profile
echo "export Appliance3=$Appliance3" | tee -a ~/.bash_profile
echo "export Appliance4=$Appliance4" | tee -a ~/.bash_profile
source ~/.bash_profile
## 앞서 변경했으면 적용하지 않습니다.
mv ~/environment/gwlbkey ~/environment/gwlbkey.pem
chmod 400 ./gwlbkey.pem

```

cloud9에서 아래와 같이 각 Appliance로 접속해 봅니다.

```text
ssh -i ~/environment/gwlbkey.pem ec2-user@$Appliance1
ssh -i ~/environment/gwlbkey.pem ec2-user@$Appliance2
ssh -i ~/environment/gwlbkey.pem ec2-user@$Appliance3
ssh -i ~/environment/gwlbkey.pem ec2-user@$Appliance4

```

각 Appliance에서 아래 명령을 통해 , GWLB IP와 어떻게 매핑되었는지 확인합니다.

```text
sudo iptables -L -n -v -t nat

```

AZ A에 배포된 Appliance는 다음과 같이 출력됩니다.

```text
[ec2-user@ip-10-254-11-101 ~]$ sudo iptables -L -n -v -t nat
Chain PREROUTING (policy ACCEPT 4987 packets, 298K bytes)
 pkts bytes target     prot opt in     out     source               destination         
  178 22464 DNAT       udp  --  eth0   *       10.254.11.60         10.254.11.101        to:10.254.11.60:6081

Chain INPUT (policy ACCEPT 4987 packets, 298K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 1315 packets, 102K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 1315 packets, 102K bytes)
 pkts bytes target     prot opt in     out     source               destination         
  178 22464 MASQUERADE  udp  --  *      eth0    10.254.11.60         10.254.11.60         udp dpt:6081
```

GENEVE 터널링의 GWLB IP주소는 10.254.11.60 이며, Appliance IP와 터널링 된 것을 확인 할 수 있습니다.

AZ B에 배포된 Appliance는 다음과 같이 출력됩니다.

```text
[ec2-user@ip-10-254-12-101 ~]$ sudo iptables -L -n -v -t nat
Chain PREROUTING (policy ACCEPT 5313 packets, 316K bytes)
 pkts bytes target     prot opt in     out     source               destination         
  192 23456 DNAT       udp  --  eth0   *       10.254.12.149        10.254.12.101        to:10.254.12.149:6081

Chain INPUT (policy ACCEPT 5313 packets, 316K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 1626 packets, 123K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 1626 packets, 123K bytes)
 pkts bytes target     prot opt in     out     source               destination         
  192 23456 MASQUERADE  udp  --  *      eth0    10.254.12.149        10.254.12.149        udp dpt:6081
```

GENEVE 터널링의 GWLB IP주소는 10.254.12.101 이며, Appliance IP와 터널링 된 것을 확인 할 수 있습니다.

이렇게 GWLB 에서 생성된 IP주소와 각 Appliance의 IP간에 UDP 6081 포트로 터널링되어 , 외부의 IP 주소와 내부의 IP 주소를 그대로 유지할 수 있습니다. 또한 터널링으로 인입시 5Tuple \(출발지 IP, Port, 목적지 IP, Port, 프로토콜\)의 정보를 TLV로 Encapsulation하여 분산처리할 때 사용합니다.

## Workload VPC 확인

이제 각 VPC에서 실제 구성과 트래픽을 확인해 봅니다.

1. VPC Endpoint 확인
2. GWLB Subnet Route Table 확인
3. Public Subnet Route Table 확인
4. ALB 확인
5. Private Subnet Route Table 확인
6. Ingress Routing Table 확인

![](.gitbook/assets/image%20%28120%29.png)

1. 외부 트래픽은 인터넷 게이트웨이로 접근
2. Ingress Route Table에 의해 GWLB Endpoint로 트래픽 처리
3. GWLB Subnet의 VPC Endpoint는 GWLB VPC Endpoint Service로 전달
4. GWLB로 트래픽 전달
5. AZ A,AZ B Target Group으로 LB 처리 - UDP 6081 GENEVE로 Encapsulation \(TLV Header - 5Tuple\)
6. Appliance으로 트래픽 인입
7. Appliance에서 트래픽 반환
8. GWLB로 트래픽 반환
9. Decap 해서 다시 VPC Endpoint Service로 전달
10. GWLB Subnet VPC Endpoint로 전달
11. Public Subnet ALB로 전달
12. ALB \(Internet Facing\) 은 Private Target Group에 포함된 인스턴스로 전달. \(Private EC2 인스턴스\)
13. Return되는 트래픽은 ALB를 거쳐서, GWLB EP-GWLB-Internet Gateway로 다시 전달

{% hint style="info" %}
VPC01,02 NAT Gateway는 Private EC2 인스턴스들의 PAT로 동작하며, Private EC2 인스턴스들이 내부에서 외부로 Initiate 되는 트래픽들을 처리합니다. \(Patch, 패키지 다운드로 등...\)
{% endhint %}

![](.gitbook/assets/image%20%28123%29.png)

### 7. Ingress Route Table 확인

AWS 관리콘솔 - VPC - 라우팅 테이블을 선택하고 VPC01,02-IGW-Ingress-RT 이름의 라우팅 테이블을 확인해 봅니다. Ingress Routing Table에 대한 구성을 확인 할 수 있습니다. GWLB Subnet,Public Subnet으로 인입 되는 트래픽을 특정 경로로 보내는 역할을 합니다. 여기에서는 GWLB VPC Endpoint로 구성하도록 되어 있습니다.

![](.gitbook/assets/image%20%28119%29.png)

{% hint style="info" %}
Ingress Routing에서 Private Subnet에 대한 라우팅 설정은 왜 없을까요?

외부에서 Ingress Routing 접근은 Internet Gateway에서 공인 IP로 설정된 트래픽만 설정할 수 있습니다. 이 랩에서는 외부에서 ALB로 접근할 것입니다. ALB는 Public Subnet에 할당 되어 있고, 공인 IP를 받을 수 있도록 설정되어 있습니다. Private Subnet에 대한 라우팅 설정은 Ingress Routing에서 설정해도 동작하지 않습니다.
{% endhint %}

### 7.VPC Endpoint 확인

AWS 관리 콘솔 - VPC - Endpoint를 선택하여 실제 구성된 VPC Endpoint를 확인해 봅니다. 2개의 VPC에 2개씩 구성된 AZ를 위해 총 4개의 Endpoint가 구성되어 있습니다. \(VPC Endpoint는 AZ Subnet당 연결됩니다.\)

![](.gitbook/assets/image%20%28121%29.png)

#### 

### 8. GWLB Subnet Route Table 확인

AWS 관리콘솔 - VPC - 라우팅 테이블을 선택하고 VPC01,02-Private-Subnet-A,B-RT 이름의 라우팅 테이블을 확인해 봅니다. Return되는 트래픽의 경로는 GWLB VPC Endpoint로 설정되어 있습니다.



### 9. Public Subnet Route Table 확인



### 10. ALB 확인



### 11. Private Subnet Route Table 확인

### 



