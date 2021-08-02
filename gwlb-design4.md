# GWLB Desing 4

## 목표 구성 개요

2개의 각 워크로드 VPC \(VPC01,02\)은 Account내에 구성된 GWLB 기반의 보안 VPC를 통해서 내, 외부 트래픽을 처리하는 구성입니다. GWLB 기반의 보안 VPC는 2개의 AZ에 4개의 가상 Appliance가 로드밸런싱을 통해 처리 됩니다.

이러한 구성은 VPC Endpoint를 특정 VPC에 구성하고, TransitGateway를 통해 GWLB에 VPC Endpoint Service를 연결하는 중앙집중 구조입니다.

GWLB Design2와 다른 점은 ALB\(Application Load Balancer\)를 

아래 그림은 목표 구성도 입니다.

**🎬 아래 동영상 링크에서 구성방법을 확인 할 수 있습니다.**

![](.gitbook/assets/image%20%28147%29.png)



