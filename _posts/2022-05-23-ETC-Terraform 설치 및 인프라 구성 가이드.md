---
layout: post
title: Readmine 설치 가이드

description: >
  참고 <br>
  - Terraform 설치 : <https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started><br>
  - 인프라 구축 : <https://learn.hashicorp.com/tutorials/terraform/aws-build?in=terraform/aws-get-started><br>
tags: [ETC]
---
## AWS & Terraform [Terraform]
## AWS & Terraform [Terraform]

Terraform은 IaC(Infrastructure as Code) 도구로서 그래픽 사용자 인터페이스가 아닌 구성 파일을 사용하여 인프라를 관리할 수 있습니다. 

Terraform을 사용하면 인프라를 수동으로 관리하는 것보다 몇 가지 이점이 있습니다.

* Terraform은 여러 클라우드 플랫폼에서 인프라를 관리할 수 있습니다.
* 사람이 읽을 수 있는 구성 언어를 사용하면 인프라 코드를 빠르게 작성할 수 있습니다.
* Terraform의 상태를 통해 배포 전반에 걸쳐 리소스 변경 사항을 추적할 수 있습니다.
* 인프라에서 안전하게 협업하기 위해 구성을 버전 제어에 커밋할 수 있습니다.


Terraform으로 배포하기 위한 단계는 아래와 같습니다.

1. Scope - 프로젝트의 인프라를 식별합니다.
2. Author - 인프라에 대한 구성을 작성합니다.
3. Initialize - Terraform이 인프라를 관리하는 데 필요한 플러그인을 설치합니다.
4. Plan - 구성과 일치하도록 Terraform이 수행할 변경 사항을 미리 봅니다.
5. Apply - 계획된 변경을 수행합니다.


## Terraform 설치

linux 환경에서 아래의 순서대로 진행합니다.
``` shell
$ sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
```

HashiCorp GPG 키를 추가합니다.
``` shell
$ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
```

공식 HashiCorp Linux 저장소를 추가합니다.
``` shell
$ sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
```

업데이트하여 저장소를 추가하고 Terraform CLI를 설치합니다.
``` shell
$ sudo apt-get update && sudo apt-get install terraform
```


### 설치 확인

Download 또는 Install이 정상적으로 되었는지는 아래의 방법으로 쉽게 확인이 가능합니다.
```
$ terraform version
Terraform v1.1.9
```

## 인프라 구축

### Write 구성

각 Terraform 구성은 자체 작업 디렉토리에 있어야 합니다.
``` shell
$ mkdir learn-terraform-aws-instance
$ cd learn-terraform-aws-instance
```

인프라를 정의하는 파일을 작성합니다. 

중복되는 값은 변수로 선언해 따로 관리하기 위해 main.tf와 variable.tf로 분리하여 작성합니다. variable.tf 파일에 정의한 변수는 main.tf 파일에서 var.변수명 으로 사용할 수 있습니다.

#### 변수파일(variable.tf) 작성 예시

``` shell
$ vi variable.tf
```

```
variable "image_id" {
  description = "내EC2 instance AMI"
  type        = string 
  default     = "ami-0faa64ace1b9511c8" # 변수 값
  validation { # 유효성 검사
    condition     = length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-" # 조건 체크
    error_message = "The image_id value must be a valid AMI id, starting with \"ami-\"." # 에러메세지 
  }
}

...

```


#### 메인파일(main.tf) 예시
``` shell
$ vi main.tf
```

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.27"
    }
  }

  required_version = ">= 0.14.9"
}

provider "aws" {
  region = var.region
}

resource "aws_instance" "terraform_test_server" {
  ami           = var.image_id
  instance_type = "t3.nano"
  key_name  = var.key_name
  subnet_id = "************"
  security_groups = ["${aws_security_group.sg.name}"]
  iam_instance_profile = var.iam_role
  ebs_block_device {
    device_name = var.device_name
    volume_size = var.volume_size
    volume_type = var.volume_type
  }

  tags = {
    Name = "${var.resource_name}_server"
  }
}

...

```


### Terraform
  
 Terraform 블록은 인프라를 프로비저닝하는 데 사용할 필수 공급자를 포함한 Terraform 설정이 포함됩니다. 각 공급자에 대해 source특성은 **선택적 호스트 이름, 네임스페이스 및 공급자 유형을 정의**합니다.

 required_providers 블록에 정의된 각 공급자에 대한 [버전 제약 조건을 설정](https://www.terraform.io/language/expressions/version-constraints)할 수도 있습니다.  


### Provider

Provider 블록은 지정된 공급자(AWS, GCP, AZURE...)에 대한 설정을 수행합니다. Provider는 Terraform이 code를 수행하기 위해 필요한 정보들을 제공해주는 역할을 수행합니다.

#### Credentials 설정

IAM 자격 증명을 사용하여 Terraform AWS 공급자를 인증하려면 AWS_ACCESS_KEY_ID 환경 변수를 설정합니다. 아래의 두가지 방법 중 하나를 택해 설정을 진행할 수 있습니다.

1. [credential 파일](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)이 없는 경우 Credentials을 환경변수로 선언하여 설정
``` shell
$ export AWS_ACCESS_KEY_ID="<YOUR_AWS_ACCESS_KEY_ID>"
$ export AWS_SECRET_ACCESS_KEY="<YOUR_AWS_SECRET_ACCESS_KEY>"
$ export AWS_DEFAULT_REGION="<YOUR_AWS_DEFAULT_REGION>"
```

2. credential 파일이 존재한다면 variable.tf파일의 provider 블록에서 code를 추가하여 설정
``` shell
provider "aws" {
  region                  = var.region
  shared_credentials_file = "Credential 파일 위치"
  profile                 = "Credential 파일 내의 저장된 profile 중 선택"
}
```

* [aws provider에서 사용할 수 있는 arguments](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#argument-reference)
  

### Resource

Resource 블록은 인프라의 구성 요소를 정의합니다.

리소스 블록에는 블록 앞에 리소스 타입과 리소스 네임이라는 두 개의 문자열이 있습니다. 이 예에서 리소스 타입은 aws_instance이고 네임은 app_server입니다. 리소스 유형과 리소스 이름은 함께 리소스의 고유 ID를 형성합니다. 예를 들어 EC2 인스턴스의 ID는 입니다 aws_instance.app_server.

#### 주요 Resource Type 종류 및 argumants

#### [aws_instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)

* ami : (선택 사항) 인스턴스에 사용할 AMI. launch_template가 지정되고 시작 템플릿이 AMI를 지정 하지 않는 한 필수입니다
  
* instance_type : (선택 사항) 인스턴스에 사용할 인스턴스 유형입니다. 이 필드를 업데이트하면 EC2 인스턴스의 중지/시작이 트리거됩니다.
  
* ebs_block_device : EBS(Elastic Block Store) 로 생성한 디스크
  * device_name : (필수) 마운트할 장치의 이름입니다.
  * volume_size : (선택 사항) 기가바이트(GiB) 단위의 볼륨 크기입니다.
  * volume_type : (선택 사항) 볼륨 유형입니다. 유효한 값은 standard, gp2, gp3, io1, io2, sc1또는 st1입니다. 기본값은 gp2입니다.
  
* subnet_id : (선택 사항) VPC 서브넷 ID입니다.

* security_groups : (선택 사항) 연결된 보안 그룹입니다.

* iam_instance_profile : (선택 사항) 인스턴스를 시작할 IAM 인스턴스 프로파일입니다. 

* key_name : (선택 사항) 인스턴스에 사용할 키 쌍의 키 이름입니다.
  

#### [aws_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc)

* cidr_block : (선택 사항) VPC에 대한 IPv4 CIDR 블록입니다.
  
#### [aws_subnet](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/subnet)

*  vpc_id :(필수) 연결할 VPC ID입니다.
  
*  cidr_block : (선택 사항) 서브넷의 IPv4 CIDR 블록입니다.
  
*  availability_zone : (선택 사항) 서브넷의 어빌리티 존 입니다. (ex. ap-northeast-2)

#### [aws_ami](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ami)

* name- (필수) AMI의 리전 고유 이름입니다.

* virtualization_type : (선택 사항) 생성된 인스턴스가 사용할 가상화 모드를 선택하는 키워드입니다. "반가상화"(기본값) 또는 "hvm"일 수 있습니다.
  
* root_device_name : (선택 사항) 루트 장치의 이름(ex. /dev/sda1, /dev/xvda)
  
#### [aws_network_interface](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/network_interface)

* subnet_id : (필수) ENI를 생성할 서브넷 ID입니다.

* private_ips : (선택 사항) 순서와 상관없이 ENI에 할당할 사설 IP 목록입니다.

* security_groups : (선택 사항) ENI에 할당할 보안 그룹 ID 목록입니다.
  
#### [aws_eip](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip)

* instance : 선택 사항) EC2 인스턴스 ID입니다.
  
* vpc : EIP가 VPC에 있는지를 나타내닌 참/거짓 값입니다.

* network_interface : (선택 사항) 연결할 네트워크 인터페이스 ID입니다.

* associate_with_private_ip : (선택 사항) 탄력적 IP 주소와 연결할 사용자 지정 기본 또는 보조 사설 IP 주소입니다.
  
#### [aws_key_pair](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/key_pair)

* key_name : (선택 사항) 키 쌍의 이름입니다. 

* public_key : (필수) 공개 키 자료입니다.

#### [aws_elb](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/elb)

* availability_zones : (EC2-classic ELB에 필요) 트래픽을 처리할 AZ입니다.
  
* subnets : (VPC ELB의 경우 필수) ELB에 연결할 서브넷 ID 목록입니다.
  
* listener : (필수) 리스너 블록 목록입니다.
  * instance_port : (필수) 라우팅할 인스턴스의 포트
  * instance_protocol- (필수) 인스턴스에 사용할 프로토콜(HTTP, HTTPS, TCP, SSL)입니다
  * lb_port : (필수) 로드 밸런서를 수신 대기할 포트입니다.
  * lb_protocol : (필수) 수신할 프로토콜입(HTTP, HTTPS, TCP, SSL)니다.
  
#### [aws_security_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)

* name : (선택 사항) 보안 그룹의 이름입니다. 생략하면 Terraform은 임의의 고유한 이름을 할당합니다.

* description : (선택 사항) 이 수신 규칙에 대한 설명입니다.

* vpc_id : (선택 사항) VPC ID 입니다.

* ingress : (선택 사항) 수신 규칙에 대한 구성 블록입니다. 각 수신 규칙에 대해 여러 번 지정할 수 있습니다.
  * from_port : (필수) 시작 포트(또는 프로토콜이 icmp또는 icmpv6인 경우 ICMP 유형 번호)
  * to_port : (필수) 끝 범위 포트(또는 프로토콜이 icmp인 경우 ICMP 코드 icmp)
  * protocol : (필수) [프로토콜](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_IpPermission.html)입니다 (ex. tcp, udp, icmp)
  * cidr_blocks :(선택 사항) CIDR 블록 목록입니다.

* egress : (선택 사항, VPC만 해당)송신 규칙에 대한 구성 블록입니다. 각 송신 규칙에 대해 여러 번 지정할 수 있습니다.
  * from_port : (필수) 시작 포트(또는 프로토콜이 icmp인 경우 ICMP 유형 번호)
  * to_port : (필수) 끝 범위 포트(또는 프로토콜이 icmp인 경우 ICMP 코드 icmp)
  * protocol : (필수) [프로토콜](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_IpPermission.html)입니다 (ex. tcp, udp, icmp)
  * cidr_blocks :(선택 사항) CIDR 블록 목록입니다.


#### 디렉터리 초기화

새 구성을 생성하거나 버전 제어에서 기존 구성을 체크아웃할 때 디렉토리를 **terraform init**로 초기화해야 합니다 

구성 디렉토리를 초기화하면 구성에 정의된 공급자(이 경우 aws공급자)가 다운로드 및 설치됩니다.

``` shell
$ terraform init

Initializing the backend...
```

#### 구성 미리보기

terraform plan 명령을 사용하여 적용시킬 구성을 미리 볼 수 있습니다.

``` shell
$ terraform plan
```

#### 인프라 생성

terraform apply 명령을 사용하여 구성을 적용한다.
``` shell
$ terraform apply

```

#### 상태 검사

Terraform은 terraform.tfstate에 관리하는 리소스의 ID와 속성을 저장하므로 앞으로 해당 리소스를 업데이트하거나 삭제할 수 있습니다.

Terraform 상태 파일은 Terraform이 관리하는 리소스를 추적할 수 있습니다. 민감한 정보를 포함하는 경우가 많으므로 상태 파일을 안전하게 저장하고 인프라를 관리해야 하는 신뢰할 수 있는 팀원만 액세스하도록 제한해야 합니다.

**terraform show** 명령어로 현재 상태를 검사할 수 있습니다.

``` shell
$ terraform show terraform show "<File Name>"
```