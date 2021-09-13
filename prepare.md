---
description: 'Update : 2021-03-31/ 20min'
---

# 사전 준비

## 시작에 앞서 

이 Lab에서는 GWLB와 연동되는 디자인을 4가지로 구성합니다.

#### Design 1 - VPC 에서 외부 전송 트래픽에 대해 GWLB VPC Endpoint와 Private Link를 사용해서 구성합니다.

#### Design 2 - VPC 에서 외부 전송 트래픽에 대해 TransitGateway를 기반으로 구성합니다. TransitGateway와 연동된 VPC에서 GWLB VPC Endpoint Link를 사용해서 보안 어플라이언스로 연동됩니다.

#### Design 3 - VPC 에서 외부 전송 트래픽에 대해 GWLB VPC Endpoint와 Private Link를 사용해서 구성합니다. 외부에 서비스를 제공하기 위해 ALB를 구성합니다. 내부 Private Subnet 자원들의 패치를 위해 NAT Gateway를 구성합니다.

#### Design 4 - VPC 에서 외부 전송 트래픽에 대해 TransitGateway를 기반으로 구성합니다. 외부에 서비스를 제공하기 위해 ALB를 구성합니다. 내부 Private Subnet 자원들의 패치를 위해 NAT Gateway를 구성합니다.

보안 어플라이언스는 상용 방화벽이나 기타 어플라이언스를 연동 가능합니다. 이 랩에서는 리눅스 기반 IPTABLE을 사용합니다.

## Cloud9 구성

### Cloud9 소개 

AWS Cloud9은 브라우저만으로 코드를 작성, 실행 및 디버깅할 수 있는 클라우드 기반 IDE\(통합 개발 환경\)입니다. 코드 편집기, 디버거 및 터미널이 포함되어 있습니다. Cloud9은 JavaScript, Python, PHP를 비롯하여 널리 사용되는 프로그래밍 언어를 위한 필수 도구가 사전에 패키징되어 제공되므로, 새로운 프로젝트를 시작하기 위해 파일을 설치하거나 개발 머신을 구성할 필요가 없습니다. Cloud9 IDE는 클라우드 기반이므로, 인터넷이 연결된 머신을 사용하여 사무실, 집 또는 어디서든 프로젝트 작업을 할 수 있습니다. 또한, Cloud9은 서버리스 애플리케이션을 개발할 수 있는 원활한 환경을 제공하므로 손쉽게 서버리스 애플리케이션의 리소스를 정의하고, 디버깅하고, 로컬 실행과 원격 실행 간에 전환할 수 있습니다. Cloud9에서는 개발 환경을 팀과 신속하게 공유할 수 있으므로 프로그램을 연결하고 서로의 입력 값을 실시간으로 추적할 수 있습니다.

🎬 아래 동영상 링크에서 구성방법을 확인 할 수 있습니다. 

{% embed url="https://youtu.be/Jdzj0fSA4YU" %}



### Cloud9 구성

Cloud9을 실행하기 위해 아래와 같이 AWS 관리콘솔에서 **`"Cloud9"`** 을 입력합니다.

![](.gitbook/assets/image%20%2889%29.png)

**`AWS 관리 콘솔 - Cloud9 - Create environment`**를 선택합니다.

* name : gwlb-console \(고유의 이름을 입력 해야 합니. 예 : username-console\)

![](.gitbook/assets/image%20%2832%29.png)

모든 설정을 기본값으로 사용하고, 인스턴스타입은 t3.small ,Cost-Saving Setting Never로 변경합니다. 절전모드로 변경되는 것을 방지하게 됩니다. 다음 진행 버튼을 계속 누르고 Cloud9을 생성합니다.

* instance type : t3.small
* Cost-saving setting : Never
* 기타 옵션 : 기본

![](.gitbook/assets/image%20%2888%29.png)

2~3분 후에 Cloud9 이 동작하는 것을 확인 할 수 있습니다. Cloud9 창에서 "+" 버튼을 누르고 New Terminal을 띄워서 터미널을 생성합니다. 추가로 "+"를 계속 생성하게 되면 Terminal을 다중으로 사용할 수 있습니다.

![](.gitbook/assets/image%20%2829%29.png)

Cloud9 IDE는 이미 AWS CLI가 설치되어 있습니다. 하지만 기본 1.x 버전이 설치되어 있습니다.

```text
$ aws --version
aws-cli/1.19.39 Python/2.7.18 Linux/4.14.225-169.362.amzn2.x86_64 botocore/1.20.39
```

아래 명령을 통해 CLI를 2.0으로 업그레이드합니다.

```text
# AWS CLI upgrade
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

```

정상적으로 업그레이드 되었는지 확인합니다.

```text
source ~/.bashrc
aws --version

```

aws cli 자동완성을 설치 합니다.

```text
# aws cli 자동완성 설치 
which aws_completer
export PATH=/usr/local/bin:$PATH
source ~/.bash_profile
complete -C '/usr/local/bin/aws_completer' aws

```

## keypair 만들기

keypair를 Cloud9에서 생성합니다.

```text
ssh-keygen

```

key이름은 gwlbkey 로 설정합니다.

```text
gwlbkey
```

아래와 같이 ssh key가 구성됩니다.

```text
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ec2-user/.ssh/id_rsa): gwlbkey
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in gwlbkey.
Your public key has been saved in gwlbkey.pub.
The key fingerprint is:
SHA256:ZId12JDdlSjIuhBym08BKU/EtYbMj9EkCYtTwYpP9sY ec2-user@ip-172-31-63-114.ap-northeast-2.compute.internal
The key's randomart image is:
+---[RSA 2048]----+
|  .+=+=o. +*....o|
|  o++B=..=ooo... |
|.o..*=++* . .    |
|..+  === .       |
| + o .+.S        |
|  . E  o         |
|   .             |
|                 |
|                 |
+----[SHA256]-----+
```

이제 생성된 Public Key를 계정으로 업로드 합니다. **`"--region {AWS Region}"`** 리전 옵션에서 각 리전을 지정하게 되면 해당 리전으로 생성한 Public Key를 전송합니다. 아래에서는 도쿄리전으로 전송하는 예제입니다.

```text
#Tokoy Region 전
aws ec2 import-key-pair --key-name "gwlbkey" --public-key-material fileb://gwlbkey.pub --region ap-northeast-1
#Seoul Region 전송
aws ec2 import-key-pair --key-name "gwlbkey" --public-key-material fileb://gwlbkey.pub --region ap-northeast-2
#버지니아 리전 전송
aws ec2 import-key-pair --key-name "gwlbkey" --public-key-material fileb://gwlbkey.pub --region us-east-1
#오레곤 리전 전송
aws ec2 import-key-pair --key-name "gwlbkey" --public-key-material fileb://gwlbkey.pub --region us-west-2

```

아래와 같이 업로드가 완료됩니다.

```text
whchoi:~/environment $ aws ec2 import-key-pair --key-name "gwlbkey" --public-key-material fileb://gwlbkey.pub --region ap-northeast-2
{
    "KeyFingerprint": "xx:xx:xx:xx:xx:65:3a:70:fb:b1:fa:dd:6c:59:c6:9e",
    "KeyName": "gwlbkey",
    "KeyPairId": "key-xxxxxxxxx"
}
```

정상적으로 public key가 업로드되었는지 AWS 관리콘솔에서 확인합니다.

**`AWS 관리 콘솔 - EC2 - 네트워크 및 보안 - 키페어`**

![](.gitbook/assets/image%20%283%29.png)

이제 사전 구성이 완료되었습니다.

