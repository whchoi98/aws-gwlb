---
description: 'update : 2021-03-31'
---

# GWLB Design 1

## Cloudformation기반 VPC 배포

### 1.VPC yaml 파일 다운로드 

아래 github에서 VPC yaml 파일을 다운로드 합니다.

```text

```

### 2.AWS 관리콘솔에서 VPC 배포

AWS 관리콘솔에서 Cloudformation을 선택합니다.

![](.gitbook/assets/image%20%283%29.png)

앞서 다운로드 해둔 yaml 파일 중에서, 아래 그림과 같이 GWLBVPC.yml 파일을 선택합니다.

![](.gitbook/assets/image%20%285%29.png)

스택 세부 정보 지정에서 , 스택이름과 VPC Parameters를 지정합니다. 대부분 기본값을 사용하면 됩니다.

* 스택이름 : GWLBVPC
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.254.0.0/16
* PublicSubnetABlock: 10.254.11.0/24
* PublicSubnetBBlock: 10.254.12.0/24
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.

![](.gitbook/assets/image%20%284%29.png)

다음 단계를 계속 진행하고, 아래와 같이 "AWS CloudFormation에서 IAM 리소스를 생성할 수 있음을 승인합니다."를 선택하고, 스택을 생성합니다.

![](.gitbook/assets/image%20%281%29.png)

3~4분 후에 GWLBVPC가 완성됩니다.

AWS 관리콘솔 - VPC - 가상 프라이빗 클라우드 - 엔드포인트 서비스 를 선택합니다. Cloudformation을 통해서 VPC Endpoint 서비스가 이미 생성되어 있습니다. 이것을 선택하고 세부 정보를 확인합니다.

서비스 이름을 복사해 둡니다. 뒤에서 생성할 VPC들의 Cloudformation에서 사용할 것입니다.

![](.gitbook/assets/image%20%282%29.png)

VPC01,02,03 3개의 VPC를 Cloudformation에서 앞서 과정과 동일하게 생성합니다. 다운로드 받은 Yaml 파일들 중에 VPC01.yml, VPC02,yml, VPC03.yml을 차례로 선택해서 생성합니다.

![](.gitbook/assets/image%20%287%29.png)

스택 이름을 생성하고, GWLBVPC의 VPC Endpoint 서비스 이름을 "VPCEndpointServiceName" 에 입력합니다. 또한 나머지 파라미터들도 입력합니다. 대부분 기본값을 사용합니다.

* 스택이름 : VPC01, VPC02, VPC03
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.1.0.0/16 \(VPC01\), 10.2.0.0/16 \(VPC02\), 10.3.0.0/16 \(VPC03\)
* PublicSubnetABlock: 10.1.11.0/24 \(VPC01\), 10.2.11.0/24 \(VPC02\), 10.3.11.0/24 \(VPC03\)
* PublicSubnetBBlock: 10.1.12.0/24 \(VPC01\), 10.2.12.0/24 \(VPC02\), 10.3.12.0/24 \(VPC03\)
* VPCEndpointServiceName : 앞서 복사해둔 GWLBVPC의 VPC endpoint service name을 입력합니다.
* PrivateToGWLB : 0.0.0.0/0 \(Private Subnet이 외부로 가는 목적지에 대한 라우팅 경로 설정입니다.\)
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다. 

![](.gitbook/assets/image%20%2815%29.png)

아래와 같이 VPC가 모두 정상적으로 설정되었는지 확인해 봅니다.

AWS 관리콘솔 - VPC

![](.gitbook/assets/image%20%288%29.png)



