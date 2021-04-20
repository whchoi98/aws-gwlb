---
description: 'Update : 2021-04-12 / 1h'
---

# GWLB Design 2

## 목표 구성 개요

3개의 각 워크로드 VPC \(VPC01,02,03\)은 Account내에 구성된 GWLB 기반의 보안 VPC를 통해서 내, 외부 트래픽을 처리하는 구성입니다. GWLB 기반의 보안 VPC는 2개의 AZ에 4개의 가상 Appliance가 로드밸런싱을 통해 처리 됩니다.

아래 그림은 목표 구성도 입니다.

![](.gitbook/assets/image%20%2858%29.png)

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

![](.gitbook/assets/image%20%2859%29.png)

AWS 관리 콘솔 - VPC 대시 보드 - 서브

![](.gitbook/assets/image%20%2860%29.png)



### 5.TransitGateway 배 



## GWLB 구성 확인

GWLBVPC 구성을 확인해 봅니다.

1. GWLB 구성
2. GWLB Target Group 구성
3. VPC Endpoint 와 Service 확인
4. Appliance 확인 

![](.gitbook/assets/image%20%2845%29.png)

### 3.GWLB 구성 

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 로드밸런서 메뉴를 선택합니다. Gateway LoadBalancer 구성을 확인할 수 있습니다. ELB 유형이 "gateway"로 구성된 것을 확인 할 수 있습니다.

![](.gitbook/assets/image%20%2839%29.png)

### 4.GWLB Target Group 구성 

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 대상 그룹을 선택합니다. GWLB가 로드밸런싱을 하게 되는 대상그룹\(Target Group\)을 확인 할 수 있습니다.

*  프로토콜 : GENEVE 6081 \(포트 6081의 GENGEVE 프로토콜을 사용하여 모든 IP 패킷을 수신하고 리스너 규칙에 지정된 대상 그룹에 트래픽을 전달합니다.\)
* 등록된 대상 : GWLB가 로드밸런싱을 하고 있는 Target 장비를 확인합니다.

![](.gitbook/assets/image%20%2855%29.png)

AWS 관리 콘솔 - EC2 - 로드밸런싱 - 대상 그룹 - 상태검사 메뉴를 확인합니다.

ELB와 동일하게 대상그룹\(Target Group\)에 상태를 검사할 수 있습니다. 이 랩에서는 HTTP  Path / 를 통해서 Health Check를 하도록 구성했습니다.

![](.gitbook/assets/image%20%2853%29.png)

### 5. VPC Endpoint Service 확인

Workload VPC\(VPC01,02,03\)들과 Private link로 연결하기 위해, GWLB VPC에 Endpoint Service를 구성하였습니다. 이를 확인해 봅니다.

AWS 관리 콘솔 - VPC - 엔드포인트 서비스를 선택합니다. 생성된 VPC Endpoint Service를 확인할 수 있습니다.

* 서비스 이름 - 예 com.amazonaws.vpce.ap-northeast-2.vpce-svc-03f01aa9fbb85beb4
* 유형 : GatewayLoadBalancer
* 가용영역 : ap-northeast-2a, ap-northeast-2b

2개 영역에 걸쳐서 GWLB에 대해 VPC Endpoint Service를 구성하고 있습니다.

![](.gitbook/assets/image%20%2842%29.png)

AWS 관리 콘솔 - VPC - 엔드포인트 서비스-엔드포인트 연결를 선택합니다.

Workload VPC \(VPC01,02,03\)의 각 가용영역들과 연결된 것을 확인 할 수 있습니다. 각 VPC별 2개의 가용영역을 구성하였기 때문에 VPC별 2개의 Endpoint가 연결됩니다.

![](.gitbook/assets/image%20%2846%29.png)

### 6. Appliance 확인 

AWS 관리 콘솔 - EC2 - 인스턴스 메뉴를 선택하고, "appliance" 키워드로 필터링 해 봅니다. 4개의 리눅스 기반의 appliance가 설치되어 있습니다.

![](.gitbook/assets/image%20%2854%29.png)

Appliance 구성 정보를 확인해 봅니다.

AWS 관리콘솔 - Cloudformation - 스택을 선택하면, 앞서 배포했던 Cloudformation 스택들을 확인 할 수 있습니다. "GWLBVPC"를 선택합니다. 그리고 출력을 선택합니다. 값을 확인해 보면 공인 IP 주소를 확인 할 수 있습니다.

![](.gitbook/assets/image%20%2850%29.png)

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

GENEVE 터널링의 GWLB IP주소는 10.254.11.60  이며, Appliance IP와 터널링 된 것을 확인 할 수 있습니다.

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

GENEVE 터널링의 GWLB IP주소는 10.254.12.101  이며, Appliance IP와 터널링 된 것을 확인 할 수 있습니다.

이렇게 GWLB 에서 생성된 IP주소와 각 Appliance의 IP간에 UDP 6081 포트로 터널링되어 , 외부의 IP 주소와 내부의 IP 주소를 그대로 유지할 수 있습니다. 또한 터널링으로 인입시 5Tuple \(출발지 IP, Port, 목적지 IP, Port, 프로토콜\)의 정보를 TLV로 Encapsulation하여 분산처리할 때 사용합니다.





## 외부 연결용 VPC 확인

이제 외부 연결 VPC에서 실제 구성과 트래픽을 확인해 봅니다.

1. VPC Endpoint 확인
2. Private Subnet Route Table 확인
3. Ingress Routing Table 확인



아래 흐름과 같이 트래픽이 처리됩니다.

1. 외부 트래픽은 인터넷 게이트웨이로 접근
2. Ingress Route Table에 의해 GWLB Endpoint로 트래픽 처리
3. Public Subnet의 VPC Endpoint는 GWLB VPC Endpoint Service로 전달
4. GWLB로 트래픽 전달
5. AZ A,AZ B Target Group으로 LB 처리 - UDP 6081 GENEVE로 Encapsulation \(TLV Header - 5Tuple\)
6. Appliance에서 트래픽 처리 후 다시 Return
7. Decap 해서 다시 VPC Endpoint Service로 전달
8. Public Subnet VPC Endpoint로 전달
9. Private Subnet 인스턴스로 전달 
10. Return되는 트래픽은 Private Subnet의 Route Table에 의해 VPC Endpoint로 다시 전



### 7.VPC Endpoint 확인

AWS 관리 콘솔 - VPC - Endpoint를 선택하여 실제 구성된 VPC Endpoint를 확인해 봅니다. 3개의 VPC에 2개씩 구성된 AZ를 위해 총 6개의 Endpoint가 구성되어 있습니다. \(VPC Endpoint는 AZ Subnet당 연결됩니다.\)



### 8. Private Subnet Route Table 확인

AWS 관리콘솔 - VPC - 라우팅 테이블을 선택하고 VPC01,02,03-Private-Subnet-A,B-RT 이름의 라우팅 테이블을 확인해 봅니다. Return되는 트래픽의 경로는 GWLB VPC Endpoint로 설정되어 있습니다.



### 9. Ingress Routing Table 확인

AWS 관리콘솔 - VPC - 라우팅 테이블을 선택하고 VPC01,02,03-IGW-Ingress-RT 이름의 라우팅 테이블을 확인해 봅니다.  Ingress Routing Table에 대한 구성을 확인 할 수 있습니다. VPC로 인입 되는 트래픽을 특정 경로로 보내는 역할을 합니다. 여기에서는 GWLB VPC Endpoint로 구성하도록 되어 있습니다.

  


