---
description: 'Update : 2021-04-12 / 1h'
---

# GWLB Design 2

## 목표 구성 개요

3개의 각 워크로드 VPC \(VPC01,02,03\)은 Account내에 구성된 GWLB 기반의 보안 VPC를 통해서 내, 외부 트래픽을 처리하는 구성입니다. GWLB 기반의 보안 VPC는 2개의 AZ에 4개의 가상 Appliance가 로드밸런싱을 통해 처리 됩니다.

아래 그림은 목표 구성도 입니다.

![](.gitbook/assets/image%20%2864%29.png)

## Cloudformation기반 VPC 배포

### 1.VPC yaml 파일 다운로드

아래 github에서 VPC yaml 파일을 다운로드 합니다.

```text

```

AWS 관리콘솔에서 Cloudformation을 선택합니다.

![](.gitbook/assets/image%20%2847%29.png)

아래와 같은 순서로 Cloudformation에서 Yaml파일을 배포합니다.

1. GWLBVPC.yml
2. N2SVPC.yml
3. VPC01.yml, VPC02.yml
4. GWLBTGW.yml

### 2.GWLB VPC 배포

앞서 다운로드 해둔 yaml 파일 중에서, 아래 그림과 같이 GWLBVPC.yml 파일을 선택합니다.

![](.gitbook/assets/image%20%2851%29.png)

스택 세부 정보 지정에서 , 스택이름과 VPC Parameters를 지정합니다. 대부분 기본값을 사용하면 됩니다.

* 스택이름 : GWLBVPC
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.254.0.0/16
* PublicSubnetABlock: 10.254.11.0/24
* PublicSubnetBBlock: 10.254.12.0/24
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.

![](.gitbook/assets/image%20%2843%29.png)

다음 단계를 계속 진행하고, 아래와 같이 "AWS CloudFormation에서 IAM 리소스를 생성할 수 있음을 승인합니다."를 선택하고, 스택을 생성합니다.

![](.gitbook/assets/image%20%2852%29.png)

3~4분 후에 GWLBVPC가 완성됩니다.

AWS 관리콘솔 - VPC - 가상 프라이빗 클라우드 - 엔드포인트 서비스 를 선택합니다. Cloudformation을 통해서 VPC Endpoint 서비스가 이미 생성되어 있습니다. 이것을 선택하고 세부 정보를 확인합니다.

서비스 이름을 복사해 둡니다. 뒤에서 생성할 VPC들의 Cloudformation에서 사용할 것입니다.

![](.gitbook/assets/image%20%2837%29.png)

### 3.N2SVPC 배포 

외부 인터넷으로 통신하는 North-South 트래픽 처리를 하는 VPC를 생성합니다.

N2SVPC를 Cloudformation에서 앞서 과정과 동일하게 생성합니다. 다운로드 받은 Yaml 파일들 중에 N2SVPC 선택해서 생성합니다.스택 이름을 생성하고, GWLBVPC의 VPC Endpoint 서비스 이름을 "VPCEndpointServiceName" 에 입력합니다. 또한 나머지 파라미터들도 입력합니다. 대부분 기본값을 사용합니다.

![](.gitbook/assets/image%20%2849%29.png)

![](.gitbook/assets/image%20%2848%29.png)

* 스택이름 : N2SVPC
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.11.0.0
* PublicSubnetABlock: 10.11.11.0/24
* PublicSubnetBBlock: 10.11.12.0/24
* PrivateSubnetABlock:10.11.21.0/24
* PrivateSubnetBBlock:10.11.22.0/24
* TGWSubnetABlock:10.11.251.0/24
* TGWSubnetBBlock:10.11.252.0/24
* DefaultRouteBlock: 0.0.0.0/0 \(Default Route Table 주소를 선언합니다.\)
* VPC1CIDRBlock : 10.1.0.0/16 \(VPC1의 CIDR Block 주소를 선언합니다.\)
* VPC2CIDRBlock: 10.2.0.0/16 \(VPC2의 CIDR Block 주소를 선언합니다.\)
* VPCEndpointServiceName : 앞서 복사해둔 GWLBVPC의 VPC endpoint service name을 입력합니다.
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.

### 4.VPC01,02 배포  

#### 나머지 VPC01,VPC02,VPC03 의 Cloudformation Yaml 파일을 업로드 합니다.

{% hint style="warning" %}
VPC는 계정당 기본 5개가 할당되어 있습니다. 1개는 Default VPC로 사용 중이고, 4개를 사용 가능하므로 일반 계정에서는 GWLBVPC, N2SVPC, VPC01,VPC02 까지만 생성 가능합니다
{% endhint %}

![](.gitbook/assets/image%20%2841%29.png)

* 스택이름 : VPC01,VPC02
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.1.0.0 \(VPC01\), 10.2.0.0 \(VPC02\)
* PrivateSubnetABlock:10.1.21.0/24 \(VPC01\), 10.2.22.0/24\(VPC02\)
* PrivateSubnetBBlock:10.1.22.0/24 \(VPC01\), 10.2.22.0/24\(VPC02\)
* TGWSubnetABlock:10.1.251.0/24 \(VPC01\), 10.2.251.0/24 \(VPC02\)
* TGWSubnetBBlock:10.1.252.0/24 \(VPC01\), 10.2.252.0/24 \(VPC02\)
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.

N2SVPC, VPC01,02,03 을 연결할 TGW를 생성합니다.  N2STGW는 TGW Routing Table과 각 VPC들이 Route Table을 자동으로 구성해 줍니다.

* Stack Name : GWLBTGW
* DefaultRouteBlock: 0.0.0.0/0
* VPC01CIDRBlock: 10.1.0.0/16
* VPC02CIDRBlock: 10.2.0.0/16

![](.gitbook/assets/image%20%2840%29.png)

아래와 같이 VPC가 모두 정상적으로 설정되었는지 확인해 봅니다.

AWS 관리 콘솔 - VPC 대시 보드 - VPC

![](.gitbook/assets/image%20%2871%29.png)

AWS 관리 콘솔 - VPC 대시 보드 - 서브

![](.gitbook/assets/image%20%2876%29.png)

### 5. TransitGateway 배포 

N2SVPC, VPC01,VPC02을 연결하기 위한 TransitGateway를 배포합니다. 앞서 git을 통해 다운 받은 파일 중 GWLBTGW.yml 파일을 Cloudformation을 통해서 배포합니다.

![](.gitbook/assets/image%20%2877%29.png)

Default Route Table과 VPC01, VPC02 CIDR 주소를 입력합니다. \(기본 값으로 설정되어 있습니다.\)

![](.gitbook/assets/image%20%2861%29.png)

### 6. 라우팅 테이블 확인  

TransitGateway 구성과 RouteTable을 아래에서 확인합니다. 

![](.gitbook/assets/image%20%2862%29.png)

AWS 관리 콘솔 - VPC 대시보드 - TransitGateway 에서 TransitGateway가 정상적으로 구성되었는지 확인합니다.

![](.gitbook/assets/image%20%2873%29.png)

AWS 관리 콘솔 - VPC 대시보드 - TransitGateway- TransitGateway 연결\(Attachment\) 에서 TransitGateway와 VPC가 정상적으로 연결되었는지 확인합니다.

![](.gitbook/assets/image%20%2859%29.png)

AWS 관리콘솔 - VPC 대시보드 -TransitGateway-TransitGateway 라우팅 테이블-Route 에서 "GWLBTGW-RT-North-To-South", "GWLBTGW-RT-East-To-West" 라우팅 테이블을 확인합니다.

* GWLBTGW-RT-North-To-South : VPC01,VPC02 에서 인터넷으로 향하는 트래픽
* GWLBTGW-RT-East-To-West: VPC01,VPC02 상호간에 트래

![](.gitbook/assets/image%20%2883%29.png)

![](.gitbook/assets/image%20%2870%29.png)

AWS 관리 콘솔 -VPC 대시보드 - 가상 프라이빗 클라우드 - 라우팅테이블에서  각 Private-Subnet-A,B-RT 라우팅 테이블을 확인합니다. 

* VPC01,02-Private-Subnet-A,B-RT  : 0.0.0.0/0 - tgw 

![](.gitbook/assets/image%20%2869%29.png)

## GWLB 구성 확인

GWLBVPC 구성을 확인해 봅니다.

1. GWLB 구성
2. GWLB Target Group 구성
3. VPC Endpoint 와 Service 확인
4. Appliance 확인 

![](.gitbook/assets/image%20%2845%29.png)

### 7.GWLB 구성 

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 로드밸런서 메뉴를 선택합니다. Gateway LoadBalancer 구성을 확인할 수 있습니다. ELB 유형이 "gateway"로 구성된 것을 확인 할 수 있습니다.

![](.gitbook/assets/image%20%2880%29.png)

### 8.GWLB Target Group 구성 

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 대상 그룹을 선택합니다. GWLB가 로드밸런싱을 하게 되는 대상그룹\(Target Group\)을 확인 할 수 있습니다.

*  프로토콜 : GENEVE 6081 \(포트 6081의 GENGEVE 프로토콜을 사용하여 모든 IP 패킷을 수신하고 리스너 규칙에 지정된 대상 그룹에 트래픽을 전달합니다.\)
* 등록된 대상 : GWLB가 로드밸런싱을 하고 있는 Target 장비를 확인합니다.

![](.gitbook/assets/image%20%2868%29.png)

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 대상 그룹 - 상태검사 메뉴를 확인합니다.

ELB와 동일하게 대상그룹\(Target Group\)에 상태를 검사할 수 있습니다. 이 랩에서는 HTTP  Path / 를 통해서 Health Check를 하도록 구성했습니다.

![](.gitbook/assets/image%20%2860%29.png)

### 9. VPC Endpoint Service 확인

N2SVPC Private link로 연결하기 위해, GWLB VPC에 Endpoint Service를 구성하였습니다. 이를 확인해 봅니다.

AWS 관리 콘솔 - VPC - 엔드포인트 서비스를 선택합니다. 생성된 VPC Endpoint Service를 확인할 수 있습니다.

* 서비스 이름 - 예 com.amazonaws.vpce.ap-northeast-2.vpce-svc-082d152b9180f8ad0
* 유형 : GatewayLoadBalancer
* 가용영역 : ap-northeast-2a, ap-northeast-2b

2개 영역에 걸쳐서 GWLB에 대해 VPC Endpoint Service를 구성하고 있습니다.

![](.gitbook/assets/image%20%2875%29.png)

AWS 관리 콘솔 - VPC - 엔드포인트 서비스-엔드포인트 연결를 선택합니다.

N2SVPC의 각 가용영역들과 연결된 것을 확인 할 수 있습니다. VPC별 2개의 가용영역의 Private Subnet에 배치된 VPC Endpoint에 연결된 것을 확인 합니다.

![](.gitbook/assets/image%20%2858%29.png)

### 10. Appliance 확인 

AWS 관리 콘솔 - EC2 - 인스턴스 메뉴를 선택하고, "appliance" 키워드로 필터링 해 봅니다. 4개의 리눅스 기반의 appliance가 설치되어 있습니다.

![](.gitbook/assets/image%20%2867%29.png)

Appliance 구성 정보를 확인해 봅니다.

AWS 관리콘솔 - Cloudformation - 스택을 선택하면, 앞서 배포했던 Cloudformation 스택들을 확인 할 수 있습니다. "GWLBVPC"를 선택합니다. 그리고 출력을 선택합니다. 값을 확인해 보면 공인 IP 주소를 확인 할 수 있습니다.

![](.gitbook/assets/image%20%2878%29.png)

앞서 사전 준비에서 생성한 Cloud9에서 Appliance로 직접 접속해 봅니다.

```text
export Appliance1={Appliance1ip address}
export Appliance2={Appliance2ip address}
export Appliance3={Appliance3ip address}
export Appliance4={Appliance4ip address}
```

아래와 같이 구성합니다.

```text
export Appliance1=3.36.108.211
export Appliance2=52.79.219.13
export Appliance3=13.125.201.96
export Appliance4=15.164.176.82
echo "export Appliance1=$Appliance1" | tee -a ~/.bash_profile
echo "export Appliance2=$Appliance2" | tee -a ~/.bash_profile
echo "export Appliance3=$Appliance3" | tee -a ~/.bash_profile
echo "export Appliance4=$Appliance4" | tee -a ~/.bash_profile
source ~/.bash_profile

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
ssh -i ~/environment/gwlbkey.pem ec2-user@$Appliance1
sudo iptables -L -n -v -t nat
```

AZ A에 배포된 Appliance는 다음과 같이 출력됩니다.

```text
[ec2-user@ip-10-254-11-101 ~]$ sudo iptables -L -n -v -t nat
Chain PREROUTING (policy ACCEPT 3417 packets, 204K bytes)
 pkts bytes target     prot opt in     out     source               destination         
  351 45728 DNAT       udp  --  eth0   *       10.254.11.107        10.254.11.101        to:10.254.11.107:6081

Chain INPUT (policy ACCEPT 3417 packets, 204K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 981 packets, 75316 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 981 packets, 75316 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  351 45728 MASQUERADE  udp  --  *      eth0    10.254.11.107        10.254.11.107        udp dpt:6081
```

GENEVE 터널링의 GWLB IP주소는 10.254.11.60  이며, Appliance IP와 터널링 된 것을 확인 할 수 있습니다.

AZ B에 배포된 Appliance는 다음과 같이 출력됩니다.

```text
ssh -i ~/environment/gwlbkey.pem ec2-user@$Appliance3
sudo iptables -L -n -v -t nat

```

```text
[ec2-user@ip-10-254-12-101 ~]$ sudo iptables -L -n -v -t nat
Chain PREROUTING (policy ACCEPT 3765 packets, 225K bytes)
 pkts bytes target     prot opt in     out     source               destination         
  353 45872 DNAT       udp  --  eth0   *       10.254.12.28         10.254.12.101        to:10.254.12.28:6081

Chain INPUT (policy ACCEPT 3765 packets, 225K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 1693 packets, 136K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain POSTROUTING (policy ACCEPT 1693 packets, 136K bytes)
 pkts bytes target     prot opt in     out     source               destination         
  353 45872 MASQUERADE  udp  --  *      eth0    10.254.12.28         10.254.12.28         udp dpt:6081
```

GENEVE 터널링의 GWLB IP주소는 10.254.12.101  이며, Appliance IP와 터널링 된 것을 확인 할 수 있습니다.

이렇게 GWLB 에서 생성된 IP주소와 각 Appliance의 IP간에 UDP 6081 포트로 터널링되어 , 외부의 IP 주소와 내부의 IP 주소를 그대로 유지할 수 있습니다. 또한 터널링으로 인입시 5Tuple \(출발지 IP, Port, 목적지 IP, Port, 프로토콜\)의 정보를 TLV로 Encapsulation하여 분산처리할 때 사용합니다.

## 외부 연결용 VPC 확인

이제 외부 연결 VPC에서 실제 구성과 트래픽을 확인해 봅니다.

1. VPC Endpoint 확인
2. Private Subnet Route Table 확인
3. Ingress Routing Table 확인



아래 흐름과 같이 트래픽이 처리됩니다.

![](.gitbook/assets/image%20%2872%29.png)

1. VPC1,2 인스턴스는 외부로 향하기 위해 TransitGateway로 접근
2. VPC 1,2 Private Subnet Route Table을 참조해서, Transit Gateway로 전
3. TransitGateway Routing Table을 참조해서, N2SVPC Attachment로 전
4. N2SVPC 로 인입된 트래픽은 N2SVPC TGW Routing Table에 의해 VPC Endpoint로 전달.
5. VPC Endpoint는 GWLBVPC Private Link의 VPC Endpoint Service로 전달하고, VPC EndpointService는 GWLB로 트래픽 전달.
6. AZ A,AZ B Target Group으로 LB 처리 - UDP 6081 GENEVE로 Encapsulation \(TLV Header - 5Tuple\)
7. Appliance에서 트래픽 처리 후 다시 Return
8. Decap 해서 다시 VPC Endpoint Service로 전달하고, N2SVPC VPC Endpoint로 전달
9. Private Subnet Route Table에서 Public Subnet의 NAT Gateway로 트래픽 전달하고, 외부로 전송.

### 11.VPC Endpoint 확인

AWS 관리 콘솔 - VPC - Endpoint를 선택하여 실제 구성된 VPC Endpoint를 확인해 봅니다. N2SVPC에 2개씩 구성된 AZ를 위해 2개의 Endpoint가 구성되어 있습니다. \(VPC Endpoint는 AZ Subnet당 연결됩니다.\)

![](.gitbook/assets/image%20%2866%29.png)

### 12. N2S VPC Route Table 확인

AWS 관리콘솔 - VPC - 라우팅 테이블을 선택하고 각 라우팅 테이블을 확인해 봅니다. 

![](.gitbook/assets/image%20%2863%29.png)

AWS 관리 콘솔 - VPC 대시보드 - 라우팅 테이블 - N2SVPC TGW Routing Table 확인 

![](.gitbook/assets/image%20%2882%29.png)

AWS 관리 콘솔 - VPC 대시보드 - 라우팅 테이블 - N2SVPC Private Routing Table 확인

![](.gitbook/assets/image%20%2874%29.png)

AWS 관리 콘솔 - VPC 대시보드 - 라우팅 테이블 - N2SVPC Public Routing Table 확인

![](.gitbook/assets/image%20%2881%29.png)

### 13. Ingress Routing Table 확인

AWS 관리콘솔 - VPC - 라우팅 테이블을 선택하고 VPC01,02,03-IGW-Ingress-RT 이름의 라우팅 테이블을 확인해 봅니다.  Ingress Routing Table에 대한 구성을 확인 할 수 있습니다. VPC로 인입 되는 트래픽을 특정 경로로 보내는 역할을 합니다. 여기에서는 GWLB VPC Endpoint로 구성하도록 되어 있습니다.

  


