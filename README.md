---
type: workshop
title: Azure Basic Workshop
description: Azure Portal에서 가상 네트워크, VM, Load Balancer와 Private Endpoint 기반 SQL Database를 구성하고 웹 애플리케이션을 배포하는 한국어 워크샵
level: intermediate
authors: [Anna Jeong]
contacts: ["@Anna-Jeong-MS"]
duration_minutes: 240
tags: [azure, virtual-network, virtual-machine, load-balancer, azure-sql, private-endpoint]
language: ko
execution: [portal]
status: draft
source: original
last_updated: 2026-10-11
validated_on:
original_content_date: 2025-04-17
---

# Azure Basic Workshop

Azure Basic Workshop은 애저 인프라 코어 서비스들을 직접 Hands on Lab 할 수 있는 컨텐츠입니다.

공식 워크샵 agenda는 L200 수준의 4시간(240분) 세션이며, Compute·Network·Storage·Security 서비스를 소개하는 이론 세션과 이 저장소의 포털 실습 세션으로 구성됩니다. 이론 세션 자료(슬라이드)는 이 저장소에 포함되어 있지 않습니다.

## 사전 요구사항

- Azure Portal에 로그인할 계정과 실습 리소스를 만들 수 있는 Azure 구독이 필요합니다.
- Azure Pass를 사용하는 행사라면 운영자가 제공한 프로모 코드를 준비합니다. 계정과 사용자 생성 절차는 [사전 준비](00-prerequisites/README.md)를 참고합니다.
- 실습에서 Azure 리소스를 생성하므로 비용이 발생할 수 있습니다. 사용할 구독과 리소스 구성을 확인하고, 실습을 마치면 각 랩의 정리 절차를 따릅니다.

<!-- TODO: 저자 확인 — 행사별 구독·권한 조건, 허용 리전, 리소스 비용과 한도 -->

## 학습 경로

[사전 준비](00-prerequisites/README.md)

  Azure Pass 활성화와 Microsoft Entra ID 사용자 생성을 준비합니다.

[Basic](01-basic/README.md)

  애저의 가장 기본적인 서비스인 가상 네트워크와 가상 머신을 활용하여 웹 애플리케이션을 배포합니다. 가상 네트워크를 구성하고, 가상머신을 배포한 후 웹서버를 배포합니다.

[Advanced](02-advanced/README.md)

  가상 머신에 웹 애플리케이션을 배포하고, 웹 애플리케이션의 데이터를 관리하기 위한 SQL 데이터베이스를 구성합니다. Load Balancer의 frontend IP를 웹 애플리케이션을 진입점으로 사용하고, 가상 머신을 백 엔드 풀로 사용합니다. 이렇게 구성하면 웹 애플리케이션을 논리적으로 격리할 수 있으며, 추후 애플리케이션의 확장을 고려한 아키텍처를 설계할 수 있습니다.

## 소요 시간

이 워크샵은 Compute·Network·Storage·Security 서비스를 소개하는 이론 세션과 포털 실습 세션으로 구성된 4시간(240분) 강의입니다.

| 세션 | 소요 시간 | 비고 |
|---|---|---|
| 이론 세션 | 40분 | Compute, Network, Storage, Security 서비스 소개. 이 저장소에는 자료가 포함되지 않음 |
| [사전 준비](00-prerequisites/README.md) | 10분 | Azure Pass 활성화, Microsoft Entra ID 사용자 생성 |
| [Basic](01-basic/README.md) | 70분 | 리소스 그룹·가상 네트워크·VM 구성과 웹 애플리케이션 배포 |
| [Advanced](02-advanced/README.md) | 120분 | Load Balancer·VM 구성과 Private Endpoint 기반 SQL Database 연동 |
| 합계 | 240분(4시간) | 공식 워크샵 agenda 기준 |

표의 예상 소요 시간은 공식 agenda의 전체 시간(4시간)을 단계 수와 복잡도에 따라 배분했습니다. 이론 세션 없이 실습만 진행할 때의 예상 소요 시간은 사전 준비·Basic·Advanced를 합쳐 약 200분입니다.

<!-- TODO: 저자 확인 — 위 추정치와 실제 E2E 완주 시간의 차이, 랩별 시간 재배분 필요 여부 -->
<!-- TODO: 저자 작성 — 이론 세션 자료(슬라이드 등)를 이 저장소에 포함할지 결정 -->

## 실행 환경과 외부 의존성

이 워크샵은 포털형입니다. Azure Portal에서 리소스를 구성하고, Azure Bastion으로 VM에 접속해 본문의 명령을 실행합니다. Codespaces나 로컬 개발 환경을 별도로 구성하지 않습니다.

| 의존성 | 사용 위치 |
|---|---|
| [BasicApp](https://github.com/Anna-Jeong-MS/BasicApp) | Basic의 웹 애플리케이션 배포 |
| [TodoApp](https://github.com/Anna-Jeong-MS/TodoApp) | Advanced의 웹 애플리케이션과 SQL 연결 |
| Ubuntu·Python 패키지 저장소 | VM에서 패키지 설치 |
| [Microsoft 패키지 저장소](https://packages.microsoft.com/) | Advanced의 ODBC 드라이버 설치 |

<!-- TODO: 저자 확인 — BasicApp·TodoApp의 접근 가능성·라이선스·크기·대체 경로, Ubuntu 이미지·설치 명령·Python 패키지 조합과 ODBC 드라이버의 현재 호환성 -->

외부 앱과 패키지는 이 저장소에 포함하지 않습니다. 각 랩에서 안내하는 외부 저장소와 서비스를 사용합니다.

## 검증과 정리

Basic은 VM의 공용 IP로 웹 애플리케이션에 접속합니다. Advanced는 Load Balancer의 공용 IP로 접속한 뒤 SQL 연결과 목록 기능을 확인합니다. 자세한 절차는 [Basic 검증](01-basic/README.md#검증)과 [Advanced 검증](02-advanced/README.md#검증)을 따릅니다.

실습 후에는 [Basic 정리](01-basic/README.md#정리-clean-up)와 [Advanced 정리](02-advanced/README.md#정리-clean-up)에 따라 실습 리소스 그룹을 정리합니다. 삭제 전에는 해당 리소스 그룹에 다른 용도의 리소스가 없는지 확인합니다.

<!-- TODO: 저자 검증 — 포털 UI와 스크린샷, Advanced의 NAT Gateway 누락·VM 이름 불일치, 리소스 삭제 완료 확인 -->
<!-- TODO: 저자 확인 — authors·contacts가 현재 저자 표기와 유지보수 연락처에 맞는지 확인 -->

## 출처와 기여

기존 [Azure Basic Workshop](https://github.com/Anna-Jeong-MS/AzureBasicWorkshop)의 실습 내용을 유지하며 문서 구조와 경로를 정리했습니다.

기여 방법은 [CONTRIBUTING](CONTRIBUTING.md), 문의 안내는 [SUPPORT](SUPPORT.md), 보안 문제 제보는 [SECURITY](SECURITY.md), 행동 강령은 [Microsoft Open Source Code of Conduct](.github/CODE_OF_CONDUCT.md)를 참고합니다.

## 라이선스

기존 Microsoft 저작권 고지와 [MIT 라이선스](LICENSE.md)를 유지합니다. 문서·이미지의 CC BY-SA 4.0 전환은 필요한 동의가 확인되기 전까지 적용하지 않습니다.
