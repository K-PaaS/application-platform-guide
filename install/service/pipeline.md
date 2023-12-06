### [Index](https://github.com/K-PaaS/Guide/blob/master/README.md) > [AP Install](../README.md) > Pipeline Service

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [참고자료](#1.3)  
  
2. [배포 파이프라인 서비스 설치](#2)  
 2.1. [Prerequisite](#2.1)  
 2.2. [Stemcell 확인](#2.2)  
 2.3. [Deployment 다운로드](#2.3)  
 2.4. [Deployment 파일 수정](#2.4)  
 2.5. [서비스 설치](#2.5)    
 2.6. [서비스 설치 확인](#2.6)  
 2.7. [[선택] 확장 언어 설치](#2.7)  
    
3. [배포 파이프라인 서비스 관리 및 신청](#3)  
 3.1. [서비스 브로커 등록](#3.1)  
 3.2. [UAA Client 등록](#3.2)  
 3.3. [Java Offline Buildpack 등록](#3.3)  
 3.4. [서비스 신청](#3.4)  
　3.4.1. [서비스 신청 - 포탈](#3.4.1)   
　3.4.2. [서비스 신청 - CLI](#3.4.2)   


## <div id='1'/> 1. 문서 개요

### <div id='1.1'/> 1.1. 목적
본 문서(배포 파이프라인 서비스팩 설치 가이드)는 K-PaaS에서 제공되는 서비스팩인 배포 파이프라인 서비스팩을 Bosh를 이용하여 설치 하는 방법을 기술하였다.  

### <div id='1.2'/> 1.2. 범위
설치 범위는 배포 파이프라인 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다.


### <div id='1.3'/> 1.3. 참고 자료
BOSH Document: [http://bosh.io](http://bosh.io)  
Cloud Foundry Document: [https://docs.cloudfoundry.org](https://docs.cloudfoundry.org)  

## <div id='2'/> 2. 배포 파이프라인 서비스 설치

### <div id='2.1'/> 2.1. Prerequisite

본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.  
서비스팩 설치를 위해서는 먼저 BOSH CLI v2 가 설치 되어 있어야 하고 BOSH 에 로그인이 되어 있어야 한다.  
BOSH CLI v2 가 설치 되어 있지 않을 경우 먼저 BOSH2.0 설치 가이드 문서를 참고 하여 BOSH CLI v2를 설치를 하고 사용법을 숙지 해야 한다.  
UAA client가 설치 되어 있지 않을 경우 UAA client의 설치가 필요하다.

- UAA client 설치 (BOSH Dependency 설치 필요)
```
$ sudo gem install cf-uaac
$ uaac -v
```

### <div id='2.2'/> 2.2. Stemcell 확인

Stemcell 목록을 확인하여 서비스 설치에 필요한 Stemcell이 업로드 되어 있는 것을 확인한다.  
본 가이드의 Stemcell은 ubuntu-jammy 1.260를 사용한다.  

> $ bosh -e ${BOSH_ENVIRONMENT} stemcells

```
Using environment '10.0.1.6' as client 'admin'

Name                                       Version   OS             CPI  CID  
bosh-openstack-kvm-ubuntu-jammy-go_agent  1.260      ubuntu-jammy  -    ce507ae4-aca6-4a6d-b7c7-220e3f4aaa7d

(*) Currently deployed

1 stemcells

Succeeded
```

만약 해당 Stemcell이 업로드 되어 있지 않다면 [bosh.io 스템셀](https://bosh.io/stemcells/) 에서 해당되는 IaaS환경과 버전에 해당되는 스템셀 링크를 복사 후 다음과 같은 명령어를 실행한다.

```
# Stemcell 업로드 명령어 예제
$ bosh -e ${BOSH_ENVIRONMENT} upload-stemcell -n {STEMCELL_URL}
```

### <div id='2.3'/> 2.3. Deployment 다운로드

서비스 설치에 필요한 Deployment를 Git Repository에서 받아 서비스 설치 작업 경로로 위치시킨다.  

- Service Deployment Git Repository URL : https://github.com/K-PaaS/service-deployment/tree/v5.1.26

```
# Deployment 다운로드 파일 위치 경로 생성 및 설치 경로 이동
$ mkdir -p ~/workspace
$ cd ~/workspace

# Deployment 파일 다운로드
$ git clone https://github.com/K-PaaS/service-deployment.git -b v5.1.26

# common_vars.yml 파일 다운로드(common_vars.yml가 존재하지 않는다면 다운로드)
$ git clone https://github.com/K-PaaS/common.git
```

### <div id='2.4'/> 2.4. Deployment 파일 수정

BOSH Deployment manifest는 Components 요소 및 배포의 속성을 정의한 YAML 파일이다.  
Deployment 파일에서 사용하는 network, vm_type, disk_type 등은 Cloud config를 활용하고, 활용 방법은 K-PaaS AP 설치 가이드를 참고한다.  

- Cloud config 설정 내용을 확인한다.   

> $ bosh -e micro-bosh cloud-config   

```
Using environment '10.0.1.6' as client 'admin'

azs:
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z1
- cloud_properties:
    availability_zone: ap-northeast-2a
  name: z2

... ((생략)) ...

disk_types:
- disk_size: 1024
  name: default
- disk_size: 1024
  name: 1GB

... ((생략)) ...

networks:
- name: default
  subnets:
  - az: z1
    cloud_properties:
      security_groups: ap-security-group
      subnet: subnet-00000000000000000
    dns:
    - 8.8.8.8
    gateway: 10.0.1.1
    range: 10.0.1.0/24
    reserved:
    - 10.0.1.2 - 10.0.1.9
    static:
    - 10.0.1.10 - 10.0.1.120

... ((생략)) ...

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

Succeeded
```

- common_vars.yml을 서버 환경에 맞게 수정한다. 
- 배포 파이프라인에서 사용하는 변수는 system_domain 이다.

> $ vi ~/workspace/common/common_vars.yml
```
... ((생략)) ...

system_domain: "61.252.53.246.nip.io"		# Domain (nip.io를 사용하는 경우 HAProxy Public IP와 동일)

... ((생략)) ...

```

- Deployment YAML에서 사용하는 변수 파일을 서버 환경에 맞게 수정한다.

> $ vi ~/workspace/service-deployment/pipeline-service/vars.yml

```
# STEMCELL
stemcell_os: "ubuntu-jammy"                                     # stemcell os
stemcell_version: "1.260"                                       # stemcell version

# NETWORK
private_networks_name: "default"                                 # private network name
public_networks_name: "vip"                                      # public network name

# UAAC
pipeline_clinet_id: "pipeclient"                                 # pipeline client id for UAA
pipeline_clinet_secret: "clientsecret"                           # pipeline client password for UAA

# MARIADB
mariadb_port: "13306"                                            # mariadb database port (default : 13306) -- Do Not Use "3306"
mariadb_azs: [z5]                                                # mariadb azs
mariadb_instances: 1                                             # mariadb instances
mariadb_persistent_disk_type: "2GB"                              # mariadb persistent disk type
mariadb_vm_type: "small"                                         # mariadb vm type (e.g. small/medium/large etc)
mariadb_internal_static_ips: "<MARIADB_PRIVATE_IP>"              # mariadb's private IP (e.g. "10.0.161.30")
mariadb_admin_password: "<MARIADB_ADMIN_PASSWORD>"               # mariadb admin password (e.g. "admin!Service")

# POSTGRES
postgres_port: "5532"                                            # postgresql port (default : 5532) -- Do Not Use "5432"
postgres_azs: [z5]                                               # postgresql azs
postgres_instances: 1                                            # postgresql instances
postgres_persistent_disk_type: "2GB"                             # postgresql persistent disk type
postgres_vm_type: "small"                                        # postgresql vm type
postgres_internal_static_ips: "<POSTGRES_PRIVATE_IP>"            # postgresql's private IP (e.g. "10.0.161.31")
postgres_datasource_username: "<POSTGRES_ADMIN_USERNAME>"        # postgresql username (e.g. sonar)
postgres_datasource_password: "<POSTGRES_ADMIN_PASSWORD>"        # postgresql password (e.g. sonar@2020)

# INSPECTION_SERVER
inspection_azs: [z5]                                             # inspection server(SonarQube) azs
inspection_instances: 1                                          # inspection server(SonarQube) instances 
inspection_vm_type: "small"                                      # inspection server(SonarQube) vm type
inspection_internal_static_ips: "<INSPECTION_SERVER_PRIVATE_IP>" # inspection server(SonarQube)'s private IP (e.g. "10.0.161.32")

# HAPROXY
haproxy_azs: [z7]                                                # haproxy azs
haproxy_instances: 1                                             # haproxy instances
haproxy_vm_type: "small"                                         # haproxy vm type
haproxy_internal_static_ips: "<HAPROXY_PRIVATE_IP>"              # haproxy's private IP (e.g. "10.0.0.11")
haproxy_public_static_ips: "<HAPROXY_PUBLIC_IP>"                 # haproxy's public IP
```

### <div id='2.5'/> 2.5. 서비스 설치

- 서버 환경에 맞추어 Deploy 스크립트 파일의 VARIABLES 설정을 수정하고, Option file을 추가할지 선택한다.  

> $ vi ~/workspace/service-deployment/pipeline-service/deploy.sh

```
#!/bin/bash

# VARIABLES
COMMON_VARS_PATH="<COMMON_VARS_FILE_PATH>"	# common_vars.yml File Path (e.g. ../../common/common_vars.yml)
CURRENT_IAAS="${CURRENT_IAAS}"			# IaaS Information (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 aws/azure/gcp/openstack/vsphere 입력)
BOSH_ENVIRONMENT="${BOSH_ENVIRONMENT}"		# bosh director alias name (K-PaaS에서 제공되는 create-bosh-login.sh 미 사용시 bosh envs에서 이름을 확인하여 입력)

# DEPLOY
bosh -e ${BOSH_ENVIRONMENT} -n -d pipeline-service deploy --no-redact pipeline-service.yml \
    -o operations/${CURRENT_IAAS}-network.yml \
    -o operations/cce.yml \
    -l ${COMMON_VARS_PATH} \
    -l vars.yml
```

- 서비스를 설치한다.  
```
$ cd ~/workspace/service-deployment/pipeline-service  
$ sh ./deploy.sh  
```


### <div id='2.6'/> 2.6. 서비스 설치 확인

설치 된 서비스를 확인한다.  

> $ bosh -e micro-bosh -d pipeline-service vms  

```
Using environment '10.0.1.6' as client 'admin'

Task 296077. Done

Deployment 'pipeline-service'

Instance                                                                   Process State  AZ  IPs            VM CID                                VM Type  Active  Stemcell  
binary_storage/63b0c3de-0037-46c7-add7-c7fe54a9ac6c                        running        z5  10.0.161.17    28b7e75b-6fb4-4e3a-8a90-68d20e934441  small    true    -  
ci_server/48d7ffb1-9ac2-42af-915c-9adc7a215656                             running        z5  10.0.161.15    9694f782-acc2-4958-946c-1a21010c6325  small    true    -  
ci_server/de71d530-6b95-482e-a419-32e3c1e64f21                             running        z5  10.0.161.16    c7858b61-2dc1-4878-9e21-5ff5cd07d47b  small    true    -  
delivery-pipeline-api/ea2486ff-6477-4899-9e95-7370ed27efbe                 running        z5  10.0.161.21    ca7c0346-b086-4948-9da1-5410c5eec778  small    true    -  
delivery-pipeline-binary-storage-api/9d70363c-7e42-4d67-a452-b50a21a4e373  running        z5  10.0.161.20    fbc6441e-73e1-41c8-a5f4-c5fc00336936  small    true    -  
delivery-pipeline-common-api/87ce092c-2fc6-4b1b-9921-78a73e191c9e          running        z5  10.0.161.18    23e8e141-c869-449c-873e-f48a49252521  small    true    -  
delivery-pipeline-inspection-api/b8f8d86a-443a-482e-a668-b94624a882fb      running        z5  10.0.161.19    88425e31-68a2-42cd-9431-5a7cc372b9e6  small    true    -  
delivery-pipeline-scheduler/d2c02e3c-e545-47e1-a98a-d81de281a166           running        z5  10.0.161.24    8401e44a-16d5-4639-990c-876655500773  small    true    -  
delivery-pipeline-service-broker/151af074-db1f-4650-b789-477e69c51016      running        z5  10.0.161.22    d1f474ed-48cf-43e3-b7e8-a4cb05c00dbf  small    true    -  
delivery-pipeline-ui/d3ff6d00-93b9-4481-ac07-66cea65322f9                  running        z5  10.0.161.23    c65f56c5-13f8-4c12-8726-295a384b0a63  small    true    -  
haproxy/5a35c6b2-ac18-45cd-9705-a7a401721989                               running        z5  10.0.161.14    a1d70ef6-064d-46b1-b8fc-83ffb08ad82f  small    true    -  
                                                                                              101.55.50.208                                                           
inspection/0a91abe1-b888-4f86-a082-efd6aa9936de                            running        z5  10.0.161.13    5c7a1f2e-b406-44d2-b5fe-2f694c36036c  small    true    -  
mariadb/521553a6-4145-4c5c-9d8f-475db29c5807                               running        z5  10.0.161.11    5476fe5d-a4b2-4b25-8db7-00a0afa30186  small    true    -  
postgres/6a8a4d71-e46f-49ca-b992-407441a90965                              running        z5  10.0.161.12    c87ffcd0-599e-4f07-8d03-3b52d7ae3762  small    true    -  

14 vms

Succeeded
```

### <div id='2.7'/> 2.7. [선택] 확장 언어 설치

- 확장 언어 스크립트를 ci_server 에 복사하고 실행한다.   
```
$ cd ~/workspace/service-deployment/pipeline-service/scripts/
$ bosh -d pipeline-service scp php-ci-server-script.sh ci_server/0:/tmp/
$ bosh -d pipeline-service ssh ci_server/0 
$ sudo su
$ mv /tmp/php-ci-server-script.sh /var/vcap/
$ sh /var/vcap/php-ci-server-script.sh
```


- 서버 환경에 맞추어 스크립트 파일의 계정 정보를 수정한다. 

> $ vi ~/workspace/service-deployment/pipeline-service/scripts/php-mariadb-script.sh

```
#!/bin/bash
mysql_path=/var/vcap/store/mariadb-10.5.15-linux-x86_64/bin/mysql #경로 확인 및 수정
mysql_port='<mysql_port>'           #mariadb port : (e.g. 13306)
mysql_user='<mysql_user>'           #mariadb user : (e.g. root)
mysql_password='<mysql_password>'   #mariadb password 
# insert code table
${mysql_path} -u${mysql_user} -p${mysql_password} -P${mysql_port} -Ddelivery_pipeline  << "EOF"
insert  into `code`(`code_type`,`code_group`,`code_name`,`code_value`,`code_order`) values ('language_type', NULL,'PHP', 'PHP', 2);
insert  into `code`(`code_type`,`code_group`,`code_name`,`code_value`,`code_order`) values ('language_type_version', 'PHP','PHP-7.4', 'PHP-7.4', 2);
insert  into `code`(`code_type`,`code_group`,`code_name`,`code_value`,`code_order`) values ('builder_type', 'PHP','COMPOSER', 'COMPOSER', 2);
insert  into `code`(`code_type`,`code_group`,`code_name`,`code_value`,`code_order`) values ('builder_type_version', 'COMPOSER','COMPOSER-2', 'COMPOSER-2.1.14', 2);
EOF

${mysql_path} -u${mysql_user} -p${mysql_password} -P${mysql_port} -Ddelivery_pipeline -e "select * from code;"

echo 'php data insert complete'
```

- 확장 언어 스크립트를 mariadb 에 복사하고 실행하여, 파이프라인 서비스에 적용한다.  
```
$ cd ~/workspace/service-deployment/pipeline-service/scripts/
$ bosh -d pipeline-service scp php-mariadb-script.sh mariadb/0:/tmp/
$ bosh -d pipeline-service ssh mariadb/0 
$ sudo su
$ mv /tmp/php-mariadb-script.sh /var/vcap/
$ sh /var/vcap/php-mariadb-script.sh
```

## <div id='3'/> 3. 배포 파이프라인 서비스 관리 및 신청
K-PaaS AP 운영자 포탈을 통해 배포파이프라인 서비스를 등록 및 공개하면, K-PaaS AP 사용자 포탈을 통해 서비스를 신청 하여 사용할 수 있다.

### <div id='3.1'/> 3.1. 서비스 브로커 등록

배포 파이프라인 서비스팩 배포가 완료되었으면 K-PaaS AP 포탈에서 서비스 팩을 사용하기 위해서 먼저 배포 파이프라인 서비스 브로커를 등록해 주어야 한다.
서비스 브로커 등록 시 개방형 클라우드 플랫폼에서 서비스 브로커를 등록할 수 있는 사용자로 로그인이 되어있어야 한다.

- 서비스 브로커 목록을 확인한다.

> $ cf service-brokers   
```  
Getting service brokers as admin...
No service brokers found
```  

- 서비스 브로커 등록 명령어
```
cf create-service-broker [SERVICE_BROKER] [USERNAME] [PASSWORD] [SERVICE_BROKER_URL]

[SERVICE_BROKER] : 서비스 브로커 명
[USERNAME] / [PASSWORD] : 서비스 브로커에 접근할 수 있는 사용자 ID / PASSWORD
[SERVICE_BROKER_URL] : 서비스 브로커 접근 URL
```

- 배포 파이프라인 서비스 브로커를 등록한다.

> $ cf create-service-broker delivery-pipeline admin cloudfoundry http://<delivery-pipeline-service-broker_ip>:8080   
```  
$ cf create-service-broker delivery-pipeline-broker admin cloudfoundry http://10.0.161.22:8080
Creating service broker delivery-pipeline-broker as admin...
OK
```  

- 등록된 배포 파이프라인 서비스 브로커를 확인한다.

> $ cf service-brokers  
```  
Getting service brokers as admin...

name                           url
delivery-pipeline-broker       http://10.0.161.22:8080
```  

- 접근 가능한 서비스 목록을 확인한다.
>$ cf service-access  

```
Getting service access as admin...

broker: delivery-pipeline
   offering   plan                 access   orgs
   pipeline   pipeline-dedicated   none     
   pipeline   pipeline-shared      none     
```
서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.

- 특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시 확인한다. (전체 조직)  
> $ cf enable-service-access pipeline   
```
Enabling access to all plans of service offering pipeline for all orgs as admin...
OK
```
> $ cf service-access   
```                          
Getting service access as admin...

broker: delivery-pipeline
   offering   plan                 access   orgs
   pipeline   pipeline-dedicated   all      
   pipeline   pipeline-shared      all  
```

### <div id='3.2'/> 3.2. UAAC Client 등록
UAAC Client 계정 등록 절차에 대한 순서를 확인한다.

- 배포 파이프라인 UAAC Client를 등록한다.
```
### uaac client add 설명
uaac client add {클라이언트 명} -s {클라이언트 비밀번호} --redirect_uri{대시보드 URL} --scope {퍼미션 범위} --authorized_grant_types {권한 타입} --authorities={권한 퍼미션} --autoapprove={자동승인권한}  
클라이언트 명 : uaac 클라이언트 명 (pipeclient)  
클라이언트 비밀번호 : uaac 클라이언트 비밀번호  
대시보드 URL: 성공적으로 리다이렉션 할 대시보드 URL   
퍼미션 범위: 클라이언트가 사용자를 대신하여 얻을 수있는 허용 범위 목록  
권한 타입 : 서비스팩이 제공하는 API를 사용할 수 있는 권한 목록  
권한 퍼미션 : 클라이언트에 부여 된 권한 목록  
자동승인권한: 사용자 승인이 필요하지 않은 권한 목록  
```

>$ uaac client add pipeclient -s clientsecret --redirect_uri "[DASHBOARD_URL]" \  
>--scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \  
>--authorized_grant_types "authorization_code , client_credentials , refresh_token" \  
>--authorities="uaa.resource" \  
>--autoapprove="openid , cloud_controller_service_permissions.read"  

```  
### uaac endpoint 설정
$ uaac target https://uaa.<DOMAIN> --skip-ssl-validation

### target 확인
$ uaac target
Target: https://uaa.<DOMAIN>
Context: uaa_admin, from client uaa_admin

### uaac 로그인
$ uaac token client get <UAA_CLIENT_ADMIN_ID> -s <UAA_CLIENT_ADMIN_SECRET>

### 배포파이프라인 uaac client 등록
$ uaac client add pipeclient -s clientsecret --redirect_uri "http://101.55.50.208:8084 http://101.55.50.208:8084/dashboard" \
   --scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
   --authorized_grant_types "authorization_code , client_credentials , refresh_token" \
   --authorities="uaa.resource" \
   --autoapprove="openid , cloud_controller_service_permissions.read"
```  


### <div id='3.3'/> 3.3. Java Offline Buildpack 등록
배포 파이프라인 서비스 사용을 위해 Java Offline Buildpack을 등록한다.

- Java Offline Buildpack 다운로드 
> wget -O java-buildpack-offline-v4.37.zip https://nextcloud.k-paas.org/index.php/s/8rGJXEFa8odFDLk/download 

**buildpack 등록**  

> $ cf create-buildpack java_buildpack_offline java-buildpack-offline-v4.37.zip 3   
```
Creating buildpack java_buildpack_offline as admin...
OK

Uploading buildpack java_buildpack_offline as admin...
 794.99 MiB / 794.99 MiB [===================================================================================] 100.00% 7s
OK

Processing uploaded buildpack java_buildpack_offline...
OK
```

**buildpack 등록 확인**  

> $ cf buildpacks 
```
Getting buildpacks as admin...

position   name                     stack        enabled   locked   filename
1          java_buildpack           cflinuxfs3   true      false    java-buildpack-cflinuxfs3-v4.50.zip
2          staticfile_buildpack     cflinuxfs3   true      false    staticfile_buildpack-cflinuxfs3-v1.5.32.zip
3          java_buildpack_offline                true      false    java-buildpack-offline-v4.37.zip
4          ruby_buildpack           cflinuxfs3   true      false    ruby_buildpack-cflinuxfs3-v1.8.56.zip
5          dotnet_core_buildpack    cflinuxfs3   true      false    dotnet-core_buildpack-cflinuxfs3-v2.3.44.zip
6          nodejs_buildpack         cflinuxfs3   true      false    nodejs_buildpack-cflinuxfs3-v1.7.72.zip
7          go_buildpack             cflinuxfs3   true      false    go_buildpack-cflinuxfs3-v1.9.48.zip
8          python_buildpack         cflinuxfs3   true      false    python_buildpack-cflinuxfs3-v1.7.56.zip
9          php_buildpack            cflinuxfs3   true      false    php_buildpack-cflinuxfs3-v4.4.64.zip
10         nginx_buildpack          cflinuxfs3   true      false    nginx_buildpack-cflinuxfs3-v1.1.41.zip
11         r_buildpack              cflinuxfs3   true      false    r_buildpack-cflinuxfs3-v1.1.31.zip
12         binary_buildpack         cflinuxfs3   true      false    binary_buildpack-cflinuxfs3-v1.0.45.zip
```
※ 참고 URL : https://github.com/cloudfoundry/java-buildpack  

  
### <div id='3.4'/> 3.4. 서비스 신청
#### <div id='3.4.1'/> 3.4.1. 서비스 신청 - 포탈
1. K-PaaS AP 운영자 포탈에 접속하여 로그인한다.
![3-1-1]

2. 로그인 후 서비스 관리 > 서비스 브로커 페이지에서 배포 파이프라인 서비스 브로커를 확인한다.
![3-1-2]

3. 서비스 관리 > 서비스 제어 페이지에서 배포 파이프라인 서비스 플랜 접근 가능 권한을 확인한다.
![3-1-3]

4. 운영관리 > 카탈로그 > 앱서비스 페이지를 확인하여 "파이프라인" 서비스 이름을 클릭한다.  
![3-2-1]

- 아래의 내용을 상세 페이지에 입력한다.

> ※ 카탈로그 관리 > 앱 서비스
> - 이름 : 파이프라인
> - 분류 :  개발 지원 도구
> - 서비스 : delivery-pipeline
> - 썸네일 : [배포 파이프라인 서비스 썸네일]
> - 문서 URL : https://github.com/K-PaaS/ap-pipeline-broker
> - 서비스 생성 파라미터 : owner
> - 앱 바인드 사용 : N
> - 공개 : Y
> - 대시보드 사용 : Y
> - 온디멘드 : N
> - 태그 : k-paas / tag6, free / tag2
> - 요약 : 개발용으로 만들어진 파이프라인
> - 설명 :
> 개발용으로 만들어진 파이프라인
> 배포 파이프라인 Server, 배포 파이프라인 서비스 브로커로 최소사항을 구성하였다.
  
![3-2-2]

- K-PaaS AP 사용자  포탈에 접속하여, 카탈로그를 통해 서비스를 신청한다.   

![003]

- 대시보드 URL을 통해 서비스에 접근한다.    

![004]  


#### <div id='3.4.2'/> 3.4.2. 서비스 신청 - CLI
CLI 를 통한 파이프라인 서비스 신청 방법을 설명한다.

- 서비스 인스턴스 신청 명령어
```
cf create-service [SERVICE] [PLAN] [SERVICE_INSTANCE]

[SERVICE] : Marketplace에서 보여지는 서비스 명
[PLAN] : 서비스에 대한 정책
[SERVICE_INSTANCE] : 생성할 서비스 인스턴스 이름
```

- 파이프라인 서비스를 신청한다. (K-PaaS AP user_id 설정)
> cf create-service pipeline pipeline-shared pipeline-service -c '{"owner":"{user_id}"}'  
```
Creating service instance pipeline-service in org system / space dev as admin...

Service instance pipeline-service created.
OK
```

- 서비스 상세의 대시보드 URL 정보를 확인하여 서비스에 접근한다.
> $ cf service pipeline-service
 ```
 ... (생략) ...
 Dashboard:        http://101.55.50.208:8084/dashboard/2bcbe484-e235-441e-bdb6-ef88f73cb516/
 Service broker:   delivery-pipeline-broker
 ... (생략) ...
 ```


[1-1-3]:./images/pipeline/Delivery_Pipeline_Architecture.jpg
[3-1-1]:./images/pipeline/adminPortal_login.png
[3-1-2]:./images/pipeline/adminPortal_serviceBroker.png
[3-1-3]:./images/pipeline/adminPortal_serviceControl.png
[3-2-1]:./images/pipeline/adminPortal_catalog.png
[3-2-2]:./images/pipeline/adminPortal_catalogDetail.png
[003]:./images/pipeline/userportal_catalog.PNG
[004]:./images/pipeline/userportal_dashboard.PNG



### [Index](https://github.com/K-PaaS/Guide/blob/master/README.md) > [AP Install](../README.md) > Pipeline Service
