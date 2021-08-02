# GWLB Desing 4

## 목표 구성 개요

2개의 각 워크로드 VPC \(VPC01,02\)은 Account내에 구성된 GWLB 기반의 보안 VPC를 통해서 내, 외부 트래픽을 처리하는 구성입니다. GWLB 기반의 보안 VPC는 2개의 AZ에 4개의 가상 Appliance가 로드밸런싱을 통해 처리 됩니다.

이러한 구성은 VPC Endpoint를 특정 VPC에 구성하고, TransitGateway를 통해 GWLB에 VPC Endpoint Service를 연결하는 중앙집중 구조입니다.

GWLB Design2와 다른 점은 ALB\(Application Load Balancer\)를 GWLB와 연계하는 VPC에 배치해서, 내부의 VPC01,02의 서비스들이 외부에 제공하도록 한다는 점입니다.

아래 그림은 목표 구성도 입니다.

**🎬 아래 동영상 링크에서 구성방법을 확인 할 수 있습니다.**

![](.gitbook/assets/image%20%28151%29.png)

## Cloudformation기반 VPC 배포

### 1.VPC yaml 파일 다운로드

Cloud9 콘솔에서 아래 github로 부터 VPC yaml 파일을 다운로드 합니다. \(앞서 다운로드 하였으면 생략합니다.\)

```text
git clone https://github.com/whchoi98/gwlb.git

```

Cloud9에서 로컬로 파일을 다운로드 받습니다.

![](.gitbook/assets/image%20%28150%29.png)

AWS 관리콘솔에서 Cloudformation을 선택합니다.

![](.gitbook/assets/image%20%28148%29.png)

아래와 같은 순서로 Cloudformation에서 Yaml파일을 배포합니다.

1. GWLBVPC.yml
2. N2SVPC.yml
3. VPC01.yml, VPC02.yml
4. GWLBTGW.yml

{% hint style="info" %}
계정에서 VPC 기본 할당량은 Default VPC 포함 5개입니다. 이 랩에서는 VPC03 은 생성하지 않습니다.
{% endhint %}

### 2.GWLB VPC 배포

앞서 다운로드 해둔 yaml 파일 중에서, 아래 그림과 같이 GWLBVPC.yml 파일을 선택합니다.

![](.gitbook/assets/image%20%28161%29.png)

스택 세부 정보 지정에서 , 스택이름과 VPC Parameters를 지정합니다. 대부분 기본값을 사용하면 됩니다.

* 스택이름 : GWLBVPC
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.254.0.0/16
* PublicSubnetABlock: 10.254.11.0/24
* PublicSubnetBBlock: 10.254.12.0/24
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다. \(예. gwlbkey\)

![](.gitbook/assets/image%20%28155%29.png)

다음 단계를 계속 진행하고, 아래와 같이 **`"AWS CloudFormation에서 IAM 리소스를 생성할 수 있음을 승인합니다."`**를 선택하고, **`스택을 생성`**합니다.

![](.gitbook/assets/image%20%28158%29.png)

3~4분 후에 GWLBVPC가 완성됩니다.

**`AWS 관리콘솔 - VPC - 가상 프라이빗 클라우드 - 엔드포인트 서비스`** 를 선택합니다. Cloudformation을 통해서 VPC Endpoint 서비스가 이미 생성되어 있습니다. 이것을 선택하고 \*\*`세부 정보`\*\*를 확인합니다.

서비스 이름을 복사해 둡니다. 뒤에서 생성할 VPC들의 Cloudformation에서 사용할 것입니다.

![](.gitbook/assets/image%20%28147%29.png)

### 3.N2SVPC 배포

외부 인터넷으로 통신하는 North-South 트래픽 처리를 하는 VPC를 생성합니다.

N2SVPC를 Cloudformation에서 앞서 과정과 동일하게 생성합니다. 다운로드 받은 Yaml 파일들 중에 N2SVPC 선택해서 생성합니다.스택 이름을 생성하고, GWLBVPC의 VPC Endpoint 서비스 이름을 **`"VPCEndpointServiceName"`** 에 입력합니다. 또한 나머지 파라미터들도 입력합니다. 대부분 기본값을 사용합니다.

![](.gitbook/assets/image%20%28156%29.png)



* 스택이름 : N2SVPC
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.11.0.0
* GWLBeSubnetABlock:10.11.1.0/24
* GWLBeSubnetBBlock:10.11.2.0/24
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
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.\(예. gwlbkey\)

### 4.VPC01,02 배포

**나머지 VPC01,VPC02,VPC03 의 Cloudformation Yaml 파일을 업로드 합니다.**

{% hint style="info" %}
VPC는 계정당 기본 5개가 할당되어 있습니다. 1개는 Default VPC로 사용 중이고, 4개를 사용 가능하므로 일반 계정에서는 GWLBVPC, N2SVPC, VPC01,VPC02 까지만 생성 가능합니다.  
 5개 모두를 사용하시려면, Default VPC를 삭제하시기 바랍니다. Default VPC는 삭제 후 다시 생성이 가능합니다.
{% endhint %}

![](.gitbook/assets/image%20%28153%29.png)

* 스택이름 : VPC01,VPC02
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.1.0.0 \(VPC01\), 10.2.0.0 \(VPC02\)
* PrivateSubnetABlock:10.1.21.0/24 \(VPC01\), 10.2.22.0/24\(VPC02\)
* PrivateSubnetBBlock:10.1.22.0/24 \(VPC01\), 10.2.22.0/24\(VPC02\)
* TGWSubnetABlock:10.1.251.0/24 \(VPC01\), 10.2.251.0/24 \(VPC02\)
* TGWSubnetBBlock:10.1.252.0/24 \(VPC01\), 10.2.252.0/24 \(VPC02\)
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.\(예. gwlbkey\)

N2SVPC, VPC01,02,03 을 연결할 TGW를 생성합니다. N2STGW는 TGW Routing Table과 각 VPC들이 Route Table을 자동으로 구성해 줍니다.

* Stack Name : GWLBTGW
* DefaultRouteBlock: 0.0.0.0/0
* VPC01CIDRBlock: 10.1.0.0/16
* VPC02CIDRBlock: 10.2.0.0/16

아래와 같이 VPC가 모두 정상적으로 설정되었는지 확인해 봅니다.

**`AWS 관리 콘솔 - VPC 대시 보드 - VPC`**

![](.gitbook/assets/image%20%28149%29.png)

**`AWS 관리 콘솔 - VPC 대시 보드 - 서브넷`**

![](.gitbook/assets/image%20%28160%29.png)

#### 

### 5. TransitGateway 배포

N2SVPC, VPC01,VPC02을 연결하기 위한 TransitGateway를 배포합니다. 앞서 git을 통해 다운 받은 파일 중 GWLBTGW.yml 파일을 Cloudformation을 통해서 배포합니다.

`Default Route Table`과 **`VPC01, VPC02 CIDR`** 주소를 입력합니다. \(기본 값으로 설정되어 있습니다.\)

![](.gitbook/assets/image%20%28152%29.png)

### 6. 라우팅 테이블 확인

TransitGateway 구성과 RouteTable을 아래에서 확인합니다.

![](.gitbook/assets/image%20%28154%29.png)

