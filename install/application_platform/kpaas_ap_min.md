### [Index](https://github.com/K-PaaS/Guide/blob/master/README.md) > [AP Install](../README.md) > K-PaaS AP - min

## Table of Contents

1. [개요](#1)  
 1.1. [목적](#1.1)  
 1.2. [범위](#1.2)  
 1.3. [참고 자료](#1.3)  
2. [K-PaaS AP min 설치](#2)  
 2.1. [Prerequisite](#2.1)  
 2.2. [설치 파일 다운로드](#2.2)  
 2.3. [Stemcell 업로드](#2.3)  
 2.4. [Runtime Config 설정](#2.4)  
 2.5. [Cloud Config 설정](#2.5)  
 2.6. [K-PaaS AP min 설치 파일](#2.6)  
　2.6.1. [K-PaaS AP min 설치 Variable 파일](#2.6.1)    
　2.6.2. [K-PaaS AP min Operation 파일](#2.6.2)  
　2.6.3. [K-PaaS AP min 설치 Shell Scripts](#2.6.3)  
 2.7. [K-PaaS AP min 설치](#2.7)  
 2.8. [K-PaaS AP min 로그인](#2.8)   

# <div id='1'/>1.  문서 개요

## <div id='1.1'/>1.1. 목적
본 문서는 Monitoring을 적용하지 않은 K-PaaS Application Platform 경량화(이하 K-PaaS AP min)을 수동으로 설치하기 위한 가이드를 제공하는 데 그 목적이 있다.

<br>

## <div id='1.2'/>1.2. 범위
K-PaaS AP min은 bosh-deployment를 기반으로 한 BOSH 환경에서 설치하며 ap-deployment v5.8.9-min의 설치를 기준으로 가이드를 작성하였다.  
K-PaaS AP min은 VMware vSphere, Google Cloud Platform, Amazon Web Services EC2, OpenStack, Microsoft Azure 등의 IaaS를 지원하며,  ap-deployment v5.8.9-min에서 검증한 IaaS 환경은 OpenStack, vSphere 환경이다.

<br>

## <div id='1.3'/>1.3. 참고 자료

본 문서는 Cloud Foundry의 BOSH Document와 Cloud Foundry Document를 참고로 작성하였다.

BOSH Document: [http://bosh.io](http://bosh.io)  
Cloud Foundry Document: [https://docs.cloudfoundry.org](https://docs.cloudfoundry.org)  
BOSH Deployment: [https://github.com/cloudfoundry/bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)  
CF Deployment: [https://github.com/cloudfoundry/cf-deployment](https://github.com/cloudfoundry/cf-deployment)  

<br><br>

# <div id='2'/>2. K-PaaS AP min 설치
## <div id='2.1'/>2.1. Prerequisite

- BOSH2 기반의 BOSH를 설치한다.
- K-PaaS AP min 설치는 BOSH를 설치한 Inception(설치 환경)에서 작업한다.
- K-PaaS AP min 설치를 위해 BOSH LOGIN을 진행한다.
- 가이드 내에 BOSH 폴더에서 실행되는 작업은 BOSH 설치 가이드에서 다운받은 ap-deployment를 이용한다.

<br>

## <div id='2.2'/>2.2. 설치 파일 다운로드
- K-PaaS AP min를 설치하기 위한 deployment가 존재하지 않는다면 다운로드 받는다

```
$ mkdir -p ~/workspace
$ cd ~/workspace
$ git clone https://github.com/K-PaaS/common.git
$ cd ~/workspace
$ git clone https://github.com/K-PaaS/ap-deployment.git -b v5.8.9-min ap-deployment-min
```

<br>

## <div id='2.3'/>2.3. Stemcell 업로드
Stemcell은 배포 시 생성되는 VM Base OS Image이다.  
ap-deployment v5.8.9-min은 Ubuntu jammy stemcell 1.260을 기반으로 한다.  
기본적인 Stemcell 업로드 명령어는 다음과 같다.  
```                     
$ bosh -e ${BOSH_ENVIRONMENT} upload-stemcell {URL}
```

ap-deployment는 v5.5.0 부터 Stemcell 업로드 스크립트를 지원하며, BOSH 로그인 후 다음 명령어를 수행하여 Stemcell을 올린다.  
BOSH_ENVIRONMENT는 BOSH 설치 시 사용한 Director 명이고, CURRENT_IAAS는 배포된 환경 IaaS(aws, azure, gcp, openstack, vsphere, 그외 입력시 bosh-lite)에 맞게 입력을 한다.
<br>(ap-deployment에서 제공되는 create-bosh-login.sh을 이용하여 BOSH LOGIN시 BOSH_ENVIRONMENT와 CURRENT_IAAS는 자동입력된다.)

- Stemcell 업로드 Script의 설정 수정 (BOSH_ENVIRONMENT 수정)

> $ vi ~/workspace/ap-deployment/bosh/upload-stemcell.sh
```                     
#!/bin/bash
JAMMY_STEMCELL_VERSION=1.260
CURRENT_IAAS="${CURRENT_IAAS}"        # IaaS Information (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 aws/azure/gcp/openstack/vsphere/bosh-lite 입력)
BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"      # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

if [[ ${CURRENT_IAAS} = "aws" ]]; then
        bosh -e ${BOSH_ENVIRONMENT} upload-stemcell https://storage.googleapis.com/bosh-core-stemcells/${JAMMY_STEMCELL_VERSION}/bosh-stemcell-${JAMMY_STEMCELL_VERSION}-aws-xen-hvm-ubuntu-jammy-go_agent.tgz -n
elif [[ ${CURRENT_IAAS} = "azure" ]]; then
        bosh -e ${BOSH_ENVIRONMENT} upload-stemcell https://storage.googleapis.com/bosh-core-stemcells/${JAMMY_STEMCELL_VERSION}/bosh-stemcell-${JAMMY_STEMCELL_VERSION}-azure-hyperv-ubuntu-jammy-go_agent.tgz -n
elif [[ ${CURRENT_IAAS} = "gcp" ]]; then
        bosh -e ${BOSH_ENVIRONMENT} upload-stemcell https://storage.googleapis.com/bosh-core-stemcells/${JAMMY_STEMCELL_VERSION}/bosh-stemcell-${JAMMY_STEMCELL_VERSION}-google-kvm-ubuntu-jammy-go_agent.tgz -n
elif [[ ${CURRENT_IAAS} = "openstack" ]]; then
        bosh -e ${BOSH_ENVIRONMENT} upload-stemcell https://storage.googleapis.com/bosh-core-stemcells/${JAMMY_STEMCELL_VERSION}/bosh-stemcell-${JAMMY_STEMCELL_VERSION}-openstack-kvm-ubuntu-jammy-go_agent.tgz -n
elif [[ ${CURRENT_IAAS} = "vsphere" ]]; then
        bosh -e ${BOSH_ENVIRONMENT} upload-stemcell https://storage.googleapis.com/bosh-core-stemcells/${JAMMY_STEMCELL_VERSION}/bosh-stemcell-${JAMMY_STEMCELL_VERSION}-vsphere-esxi-ubuntu-jammy-go_agent.tgz -n
elif [[ ${CURRENT_IAAS} = "bosh-lite" ]]; then
        bosh -e ${BOSH_ENVIRONMENT} upload-stemcell https://storage.googleapis.com/bosh-core-stemcells/${JAMMY_STEMCELL_VERSION}/bosh-stemcell-${JAMMY_STEMCELL_VERSION}-warden-boshlite-ubuntu-jammy-go_agent.tgz -n
else
        echo "plz check CURRENT_IAAS"
fi

```

- Stemcell 업로드 Script 실행

```
$ cd ~/workspace/ap-deployment/bosh
$ source upload-stemcell.sh
```

<br>

## <div id='2.4'/>2.4. Runtime Config 설정
Runtime config는 BOSH로 배포되는 VM에 일괄 적용되는 설정이다.
기본적인 Runtime Config 설정 명령어는 다음과 같다.  
```                     
$ bosh -e ${BOSH_ENVIRONMENT} update-runtime-config {PATH} --name={NAME}
```

K-PaaS AP min에서 적용하는 Runtime Config는 다음과 같다.  

- DNS Runtime Config
  K-PaaS AP Component 간의 통신을 위해 BOSH DNS 배포가 선행되어야 한다.  

- OS Configuration Runtime Config  
  BOSH Linux OS 구성 릴리스를 이용하여 sysctl을 구성한다.  

ap-deployment는 v5.5.0 부터 Runtime Config 설정 스크립트를 지원하며, BOSH 로그인 후 다음 명령어를 수행하여 Runtime Config를 설정한다.  

  - Runtime Config 업데이트 Script 수정 (BOSH_ENVIRONMENT 수정)
> $ vi ~/workspace/ap-deployment/bosh/update-runtime-config.sh
```                     
#!/bin/bash

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"			 # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} update-runtime-config -n runtime-configs/dns.yml
bosh -e ${BOSH_ENVIRONMENT} update-runtime-config -n --name=os-conf runtime-configs/os-conf.yml
```
- Runtime Config 업데이트 Script 실행
```                     
$ cd ~/workspace/ap-deployment/bosh
$ source update-runtime-config.sh
```

  - Runtime Config 확인  
  ```  
  $ bosh -e ${BOSH_ENVIRONMENT} runtime-config
  $ bosh -e ${BOSH_ENVIRONMENT} runtime-config --name=os-conf
  ```

<br>

## <div id='2.5'/>2.5. Cloud Config 설정

BOSH를 통해 VM을 배포 시 IaaS 관련 Network, Storage, VM 관련 설정을 Cloud Config로 정의한다.  
deployment 설치 파일을 내려받으면 ~/workspace/ap-deployment-min/cloud-config 디렉터리 이하에 IaaS 별 Cloud Config 예제를 확인할 수 있으며, 예제를 참고하여 cloud-config.yml을 IaaS에 맞게 수정한다.  
K-PaaS AP min 배포 전에 Cloud Config를 BOSH에 적용해야 한다.

- AWS을 기준으로 한 [cloud-config.yml](https://github.com/K-PaaS/ap-deployment/blob/master/cloud-config/aws-cloud-config.yml) 예제

```
## azs :: 가용 영역(Availability Zone)을 정의한다.
azs:
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z1
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z2

... ((생략)) ...

## compilation :: 컴파일 가상머신이 생성될 가용 영역 및 가상머신 유형 등을 정의한다.
compilation:
  az: z4
  network: default
  reuse_compilation_vms: true
  vm_type: xlarge
  workers: 5

## disk_types :: 디스크 유형(Disk Type, Persistent Disk)을 정의한다.
disk_types:
- disk_size: 1024
  name: default
- disk_size: 1024
  name: 1GB
  
... ((생략)) ...

## networks :: 네트워크(Network)를 정의한다. (AWS 경우, Subnet 및 Security Group, DNS, Gateway 등 설정)
networks:
- name: default
  subnets:
  - az: z1
    cloud_properties:
      security_groups: ap-v50-security
      subnet: subnet-XXXXXXXXXXXXXXXXX
    dns:
    - 8.8.8.8
    gateway: 10.0.1.1
    range: 10.0.1.0/24
    reserved:
    - 10.0.1.2 - 10.0.1.9
    static:
    - 10.0.1.10 - 10.0.1.120

... ((생략)) ...

## vm_extentions :: 임의의 특정 IaaS 구성을 지정하는 가상머신 구성을 정의한다. (Security Groups 및 Load Balancers 등)
vm_extensions:
- name: cf-router-network-properties
- name: cf-tcp-router-network-properties
- name: diego-ssh-proxy-network-properties
- name: cf-haproxy-network-properties
- cloud_properties:
    ephemeral_disk:
      size: 51200
      type: gp2
  name: 50GB_ephemeral_disk

... ((생략)) ...

## vm_type :: 가상머신 유형(VM Type)을 정의한다. (AWS 경우, Instance type 설정)
vm_types:
- cloud_properties:
    ephemeral_disk:
      size: 3000
      type: gp2
    instance_type: t2.small
  name: minimal
- cloud_properties:
    ephemeral_disk:
      size: 10000
      type: gp2
    instance_type: t2.small
  name: small
  
... ((생략)) ...
```

- AZs

K-PaaS에서 제공되는 Cloud Config 예제는 z1 ~ z6까지 설정되어 있다.  
z1 ~ z3까지는 K-PaaS AP min VM이 설치되는 Zone이며, z4 ~ z6까지는 서비스가 설치되는 Zone으로 정의한다.   
3개 단위로 설정하는 이유는 서비스 3중화를 위해서이며, 설치하는 환경에 따라 다르게 설정해도 무방하다.  

- VM Types

VM Type은 IaaS에서 정의된 VM Type이다.  

※ 다음은 AWS에서 정의한 Instance Type이다.
![FLAVOR_Image]

- Compilation

K-PaaS AP min 및 서비스 설치 시, BOSH는 Compile 작업용 VM을 생성하여 소스를 컴파일하고, 이후 VM을 생성하여 컴파일된 파일을 대상 VM에 설치한 뒤 Compile 작업용 VM은 삭제된다. (Worker 수는 Compile VM의 수로, 많을수록 컴파일 속도가 빨라진다.)  

- Disk Size

K-PaaS AP min 및 서비스가 설치되는 VM의 Persistent Disk Size이다.

- Networks

Networks는 AZ 별 Subnet Network, DNS, Security Groups, Network ID를 정의한다.  
보통 AZ 별로 256개의 IP를 정의할 수 있도록 Range Cider를 정의한다.

<br>

- Cloud Config 업데이트

```
$ bosh -e ${BOSH_ENVIRONMENT} update-cloud-config ~/workspace/ap-deployment-min/cloud-config/{iaas}-cloud-config.yml
```

- Cloud Config 확인

```
$ bosh -e ${BOSH_ENVIRONMENT} cloud-config  
```

<br>

## <div id='2.6'/>2.6.  K-PaaS AP min 설치 파일

common_vars.yml파일과 vars.yml을 수정하여 K-PaaS AP min 설치시 적용하는 변수를 설정할 수 있다.

<table>
<tr>
<td>common_vars.yml</td>
<td>K-PaaS AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일</td>
</tr>
<tr>
<td>min-vars.yml</td>
<td>K-PaaS AP min 설치시 적용하는 변수 설정 파일</td>
</tr>
<tr>
<td>deploy-aws-4vms.sh</td>
<td>AWS 환경에 K-PaaS AP min 4vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-aws-7vms.sh</td>
<td>AWS 환경에 K-PaaS AP min 7vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-openstack-4vms.sh</td>
<td>OpenStack 환경에 K-PaaS AP min 4vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-openstack-7vms.sh</td>
<td>OpenStack 환경에 K-PaaS AP min 7vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-vsphere-4vms.sh</td>
<td>vSphere 환경에 K-PaaS AP min 4vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-vsphere-7vms.sh</td>
<td>vSphere 환경에 K-PaaS AP min 7vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-azure-4vms.sh</td>
<td>Azure 환경에 K-PaaS AP min 4vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-azure-7vms.sh</td>
<td>Azure 환경에 K-PaaS AP min 7vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-gcp-4vms.sh</td>
<td>GCP 환경에 K-PaaS AP min 4vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>deploy-gcp-7vms.sh</td>
<td>GCP 환경에 K-PaaS AP min 7vm 설치를 위한 Shell Script 파일</td>
</tr>
<tr>
<td>min-ap-deployment.yml</td>
<td>K-PaaS AP min을 배포하는 Manifest 파일</td>
</tr>
</table>

<br>

### <div id='2.6.1'/>2.6.1. K-PaaS AP min 설치 Variable File


- common_vars.yml  

~/workspace/common 폴더에 있는 [common_vars.yml](https://github.com/K-PaaS/common/blob/master/common_vars.yml)에는 K-PaaS AP min 및 각종 Service 설치 시 적용하는 공통 변수 설정 파일이 존재한다.  
K-PaaS AP min을 설치 시 system_domain, ap_admin_username, ap_admin_password, ap_database_port, ap_cc_db_password, ap_uaa_db_password, uaa_client_admin_secret, uaa_client_portal_secret의 값을 변경 하여 설치 할 수 있다.

> $ vi ~/workspace/common/common_vars.yml

```
... ((생략)) ...

system_domain: "xx.xx.xxx.xxx.nip.io"			# Domain (nip.io를 사용하는 경우 HAProxy Public IP와 동일)
ap_admin_username: "admin"				# Application Platform Admin Username
ap_admin_password: "admin"				# Application Platform Admin Password
ap_database_port: 5524				# Application Platform Database Port (e.g. 5524(postgresql)/13307(mysql)) -- Do Not Use "3306"&"13306" in mysql
ap_cc_db_password: "cc_admin"			# CCDB Password(e.g. "cc_admin")
ap_uaa_db_password: "uaa_admin"			# UAADB Password(e.g. "uaa_admin")
uaa_client_admin_secret: "admin-secret"			# UAAC Admin Client에 접근하기 위한 Secret 변수
uaa_client_portal_secret: "clientsecret"		# UAAC Portal Client에 접근하기 위한 Secret 변수

... ((생략)) ...
```

- min-vars.yml

K-PaaS AP min 설치 할 때 적용되는 각종 변수값이나 배포 될 VM의 설정을 변경할 수 있다.

> $ vi ~/workspace/ap-deployment-min/ap/min-vars.yml
```
# SERVICE VARIABLE
deployment_name: "ap"		# Deployment Name
network_name: "default"			# VM에 별도로 지정하지 않는 Default Network Name
haproxy_public_ip: "52.78.32.153"	# HAProxy IP (Public IP, HAproxy VM 배포시 필요)
haproxy_public_network_name: "vip"	# Application Platform Public Network Name
haproxy_private_network_name: "private" # Application Platform Private Network Name (vSphere use-haproxy-public-network-vsphere.yml 포함 배포시 설정 필요)
cc_db_encryption_key: "db-encryption-key"	# Database Encryption Key (Version Upgrade 시 동일 KEY 필수)
cert_days: 3650				# Application Platform 인증서 유효기간
private_ip: "10.244.0.34"   # Proxy IP (Private IP, BOSH-LITE 사용시 설정 필요)
uaa_login_logout_redirect_parameter_disable: "false"	
uaa_login_logout_redirect_parameter_whitelist: ["http://portal-web-user.15.165.2.88.xip.io","http://portal-web-user.15.165.2.88.xip.io/callback","http://portal-web-user.15.165.2.88.xip.io/login"]	# 포탈 페이지 이동을 위한 UAA Redirect Whitelist 등록 변수
uaa_login_branding_company_name: "K-PaaS R&D"	# UAA 페이지 타이틀 명
uaa_login_branding_footer_legal_text: "Copyright © K-PaaS R&D Foundation, Inc. 2017. All Rights Reserved."	# UAA 페이지 하단 영역 텍스트 
uaa_login_branding_product_logo: "iVBORw0KGgoAAAANSUhEUgAAAM0AAAAdCAYAAAAJguhGAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDkuMS1jMDAxIDc5LjE0NjI4OTk3NzcsIDIwMjMvMDYvMjUtMjM6NTc6MTQgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOkNFRkFFOUQyODg1MDExRUU5MUM0QzUzNkMwNzkwMzIzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOkNFRkFFOUQxODg1MDExRUU5MUM0QzUzNkMwNzkwMzIzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDUzYgKFdpbmRvd3MpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6QUNDMTA1MTZCRDNBMTFFNjkzMTVEQjMxRkE5QjkxNUMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6QUNDMTA1MTdCRDNBMTFFNjkzMTVEQjMxRkE5QjkxNUMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz4FTzpAAAAJtUlEQVR42uxc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"  # UAA 페이지 로고 이미지 (Base64)
uaa_login_branding_square_logo: "iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyxpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDkuMS1jMDAxIDc5LjE0NjI4OTk3NzcsIDIwMjMvMDYvMjUtMjM6NTc6MTQgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjEzRkZCNUMwODg1MTExRUVCQ0I5QThFNDZFMDJCODkzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjEzRkZCNUJGODg1MTExRUVCQ0I5QThFNDZFMDJCODkzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1LjUgKFdpbmRvd3MpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6QkIwMjA5M0Q5NEQ0MTFFNjk1M0FFQ0UxNkIxNEZFNjciIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6QkIwMjA5M0U5NEQ0MTFFNjk1M0FFQ0UxNkIxNEZFNjciLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz5Pn44RAAAEE0lEQVR42sSXW2hcVRSG1zkzmZlkZs7EpGIEERFtRKW1pRK11dk1USki2NQWalCqCKa20GqRFlQUReiLF4R6g1IsSqs+1D7YB2nS02ohtaDtg/gQqEiNJU07tzOZTDI3v52ewSKZyYyTpBt+9rlwzvr3Xmv/ay2jVCrJtRzeq28Mw6j5w1s3RpsSl+SjjGOs9TfLO8lj9p56DJcXbly9A7USwPgapg/GRoxOJ3HlWXNQzkNkQ2LQHpo3Ahju1IaBJiAQKEHA+Pd7kZawnPD5ZV18wL40ZwQwHGF6E2wFTeAC2BW/aPTHx+T+cKvsKxRkeSYtS4XfebxSgMgnXq9sg0ixGgFzlhWb4EUuh8HLrnE9tpw7cHy//s80cVNGMyfteyLt8iRuuFzIi8eJy9ZxRxKR1eqZajbMKsajTL+AT8H14BA4Wl7ATN8kB+3Dk6fsRVabvOttktxUVsLJy7K/+QE13NqtltREAMO3gG+4tMFS9/EuVtzLPFJLgKVs+3Xc0haKyBHTlNJEWm6DyJnQg+rIdd3KqkgAwyGm38F6oIPorPtqtN7zje/T6R/tx612WdIckuFSUYx0UtaAMUi0VdqBFhAA34HF7i40Ohxi2ynHd25KfMTfnbPFwHG2PN6IVVbpCz+kvnRici7jyHI3WGsPwkZGRKnt6ZQk0Ig+jqcZaJG/OSFRAnOiqhQ3OlofVquyE3IgGZOb9H2TT7L4/zWC8v2svr9XybwQwKeR4Cr1E5G+UuuL6ZFSMCxfs+LnCMZszcno/450QjZPC9sVKT7jD0gvhv+oOxs2sAOC0WQgJH2I0feZOr6dsyDM5cQq5uVZoj8gC01AB1uxIAZRvwGhSVhK7VxQAkT6hxyzLaThDELjT8Vkd+A+dYFT0b1gLqAi+jhoSYQc8IXHI8VsRjo4FUeDK9UQiejGeSfg6n/eOWFvCrfJzRzDn/UzUnIXqfkvVHEvOdRYECVMDNgj4yftLtzSgwqOUh+YxMfzuCcwG4Fynu8nMz7SMJFBeyA7ZHfo+uA/xVaukg4kwa9gGfjBvW9oWFG1m2S0o1z5IVS/QezUjDtABpxiWgFeABd1XnFf9bAjVl0JabXq9XepWCouO/M58WqhwiVPU7rdXVUJIaGLyL0Y/Jb5DbAN9GkSIFZDGr59MiuHSEh3uQVqnlXvoUB9ZaYCtaIUQyTF9CpEPmd+DzwBbnBfd8xgOIAafoXhtVqUdO5vseSYzydPYbgi8Zr7Aog86vYE5WrmYGzUWEZ31MkKz+Ym5Q4tQm6T8ifV8Xp8fXquGxO9Yy+Bt/SiaUyk3BnpgRKOo4o7EKXP5rs1a2d6GwKbdWeEn4vswj783K/FqJ7WbPqijHrHoseiPSjcwWpSW42AhnGt2/N/BBgAR36z76Z6aygAAAAASUVORK5CYII="  # UAA 페이지 타이틀 로고 이미지 (Base64)
uaa_login_links_passwd: "http://portal-web-user.15.165.2.88.xip.io/resetpasswd"	# UAA 페이지에서 Reset Password 누를 시 이동하는 링크 주소
uaa_login_links_signup: "http://portal-web-user.15.165.2.88.xip.io/createuser"	# UAA 페이지에서 Create Account 누를 시 이동하는 링크 주소
uaa_client_portal_redirect_uri: "http://portal-web-user.15.165.2.88.xip.io,http://portal-web-user.15.165.2.88.xip.io/callback"	# UAA Portal Client의 Redirect URI 지정 변수, 포탈에서 로그인 버튼 클릭 후 UAA 페이지에서 성공적으로 로그인했을 경우 이동하는 URI 경로

syslog_custom_rule: 'if ($msg contains "DEBUG") then stop'      # [MONITORING] Logging Agent에서 전송할 Custom Rule
syslog_fallback_servers: []             # [MONITORING] Syslog Fallback Servers


# STEMCELL
stemcell_os: "ubuntu-jammy"		# Stemcell OS
stemcell_version: "1.260"		# Stemcell Version

# SMOKE-TEST
smoke_tests_azs: ["z1"]			# Smoke-Test 가용 존
smoke_tests_instances: 1		# Smoke-Test 인스턴스 수
smoke_tests_vm_type: "minimal"		# Smoke-Test VM 종류
smoke_tests_network: "default"		# Smoke-Test 네트워크

# ROTATE-CC-DATABASE-KEY
rotate_cc_database_key_azs: ["z1"] 	# Rotate-CC-Database-Key 가용 존
rotate_cc_database_key_instances: 1 	# Rotate-CC-Database-Key 인스턴스 수
rotate_cc_database_key_vm_type: "minimal" # Rotate-CC-Database-Key VM 종류
rotate_cc_database_key_network: "default" # Rotate-CC-Database-Key 네트워크



## 4VM

# DATABASE
database_azs: ["z1"]      		# Database 가용 존
database_instances: 1     		# Database 인스턴스 수
database_vm_type: "small"   		# Database VM 종류
database_network: "default"   		# Database 네트워크
database_persistent_disk_type: "100GB"  # Database 영구 Disk 종류

# CONTROL
control_azs: ["z1"]  	    		# Control 가용 존
control_instances: 1     		# Control 인스턴스 수
control_vm_type: "small-highmem-16GB"   # Control VM 종류
control_network: "default"   		# Control 네트워크
control_vm_extensions: ["diego-ssh-proxy-network-properties"] # Control VM 확장

# ROUTER
router_azs: ["z1"]    			# Router 가용 존
router_instances: 1     		# Router 인스턴스 수
router_vm_type: "minimal"   		# Router VM 종류
router_network: "default"   		# Router 네트워크
router_vm_extensions: ["cf-router-network-properties"]  # Router VM 확장

# COMPUTE
compute_azs: ["z1"]    			# COMPUTE 가용 존
compute_instances: 1     		# COMPUTE 인스턴스 수
compute_vm_type: "small-highmem-16GB"  	# COMPUTE VM 종류
compute_network: "default"   		# COMPUTE 네트워크
compute_vm_extensions: ["100GB_ephemeral_disk"]  # COMPUTE VM 확장


## 7VM

# SINGLETON-BLOBSTORE
singleton_blobstore_azs: ["z1"]   	# Singleton-Blobstore 가용 존
singleton_blobstore_instances: 1  	# Singleton-Blobstore 인스턴스 수
singleton_blobstore_vm_type: "small"  	# Singleton-Blobstore VM 종류
singleton_blobstore_network: "default"  # Singleton-Blobstore 네트워크
singleton_blobstore_persistent_disk_type: "100GB" # Singleton-Blobstore 영구 Disk 종류

# TCP-ROUTER
tcp_router_azs: ["z1"]    		# TCP-Router 가용 존
tcp_router_instances: 1     		# TCP-Router 인스턴스 수
tcp_router_vm_type: "minimal"   	# TCP-Router VM 종류
tcp_router_network: "default"   	# TCP-Router 네트워크
tcp_router_vm_extensions: ["cf-tcp-router-network-properties"]  # TCP-Router VM 확장

# HAPROXY
haproxy_azs: ["z7"]     		# HAProxy 가용 존
haproxy_instances: 1      		# HAProxy 인스턴스 수
haproxy_vm_type: "minimal"    		# HAProxy VM 종류
haproxy_network: "default"    		# HAProxy 네트워크
```

이하는 UAA와 관련된 변수의 설명이다.

1. uaa_login_logout_redirect_parameter_whitelist : 포탈 페이지 이동을 위한 UAA Redirect Whitelist 등록 변수

```
ex) uaa_login_logout_redirect_parameter_whitelist=["{AP PORTAL URI}","{AP PORTAL URI}/callback","{AP PORTAL URI}/login"]
```

2. uaa_login_links_signup : UAA 페이지에서 Create Account 버튼 클릭 시 이동하는 링크 주소

```
ex) uaa_login_links_signup="{AP PORTAL URI}/createuser"
```

![UAA_Login_Create_Account]

3. uaa_login_links_passwd : UAA 페이지에서 Reset Password 버튼 클릭 시 이동하는 링크 주소

```
ex) uaa_login_links_passwd="{AP PORTAL URI}/resetpasswd"
```

![UAA_Login_Reset_Password]


4. uaa_client_portal_redirect_uri : UAAC Portal Client의 Redirect URI 지정 변수, 포탈에서 로그인 버튼 클릭 후 UAA 페이지에서 로그인 성공 시 이동하는 URI

```
ex) uaa_client_portal_redirect_uri="{AP PORTAL URI}, {AP PORTAL URI}/callback"
```

5. uaa_client_portal_secret : UAAC Portal Client에 접근하기 위한 Secret 변수

```
ex) uaa_client_portal_secret="portalclient"
```

6. uaa_client_admin_secret : UAAC Admin Client에 접근하기 위한 Secret 변수

```
ex) uaa_client_admin_secret="admin-secret"
```

K-PaaS AP min을 설치 후 UAAC의 활용 방법은 사용 가이드에 기타 CLI를 참고한다.

<br>

### <div id='2.6.2'/>2.6.2. K-PaaS AP min Operation 파일

<table>
<tr>
<td>파일명</td>
<td>설명</td>
<td>요구사항</td>
</tr>
<tr>
<td>operations/min-use-postgres.yml</td>
<td>Database를 Postgres로 설치 <br>
    - min-use-postgres.yml 미적용 시 MySQL 설치  <br>
    - 3.5 이전 버전에서 Migration 시 필수  
</td>
<td></td>
</tr>
<tr>
<td>operations/min-use-haproxy.yml</td>
<td>HAProxy 적용 <br>
    - IaaS에서 제공하는 LB를 사용하여 K-PaaS AP min 설치 시, Operation 파일을 제거하고 설치한다.
</td>
<td>Requires operation file: use-haproxy-public-network.yml <br>
    Requires value :  -v haproxy_private_ip
</td>
</tr>
<tr>
<td>operations/use-haproxy-public-network.yml</td>
<td>HAProxy Public Network 설정 <br>
    - IaaS에서 제공하는 LB를 사용하여 K-PaaS AP min 설치 시, Operation 파일을 제거하고 설치한다.
</td>
<td>Requires: use-haproxy.yml <br>
    Requires Value :  <br>
    -v haproxy_public_ip <br>
    -v haproxy_public_network_name
</td>
</tr>
<tr>
<td>operations/min-use-router-public-network.yml</td>
<td>router를 외부 접근을 가능하게 수정한다.</td>
<td>4VMs 배포시 사용</td>
</tr>
<tr>
<td>operations/min-create-vm-singleton-blobstore.yml</td>
<td>4VMs에서 사용되는 database의 singleton-blobstore를 단일 VM으로 배포한다.</td>
<td>Requires: min-use-haproxy.yml <br>
7VMs 배포시 사용 <br>
Requires operation file: min-option-network-and-deployment.yml</td>
</tr>
<tr>
<td>operations/min-create-vm-tcp-router.yml</td>
<td>4VMs에서 사용되는 router의 tcp-router를 단일 VM으로 배포한다.</td>
<td>7VMs 배포시 사용</td>
</tr>
<tr>
<td>operations/min-cce.yml</td>
<td>CCE 조치를 적용하여 설치한다.</td>
<td></td>
</tr>
</table>

<br>

### <div id='2.6.3'/>2.6.3.   K-PaaS AP min 설치 Shell Scripts
min-ap-deployment.yml 파일은 K-PaaS AP min를 배포하는 Manifest 파일이며, K-PaaS AP min VM에 대한 설치 정의를 하게 된다.  

**※ K-PaaS AP min 설치 시 명령어는 BOSH deploy를 사용한다. (IaaS 환경에 따라 Option이 다름)**

K-PaaS AP min 배포 BOSH 명령어 예시

```
$ bosh -e ${BOSH_ENVIRONMENT} -d ap deploy min-ap-deployment.yml
```

K-PaaS AP min 배포 시, 설치 Option을 추가해야 한다. 설치 Option에 대한 설명은 아래와 같다.

<table>
<tr>
<td>-e</td>
<td>BOSH Director 명</td>
</tr>
<tr>
<td>-d</td>
<td>Deployment 명 (기본값 ap, 수정 시 다른 K-PaaS 서비스에 영향을 준다.)</td>
</tr>   
<tr>
<td>-o</td>
<td>K-PaaS 설치 시 적용하는 Option 파일로 IaaS별 속성, Haproxy 사용 여부, Database 설정 기능을 제공한다.
</td>
</tr>
<tr>
<td>-v</td>
<td>K-PaaS 설치 시 적용하는 변수 또는 Option 파일에 변수를 설정할 경우 사용한다. <br> Option 파일 속성에 따라 필수 또는 선택 항목으로 나뉜다.</td>
</tr>
<tr>
<td>-l, --var-file</td>
<td>YAML파일에 작성한 변수를 읽어올때 사용한다.</td>
</tr>
</table>

- AWS 환경 4vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-aws-4vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-aws.yml \					# AWS 설정
	-o operations/min-use-router-public-network.yml \		# Router 외부 접근 설정
	-o operations/min-use-router-public-network-aws.yml \
        -o operations/min-use-postgres.yml \				# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \		# Rename Network and Deployment
	-o operations/min-cce.yml \					# CCE 조치 적용
        -l min-vars.yml \						# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml					# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- AWS 환경 7vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-aws-7vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-aws.yml \					# AWS 설정
        -o operations/min-create-vm-singleton-blobstore.yml \		# singleton-blobstore VM 배포
        -o operations/min-create-vm-tcp-router.yml \			# tcp-router VM 배포
        -o operations/min-use-haproxy.yml \				# HAProxy 적용
        -o operations/use-haproxy-public-network.yml \			# HAProxy Public Network 적용
        -o operations/min-use-postgres.yml \				# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \		# Rename Network and Deployment
        -o operations/min-option-network-and-deployment.yml \		# singleton-blobstore Rename Network and Deployment
	-o operations/min-cce.yml \					# CCE 조치 설정
        -l min-vars.yml \						# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml					# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- Openstack 환경 4vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-openstack-4vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-openstack.yml \					# Openstack 설정
	-o operations/min-use-router-public-network.yml \			# Router 외부 접근 설정
        -o operations/min-use-postgres.yml \					# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \			# Rename Network and Deployment
	-o operations/min-cce.yml \						# CCE 조치 적용
        -l min-vars.yml \							# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml						# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- Openstack 환경 7vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-openstack-7vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-openstack.yml \					# Openstack 설정
        -o operations/min-create-vm-singleton-blobstore.yml \		# singleton-blobstore VM 배포
        -o operations/min-create-vm-tcp-router.yml \			# tcp-router VM 배포
        -o operations/min-use-haproxy.yml \				# HAProxy 적용
        -o operations/use-haproxy-public-network.yml \			# HAProxy Public Network 적용
        -o operations/min-use-postgres.yml \				# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \		# Rename Network and Deployment
        -o operations/min-option-network-and-deployment.yml \		# singleton-blobstore Rename Network and Deployment
	-o operations/min-cce.yml \					# CCE 조치 설정
        -l min-vars.yml \						# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml					# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- vSphere 환경 4vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-vsphere-4vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
	-o operations/min-use-router-public-network.yml \			# Router 외부 접근 설정
        -o operations/min-use-router-public-network-vsphere.yml \
        -o operations/min-use-postgres.yml \					# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \			# Rename Network and Deployment
	-o operations/min-cce.yml \						# CCE 조치 적용
        -l min-vars.yml \							# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml						# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- vSphere 환경 7vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-vsphere-7vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-create-vm-singleton-blobstore.yml \		# singleton-blobstore VM 배포
        -o operations/min-create-vm-tcp-router.yml \			# tcp-router VM 배포
        -o operations/min-use-haproxy.yml \				# HAProxy 적용
        -o operations/use-haproxy-public-network-vsphere.yml \			# HAProxy Public Network 적용
        -o operations/min-use-postgres.yml \				# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \		# Rename Network and Deployment
        -o operations/min-option-network-and-deployment.yml \		# singleton-blobstore Rename Network and Deployment
	-o operations/min-cce.yml \					# CCE 조치 설정
        -l min-vars.yml \						# AP-min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml					# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- Azure 환경 4vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-azure-4vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-azure.yml \					        # Azure 설정
	-o operations/min-use-router-public-network.yml \			# Router 외부 접근 설정
        -o operations/min-use-postgres.yml \					# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \			# Rename Network and Deployment
	-o operations/min-cce.yml \						# CCE 조치 적용
        -l min-vars.yml \							# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml						# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- Azure 환경 7vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-azure-7vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-azure.yml \					# Azure 설정
        -o operations/min-create-vm-singleton-blobstore.yml \		# singleton-blobstore VM 배포
        -o operations/min-create-vm-tcp-router.yml \			# tcp-router VM 배포
        -o operations/min-use-haproxy.yml \				# HAProxy 적용
        -o operations/use-haproxy-public-network.yml \			# HAProxy Public Network 적용
        -o operations/min-use-postgres.yml \				# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \		# Rename Network and Deployment
        -o operations/min-option-network-and-deployment.yml \		# singleton-blobstore Rename Network and Deployment
	-o operations/min-cce.yml \					# CCE 조치 설정
        -l min-vars.yml \						# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml					# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- GCP 환경 4vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-gcp-4vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
	-o operations/min-use-router-public-network.yml \			# Router 외부 접근 설정
        -o operations/min-use-postgres.yml \					# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \			# Rename Network and Deployment
	-o operations/min-cce.yml \						# CCE 조치 적용
        -l min-vars.yml \							# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml						# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- GCP 환경 7vm 설치 시

```
$ vi ~/workspace/ap-deployment-min/ap/deploy-gcp-7vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-create-vm-singleton-blobstore.yml \		# singleton-blobstore VM 배포
        -o operations/min-create-vm-tcp-router.yml \			# tcp-router VM 배포
        -o operations/min-use-haproxy.yml \				# HAProxy 적용
        -o operations/use-haproxy-public-network.yml \			# HAProxy Public Network 적용
        -o operations/min-use-postgres.yml \				# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \		# Rename Network and Deployment
        -o operations/min-option-network-and-deployment.yml \		# singleton-blobstore Rename Network and Deployment
	-o operations/min-cce.yml \					# CCE 조치 설정
        -l min-vars.yml \						# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml					# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- Shell script 파일에 실행 권한 부여

```
$ chmod +x ~/workspace/ap-deployment-min/ap/*.sh
```

<br>

## <div id='2.7'/>2.7.  K-PaaS AP min 설치
- 서버 환경에 맞추어 common_vars.yml와 min-vars.yml를 수정 한 뒤, Deploy 스크립트 파일의 설정을 수정한다.

- 4VM 배포 시
```
$ vi ~/workspace/ap-deployment-min/ap/deploy-aws-4vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-aws.yml \					# AWS 설정
	-o operations/min-use-router-public-network.yml \		# Router 외부 접근 설정
	-o operations/min-use-router-public-network-aws.yml \
        -o operations/min-use-postgres.yml \				# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \		# Rename Network and Deployment
	-o operations/min-cce.yml \					# CCE 조치 적용
        -l min-vars.yml \						# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml					# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```


- 7VM 배포 시
```
$ vi ~/workspace/ap-deployment-min/ap/deploy-aws-7vms.sh

BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"                   # bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

bosh -e ${BOSH_ENVIRONMENT} -d ap -n deploy min-ap-deployment.yml \	# AP min Manifest File
        -o operations/min-aws.yml \					# AWS 설정
        -o operations/min-create-vm-singleton-blobstore.yml \		# singleton-blobstore VM 배포
        -o operations/min-create-vm-tcp-router.yml \			# tcp-router VM 배포
        -o operations/min-use-haproxy.yml \				# HAProxy 적용
        -o operations/use-haproxy-public-network.yml \			# HAProxy Public Network 적용
        -o operations/min-use-postgres.yml \				# Database Type 설정 (3.5버전 이하에서 Migration 시 필수)
        -o operations/min-rename-network-and-deployment.yml \		# Rename Network and Deployment
        -o operations/min-option-network-and-deployment.yml \		# singleton-blobstore Rename Network and Deployment
	-o operations/min-cce.yml \					# CCE 조치 설정
        -l min-vars.yml \						# AP min 설치시 적용하는 변수 설정 파일
        -l ../../common/common_vars.yml					# AP 및 각종 Service 설치시 적용하는 공통 변수 설정 파일
```

- K-PaaS AP min 설치 시 Shell Script 파일 실행 (BOSH 로그인 필요)

```
$ cd ~/workspace/ap-deployment-min/ap
$ ./deploy-{IaaS}-{VMs_Number}.sh
```

- K-PaaS AP min 설치 확인

> $ bosh -e ${BOSH_ENVIRONMENT} vms -d ap

```
ubuntu@inception:~$ bosh -e micro-bosh vms -d ap
Using environment '10.0.1.6' as client 'admin'

Task 134. Done

Deployment 'ap'

Instance                                       Process State  AZ  IPs          VM CID               VM Type             Active  Stemcell  
compute/e154dcdc-a2c1-4a85-86b7-607a02a30acf   running        z1  10.0.31.235  i-0f92f55575bf2567e  small-highmem-16GB  true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
control/a18f5e97-098c-47ab-9147-77f594571bd6   running        z1  10.0.31.234  i-053cd8f71d99f1a15  small-highmem-16GB  true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
database/7ea28d82-5d5b-471f-bde6-a65d4809062e  running        z1  10.0.31.233  i-0b2e54deaf0734f59  small               true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
router/c01b1aa4-43c9-42f6-9003-cf8f8664d142    running        z7  10.0.30.204  i-0a449def3351877b3  minimal             true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
                                                                  54.180.53.80                                                 

4 vms

Succeeded
```
```
ubuntu@inception:~$ bosh -e micro-bosh vms -d ap
Using environment '10.0.1.6' as client 'admin'

Task 134. Done

Deployment 'ap'

Instance                                                  Process State  AZ  IPs             VM CID               VM Type             Active  Stemcell  
compute/c3f53aed-469f-47ab-aa9b-94be30ca3687              running        z1  10.0.21.156     i-0617a496567bd859e  small-highmem-16GB  true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
control/acd880a6-b309-452e-b996-0ef4252f8dd3              running        z1  10.0.21.153     i-0d9fbf3f662dec9a0  small-highmem-16GB  true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
database/c92fd45f-1165-4d71-8df4-9b4270abdd0a             running        z1  10.0.21.151     i-0ead4f61c9be951b9  small               true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
haproxy/5ccc73dd-cf7e-4f4c-a204-1e933eddfcf8              running        z7  10.0.20.151     i-02e5277d6fd829f34  minimal             true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
                                                                             54.180.53.80                                             
router/4f58af5a-529c-41f7-866c-e2327978ea99               running        z1  10.0.21.154     i-0b5f2d42d2c2b9d06  minimal             true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
singleton-blobstore/5ed376fe-1d84-45c8-a6e8-f938b7320a36  running        z1  10.0.21.152     i-08a432269ffb76663  small               true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 
tcp-router/f8fe5974-8340-4d16-ae02-0b7150828388           running        z1  10.0.21.155     i-04a845c8e7fc7cfb4  minimal             true    bosh-aws-xen-hvm-ubuntu-jammy-go_agent/1.260 


7 vms

Succeeded
```

<br>

## <div id='2.8'/>2.8.  K-PaaS AP min 로그인

CF CLI를 설치하고 K-PaaS AP min에 로그인한다.  
CF CLI는 v6과 v7중 선택해서 설치를 한다.  
CF API는 K-PaaS AP min 배포 시 지정했던 System Domain 명을 사용한다.

- CF CLI v6 설치

```
$ wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
$ echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
$ sudo apt update
$ sudo apt install cf-cli -y
$ cf --version
```

- CF CLI v7 설치 (K-PaaS AP min 5.1.0 이상)

```
$ wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
$ echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
$ sudo apt update
$ sudo apt install cf7-cli -y
$ cf --version
```

- CF API URL 설정

> $ cf api api.{system_domain} --skip-ssl-validation

```
ubuntu@inception:~$ cf api api.54.180.53.80.nip.io --skip-ssl-validation
Setting api endpoint to api.54.180.53.80.nip.io...
OK

api endpoint:   https://api.54.180.53.80.nip.io
api version:    3.87.0
```

- K-PaaS AP min 로그인

> $ cf login

```
ubuntu@inception:~$ cf login
API endpoint: https://api.54.180.53.80.nip.io

Email> admin

Password>
Authenticating...
OK

Select an org (or press enter to skip):
```

[FLAVOR_Image]:./images/ap/aws-vmtype.png
[UAA_Login_Create_Account]:./images/ap/uaa-login-2.png
[UAA_Login_Reset_Password]:./images/ap/uaa-login.png

### [Index](https://github.com/K-PaaS/Guide/blob/master/README.md) > [AP Install](../README.md) > K-PaaS AP - min
