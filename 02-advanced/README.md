---
title: 02. Advanced
duration_minutes: 120
last_updated: 2026-10-11
---

# 02. Advanced

## 개요와 학습 목표

이번 실습에서는 애저에서 가장 기본적인 형태의 웹 애플리케이션을 배포하는 방법에 대해 알아보도록 하겠습니다. 가상 머신에 웹 애플리케이션을 배포하고 웹 애플리케이션의 데이터를 관리하기 위한 SQL 데이터베이스를 구성해 보도록 하겠습니다. 부하 분산 장치의 프런트 엔드 IP를 웹 애플리케이션을 진입점으로 사용하고 가상 머신을 백 엔드 풀로 사용합니다. 이렇게 구성하면 웹 애플리케이션을 논리적으로 격리할 수 있으며, 추후 애플리케이션의 확장을 고려한 아키텍처를 설계할 수 있습니다.

![Advanced 실습의 웹 애플리케이션과 SQL Database 구성](assets/01-architecture.png)

## 사전 요구사항

- [사전 준비](../00-prerequisites/README.md)의 계정·구독 준비와 [Basic](../01-basic/README.md)의 VM·Bastion 사용 방법
- Azure Portal에서 가상 네트워크·Load Balancer·VM·Bastion·SQL Database·Private Endpoint를 만들 수 있는 권한
- VM에서 [TodoApp](https://github.com/Anna-Jeong-MS/TodoApp), Python 패키지와 Microsoft 패키지 저장소에 접근할 수 있는 환경

<!-- TODO: 저자 확인 — 구독 권한·리소스 한도·비용, VM 운영체제와 ODBC 드라이버 호환성 -->

## 소요 시간

예상 소요 시간은 약 120분이며, [워크샵 전체 agenda](../README.md#소요-시간)(4시간)를 기준으로 배분했습니다.

<!-- TODO: 저자 확인 — 실제 완주 시간과의 차이 -->

## 실습 단계

1. [가상 네트워크 만들기](#가상-네트워크-만들기)
2. [부하 분산 장치 만들기](#부하-분산-장치-만들기)
3. NAT 게이트웨이 만들기
4. [웹 애플리케이션 배포](#웹-애플리케이션-배포)
    - 가상 머신 만들기
    - 베스천 설정
    - 웹 애플리케이션 배포
    - 배포 테스트
5. [SQL 데이터베이스 연동](#sql-데이터베이스-연동)
    - SQL 데이터베이스 서버 만들기
    - SQL 데이터베이스 만들기
    - 데이터베이스 엔드포인트 만들기
    - SQL 데이터베이스 연동

<!-- TODO: 저자 확인 — NAT Gateway 단계 누락. 해당 단계의 필요성과 VM의 외부 연결 구성을 확인하되, 실습 절차를 임의로 추가하거나 목차에서 삭제하지 않음 -->

### 가상 네트워크 만들기

![가상 네트워크 만들기 단계 안내](assets/02-virtual-network-overview.png)

1. 왼쪽 상단 검색창에서 `가상 네트워크` 입력하여 가상 네트워크 화면으로 이동합니다.
2. `만들기` 버튼을 클릭합니다.
3. 아래와 같이 구성하고 `다음: IP 주소` 버튼을 클릭합니다.

![BasicVNet 가상 네트워크 기본 설정](assets/03-virtual-network-basics.png)

- 구독 : 생성한 구독 선택
- 리소스 그룹 : 새로 만들기 버튼 클릭 후, BasicWorkshopRG 입력하고 `확인` 버튼 클릭
- 이름 : BasicVNet
- 지역 : KoreaCentral

1. 아래와 같이 구성된 것을 확인하고 `검토 + 만들기` 버튼을 클릭합니다.

![가상 네트워크 구성 검토 화면](assets/04-virtual-network-review.png)

1. `만들기` 버튼을 클릭하여 가상 네트워크를 생성합니다.

### 부하 분산 장치 만들기

![Load Balancer 만들기 단계 안내](assets/05-load-balancer-overview.png)

1. 왼쪽 상단 검색창에서 `부하 분산 장치` 입력하여 부하 분산 장치 화면으로 이동합니다.
2. `만들기` 버튼을 클릭합니다.
3. 아래와 같이 구성합니다.

![Load Balancer 기본 설정](assets/06-load-balancer-basics.png)

- 구독 : 생성한 구독 선택
- 리소스 그룹 : BasicWorkshopRG
- 이름 : todoLB
- 지역 : Korea Central
- SKU : 표준
- 형식 : 공개
- 계층 : 지역
1. `다음: 프런트 엔드 IP 구성` 버튼을 클릭합니다.
2. `프런트 엔드 IP 구성 추가` 버튼을 클릭하고 아래와 같이 구성 후, `추가` 버튼을 클릭합니다.

![Load Balancer 프런트 엔드 IP 설정](assets/07-load-balancer-frontend.png)

- 이름 : todoPublic
- IP 버전 : IPv4
- IP 유형 : IP 주소
- 공용 IP 주소 : `새로 만들기` 버튼 클릭
    - 이름 : todoPublicIP

1. `다음: 백 엔드 풀` 버튼을 클릭하고 `백 엔드 풀 추가`를 클릭합니다.
2. 아래와 같이 구성하고 `저장` 버튼을 클릭합니다.

![TodoBackend 백 엔드 풀 설정](assets/08-load-balancer-backend.png)

- 이름 : TodoBackend
- 가상 네트워크 : BasicVNet(BasicWorkshopRG)
- 백 엔드 풀 구성 : IP 주소

1. `다음: 인바운드 규칙` 버튼을 클릭합니다.
2. 부하 분산 규칙을 생성하여 적용해 보도록 하겠습니다. `부하 분산 규칙 추가` 버튼을 클릭하고 다음과 같이 구성합니다.

![부하 분산 규칙 설정](assets/09-load-balancing-rule.png)

1. 상태 프로브는 백 엔드 풀의 정상 상태를 확인합니다. 상태 프로브에서 `새로 만들기`를 클릭하고 다음과 같이 설정하고 `확인` 버튼을 클릭합니다.

![백 엔드 상태 프로브 설정](assets/10-health-probe.png)

1. TodoRule이 정상적으로 추가된 것을 확인한 뒤, `검토 + 만들기` 버튼을 클릭하고 `만들기` 버튼을 클릭하여 부하 분산 장치를 생성합니다.

### 웹 애플리케이션 배포

#### 가상 머신 만들기

![가상 머신 만들기 단계 안내](assets/11-virtual-machine-overview.png)

1. 왼쪽 상단 검색창에서 `가상 머신` 입력하여 가상 머신 화면으로 이동합니다.
2. `만들기` 버튼을 클릭하고 Azure 가상 머신을 선택합니다.

![Azure 가상 머신 만들기 메뉴](assets/12-virtual-machine-create.png)

1. 아래와 같이 구성하고 `네트워킹` 탭을 선택하거나 `다음: 디스크, 다음: 네트워킹`을 차례로 선택합니다.

![TodoVM 가상 머신 기본 설정](assets/13-virtual-machine-basics.png)

- 구독 : 생성한 구독 선택
- 리소스 그룹 : BasicWorkshopRG
- 가상 머신 이름 : TodoVM
- 지역 : (Asia Pacific) Korea Central
1. 네트워킹 탭에서 다음 정보를 선택하거나 입력합니다.

![가상 머신의 applicationSubnet 연결 설정](assets/14-virtual-machine-networking.png)

- 가상 네트워크 : BasicVNet
- 서브넷 : `서브넷 구성 관리` 클릭
    - `서브넷` 버튼 클릭
    - 이름 : applicationSubnet
    - 서브넷 주소 범위 : 10.0.1.0/24
    - `저장` 버튼 클릭
    - 오른쪽 상단 `x` 버튼 클릭
    - `applicationSubnet` 선택
- 공용 IP : 없음
- NIC 네트워크 보안 그룹 : 고급

1. 부하 분산에서 `기존 부하 분산 솔루션 뒤에 이 가상 머신을 배치하겠습니까?` 체크박스를 클릭하고 아래와 같이 구성합니다.

![가상 머신과 TodoBackend 백 엔드 풀 연결 설정](assets/15-virtual-machine-load-balancer.png)

- 부하 분산 옵션 : Azure Load Balancer
- 부하 분산 장치 선택 : todoLB
- 백 엔드 풀 선택 : TodoBackend

1. `네트워크 보안 그룹 구성에서 새로 만들기를 클릭합니다. (중요)`
2. 인바운드 보안 규칙 추가를 클릭하고 다음과 같이 설정 후 추가 버튼을 클릭합니다.

![가상 머신의 인바운드 보안 규칙 설정](assets/16-network-security-rule.png)

> **참고**: Standard Load Balancer 뒤의 VM에는 인바운드 규칙이 최소 2개 필요합니다 — 상태 프로브 트래픽(소스: 서비스 태그 `AzureLoadBalancer`, 대상 포트 80)과 실제 서비스 트래픽(소스: `Internet` 또는 `Any`, 대상 포트 80)입니다. 상태 프로브 규칙만 추가하면 상태 프로브는 정상으로 표시되어도 Load Balancer의 공용 IP로 접속이 되지 않습니다. 위 스크린샷에 두 규칙이 모두 포함되어 있는지 확인하세요.

1. `검토 + 만들기` 버튼을 클릭하고 설정을 검토한 다음 `만들기`를 선택합니다. 새 키 쌍 생성 화면이 뜨면 `프라이빗 키 다운로드 및 리소스 만들기` 버튼을 클릭합니다.

![SSH 프라이빗 키 다운로드 화면](assets/17-ssh-key-download.png)

#### 베스천 배포하기

1. `가상 머신 화면`에서 TodoVM1을 클릭하고, 왼쪽 `베스천` 메뉴를 클릭합니다.
2. `기본값을 사용하여 Azure Bastion …` ****버튼을 클릭합니다.

![가상 머신의 Bastion 배포 화면](assets/18-bastion-deployment.png)

#### 웹 애플리케이션 배포

베스천을 사용하여 웹 애플리케이션을 배포해 보도록 하겠습니다.

1. 베스천이 생성되면 아래 정보를 입력하고 연결 버튼을 클릭합니다.

![Bastion 가상 머신 연결 화면](assets/19-bastion-connection.png)

1. 아래 명령어를 사용하여 깃허브에서 `TodoApp` 리포지토리를 복제합니다.

```bash
git clone https://github.com/Anna-Jeong-MS/TodoApp.git
```

1. `TodoApp`을 실행하기 위해 몇 가지 파이썬 종속성을 설치해야 합니다. 파이썬 종속성 설치하기 위해 파이썬 종속성을 설치하기 위한 도구인 `pip`를 설치합니다.

```bash
sudo apt update && sudo apt install python3-pip
```

1. 이제 `TodoApp`을 실행하기 위한 종속성을 다운로드 받습니다.

```bash
sudo pip install fastapi
sudo pip install "uvicorn[standard]"
sudo pip install aiofiles
sudo pip install pyodbc
```

1. SQL 데이터베이스 연결을 위해 ODBC 드라이버를 다운로드 합니다.

```bash
sudo su
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -

curl https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/prod.list > /etc/apt/sources.list.d/mssql-release.list

exit
sudo apt-get update
sudo ACCEPT_EULA=Y apt-get install -y msodbcsql17
```

1. 아래 명령어를 사용하여 애플리케이션을 실행합니다.

```bash
cd TodoApp
sudo uvicorn main:app --host 0.0.0.0 --port 80 > app.log 2>&1 &
```

*** 사용자 지정 데이터 사용**

```bash
#!/bin/sh
sudo apt-get update
sudo ACCEPT_EULA=Y apt-get install -y msodbcsql17
sudo apt-get install -y python3-pip
cd /home/azureuser
git clone https://github.com/Anna-Jeong-MS/TodoApp.git
cd TodoApp
sudo pip install -r requirements.txt
sudo uvicorn main:app --host 0.0.0.0 --port 80 > app.log 2>&1 &
```

#### 배포 테스트

1. `부하 분산 장치 화면`에서 생성한 `TodoLB`에서 왼쪽 `프런트 엔드 IP` 구성 메뉴를 클릭합니다.

![Load Balancer 프런트 엔드 IP 주소 확인 화면](assets/20-load-balancer-public-ip.png)

1. 새 탭에서 해당 퍼블릭 아이피를 통해 정상적으로 웹 애플리케이션에 접속되는 것을 확인합니다.

![브라우저의 Todo 웹 애플리케이션 확인 화면](assets/21-application-browser.png)

### SQL 데이터베이스 연동

> **참고**: 아래 SQL 계정의 고정 암호는 문서용 예시입니다. 실제 운영 자격 증명으로 재사용하지 않습니다.

#### SQL 데이터베이스 서버 만들기

![SQL 서버 만들기 단계 안내](assets/22-sql-server-overview.png)

1. 왼쪽 상단 검색창에서 `SQL Server` 입력하여 SQL Server 화면으로 이동합니다.
2. `만들기` 버튼을 클릭합니다.
3. 아래와 같이 구성하고 `검토 + 만들기` 버튼을 클릭합니다.

![SQL 서버 기본 설정](assets/23-sql-server-basics.png)

- 서버 이름 : basic-workshop-<name>
- 위치 : (Asia Pacific) 한국 중부
- 인증 방법 : SQL 인증 사용
    - 서버 관리자 로그인 : db-admin
    - 암호 : Todo123!@

1. 리소스가 생성되면 `리소스로 이동` 버튼을 클릭합니다.

#### SQL 데이터베이스 만들기

1. 왼쪽 상단 검색창에서 `SQL 데이터베이스` 입력하여 SQL 데이터베이스 화면으로 이동합니다.
2. `만들기` 버튼을 클릭합니다.

![SQL 데이터베이스 만들기 화면](assets/24-sql-database-create.png)

1. 아래와 같이 구성합니다.
- 구독 : 생성한 구독 선택
- 리소스 그룹 : BasicWorkshopRG
- 데이터베이스 이름 : basic-workshop
- 서버 : basic-workshop

1. 나머지 설정은 그대로 두고, `검토+만들기` 버튼을 클릭합니다. 다음과 같은 화면이 뜨면 구성 내용을 확인하고 `만들기` 버튼을 클릭합니다.

![SQL 데이터베이스 구성 검토 화면](assets/25-sql-database-review.png)

#### 데이터베이스 프라이빗 엔드포인트 만들기

1. 왼쪽 메뉴에서 `네트워킹`을 클릭하고 `프라이빗 액세스` 탭을 클릭합니다.
2. `프라이빗 엔드포인트 만들기`를 클릭합니다.

![SQL 서버의 프라이빗 액세스 설정](assets/26-sql-private-access.png)

1. 아래와 같이 구성 후, `다음: 리소스 >` 버튼을 클릭합니다.

![SQL Private Endpoint 기본 설정](assets/27-private-endpoint-basics.png)

- 구독 : 생성한 구독 선택
- 리소스 그룹 : BasicWorkshopRG
- 인스턴스 정보
    - 이름 : DBPrivateEP
    - 네트워크 인터페이스 이름 : DBPrivateEP-nic (자동생성)
    - 지역 : 한국 중부

1. 대상 하위 리소스가 자동으로 `sqlServer`로 선택되면 `다음: 가상 네트워크 >` 버튼을 클릭합니다.
2. 아래와 같이 설정한 뒤 나머지 설정은 그대로 두고 프라이빗 엔드포인트를 생성합니다.

![SQL Private Endpoint의 가상 네트워크 설정](assets/28-private-endpoint-networking.png)

- 가상 네트워크 : BasicVNet(basicworkshoprg)
- 서브넷 : default

#### SQL 데이터베이스 연동

1. 리소스가 생성되면 왼쪽 메뉴에서 `개요`를 클릭하고 `서버 이름`을 복사합니다.

![SQL 서버 이름 확인 화면](assets/29-sql-server-name.png)

1. 전 단계에서 띄워둔 웹 페이지(부하 분산 장치의 프런트 엔드 IP)로 접속하고 `Connect to DB` 탭을 클릭합니다.

![Todo 웹 애플리케이션의 Connect to DB 탭](assets/30-application-database-tab.png)

1. 아래와 같이 구성 후, `Connect` 버튼을 클릭합니다.

![Todo 웹 애플리케이션의 SQL 연결 설정](assets/31-application-database-connect.png)

- Server Name : SQL 데이터베이스 서버 이름
- Server Admin : db-admin
- Password : Todo123!@

1. 정상적으로 연결이 완료되면, 다시 List 화면으로 돌아와서 기능들을 테스트합니다.

![SQL 연결 후 Todo 목록 기능 확인 화면](assets/32-application-database-result.png)

## 검증

웹 애플리케이션 배포 단계의 [배포 테스트](#배포-테스트)에서 Load Balancer의 공용 IP로 접속되는지 확인합니다. SQL 연결 후에는 List 화면으로 돌아와 기능을 테스트합니다.

<!-- TODO: 저자 작성 — SQL 연결 후 목록 기능의 구체적인 성공 기준과 확인할 데이터 -->

## 정리 (Clean-up)

실습이 끝난 후 리소스 그룹을 삭제하여 불필요한 비용이 발생하지 않도록 합니다.

수고하셨습니다. 감사합니다🙂

<!-- TODO: 저자 확인 — 삭제 대상이 실습 전용 BasicWorkshopRG인지 확인하는 안내와 삭제 완료 확인 절차 -->

## 트러블슈팅

<!-- TODO: 저자 확인 — VM 생성 단계의 TodoVM과 Bastion 연결 단계의 TodoVM1 불일치, SQL 서버 선택 값, 패키지 설치·ODBC 연결·Private Endpoint 오류의 해결 방법 -->

## 이전·다음

[이전: Basic](../01-basic/README.md) · [다음: 워크샵 홈](../README.md)
