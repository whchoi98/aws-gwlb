# GWLB Design 3

### 목표 구성 개요

3개의 각 워크로드 VPC \(VPC01,02,03\)은 Account내에 구성된 GWLB 기반의 보안 VPC를 통해서 내, 외부 트래픽을 처리하는 구성입니다. GWLB 기반의 보안 VPC는 2개의 AZ에 4개의 가상 Appliance가 로드밸런싱을 통해 처리 됩니다.

이러한 구성은 VPC Endpoint를 각 VPC에 분산하고, GWLB에 VPC Endpoint Service를 연결하는 분산형 구조입니다.

아래 그림은 목표 구성도 입니다.  


