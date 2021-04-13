# GWLB Design 1

## Cloudformation기반 VPC 배포

### 1.VPC yaml 파일 다운로드 

아래 github에서 VPC yaml 파일을 다운로드 합니다.

```text

```

2.AWS 관리콘솔에서 VPC 배포

AWS 관리콘솔에서 Cloudformation을 선택합니다.

![](.gitbook/assets/image%20%282%29.png)

앞서 다운로드 해둔 yaml 파일 중에서, 아래 그림과 같이 GWLBVPC.yml 파일을 선택합니다.

![](.gitbook/assets/image%20%284%29.png)

스택 세부 정보 지정에서 , 스택이름과 VPC Parameters를 지정합니다. 대부분 기본값을 사용하면 됩니다.

* 스택이름 : GWLBVPC
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.254.0.0/16
* PublicSubnetABlock: 10.254.11.0/24
* PublicSubnetBBlock: 10.254.12.0/24
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.

![](.gitbook/assets/image%20%283%29.png)

![](.gitbook/assets/image%20%289%29.png)

![](.gitbook/assets/image.png)



![](.gitbook/assets/image%20%281%29.png)

![](.gitbook/assets/image%20%286%29.png)

![](.gitbook/assets/image%20%2811%29.png)

