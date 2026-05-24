---
layout: post
title: "[NF] Salesloft Drift OAuth 토큰 탈취 사고 분석"
date: 2026-05-24 21:10:00 +0900
categories: [Security, Incident-Analysis]
tags: [nf, security, incident-analysis, salesloft, drift, salesforce, oauth, token, saas, api]
permalink: /posts/nf-salesloft-drift-oauth-token-breach/
description: "Salesloft Drift OAuth 토큰 탈취 사고의 공격 흐름과 SaaS 연동 보안 문제를 정리한 글입니다."
---

## 1. 사건 개요

2025년 8월, **Salesloft Drift(Salesloft가 제공하는 대화형 마케팅·영업 챗봇 및 고객 응대 도구)** 와 **Salesforce(고객 정보, 영업 활동, 문의 내역 등을 관리하는 CRM 플랫폼)** 연동에서 사용되던 OAuth 토큰이 탈취되어 여러 기업의 Salesforce 데이터가 유출되는 사고가 발생했다.

Google Threat Intelligence Group은 이 캠페인을 **UNC6395(구글이 추적한 위협 행위자 이름)** 로 분류했다. GTIG에 따르면 공격자는 2025년 8월 8일부터 최소 8월 18일까지 Salesloft Drift 제3자 애플리케이션과 연결된 Salesforce 고객 인스턴스를 대상으로 대량 데이터 탈취를 수행했다. 공격자는 단순히 데이터를 가져가는 데서 끝나지 않고, 탈취한 데이터 안에서 AWS access key, 비밀번호, Snowflake 토큰 같은 추가 인증정보를 찾으려 했다. ([Google Cloud](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift))

이 사건은 Salesforce 자체가 해킹된 사건이라기보다는, Salesforce와 연결된 제3자 SaaS 애플리케이션의 인증 토큰이 악용된 사건이다. GTIG도 이 문제가 Salesforce 핵심 플랫폼의 취약점에서 비롯된 것은 아니라고 설명했다. ([Google Cloud](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift))

## 2. 공격 방식

공격 방식은 일반적인 비밀번호 탈취와 달랐다. 공격자는 사용자의 Salesforce 비밀번호를 훔친 것이 아니라, Drift 연동에 사용되던 **OAuth 토큰(OAuth 인증 과정에서 발급되는 접근 권한 증명값으로, 사용자가 매번 로그인하지 않아도 애플리케이션이 대신 API에 접근할 수 있게 해주는 토큰)** 을 악용했다.

OAuth는 정상적으로 쓰이면 편리한 인증 방식이다. 예를 들어 사용자가 Drift와 Salesforce를 연결하면, Drift는 사용자가 다시 로그인하지 않아도 Salesforce API에 접근해 고객 정보나 문의 데이터를 가져올 수 있다. 그러나 이 토큰이 탈취되면 공격자도 정상 앱처럼 보이면서 Salesforce API에 접근할 수 있다. 즉, 비밀번호나 MFA를 직접 우회하지 않아도 이미 발급된 권한을 이용해 데이터를 조회할 수 있는 것이다.

Cloudflare의 사고 분석을 보면 공격자는 실제로 Salesforce API를 이용해 환경을 파악하고 데이터를 빼냈다. 공격자는 `/services/data/v58.0/sobjects/` 엔드포인트를 호출해 Salesforce 객체 목록을 확인했고, `/sobjects/Case/describe/`를 통해 Case 객체의 구조를 파악했다. 이후 Case 객체에 대한 쿼리를 실행해 고객 지원 티켓 데이터를 가져갔다. ([The Cloudflare Blog](https://blog.cloudflare.com/response-to-salesloft-drift-incident/))

여기서 **API(Application Programming Interface, 프로그램끼리 데이터를 주고받기 위한 약속된 통신 방식)** 는 정상 서비스 연동에도 사용되지만, 공격자가 인증 토큰을 가지고 있으면 대량 데이터 조회 통로가 될 수 있다. 또한 **Case 객체(Salesforce에서 고객 문의, 지원 티켓, 상담 기록 등을 저장하는 데이터 객체)** 에는 고객이 문제 해결을 위해 남긴 로그, 설정값, 토큰, 비밀번호 등이 포함될 수 있어 피해가 더 커질 수 있다.

## 3. 기술적으로 중요한 특징

이 사고에서 가장 중요한 특징은 공격자가 “로그인 계정”이 아니라 “연동 권한”을 악용했다는 점이다. 많은 기업은 Salesforce, Slack, Google Workspace, Drift, GitHub 같은 여러 SaaS를 서로 연결해 사용한다. 이런 구조에서는 하나의 서비스가 다른 서비스에 접근할 수 있도록 OAuth 토큰이나 API 키를 저장한다.

문제는 이 토큰이 사용자 계정과 비슷한 권한을 갖는다는 점이다. 예를 들어 Drift가 Salesforce의 고객 지원 데이터를 읽을 수 있는 권한을 가지고 있었다면, 공격자도 탈취한 Drift OAuth 토큰을 이용해 같은 데이터를 읽을 수 있다. 사용자는 Salesforce에 직접 로그인하지 않았고, Salesforce 자체 계정 비밀번호가 유출되지 않았더라도 데이터는 빠져나갈 수 있다.

GTIG는 공격자가 Salesforce의 Cases, Accounts, Users, Opportunities 같은 객체에서 정보를 조회했다고 밝혔다. 또한 `SELECT COUNT() FROM Account`, `SELECT COUNT() FROM User`, `SELECT COUNT() FROM Case` 같은 SOQL 쿼리를 사용해 각 데이터 객체의 규모를 확인한 정황도 공개했다. **SOQL(Salesforce Object Query Language, Salesforce 데이터 객체를 조회하기 위한 쿼리 언어)** 은 데이터베이스의 SQL과 비슷하게 Salesforce 내부 객체를 검색하는 데 사용된다. ([Google Cloud](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift))

Cloudflare 사례에서는 공격자가 정찰 단계에서 Salesforce 객체를 열거하고, Case 객체 구조를 확인하고, API 제한을 확인한 뒤, 최종적으로 Salesforce Bulk API 2.0 작업을 이용해 고객 지원 티켓 텍스트를 반출했다. **Bulk API(대량의 데이터를 비동기 방식으로 조회·삽입·수정할 수 있는 Salesforce API)** 는 정상적으로는 대규모 데이터 처리에 유용하지만, 공격자가 악용하면 대량 유출 통로가 될 수 있다. ([The Cloudflare Blog](https://blog.cloudflare.com/response-to-salesloft-drift-incident/))

## 4. 사고 범위와 추가 영향

초기에는 Drift와 Salesforce 연동이 주요 피해 범위로 알려졌지만, 이후 조사에서 Drift Email 연동도 문제가 된 것으로 확인됐다. GTIG는 2025년 8월 28일 Drift Email 연동 OAuth 토큰도 침해되었고, 2025년 8월 9일 공격자가 이 토큰을 사용해 Salesloft Drift와 연동된 매우 적은 수의 Google Workspace 계정 이메일에 접근했다고 밝혔다. 다만 Google은 Google Workspace나 Alphabet 자체가 침해된 것은 아니라고 설명했다. ([Google Cloud](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift))

이 부분이 중요한 이유는 “Salesforce만 확인하면 된다”로 끝나지 않기 때문이다. Drift와 연결된 다른 SaaS나 이메일 연동이 있다면, 해당 연동 토큰도 함께 점검해야 한다. GTIG도 Drift 인스턴스에 연결된 모든 제3자 연동을 검토하고, 관련 자격증명과 토큰을 폐기·회전하라고 권고했다. ([Google Cloud](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift))

## 5. 근본 원인

이 사건의 근본 원인은 크게 세 가지로 볼 수 있다.

첫째, **제3자 SaaS 연동 보안(third-party SaaS integration security, 외부 클라우드 서비스끼리 연결할 때 발생하는 권한·토큰·데이터 접근 관리 문제)** 이 충분히 통제되지 않았다. 기업들은 업무 효율을 위해 여러 SaaS를 연결하지만, 연결된 앱이 어떤 데이터에 접근할 수 있는지, 토큰이 어디에 저장되는지, 토큰이 유출되면 어디까지 접근 가능한지까지 세밀하게 관리하지 못하는 경우가 많다.

둘째, **과도한 권한 부여(over-permission, 필요한 범위보다 더 넓은 접근 권한을 부여하는 상태)** 가 위험을 키웠다. GTIG는 연결 앱의 scope를 최소 권한으로 제한하고, `full` access 같은 과도한 scope를 피하라고 권고했다. **scope(OAuth 토큰에 부여되는 접근 범위)** 가 넓을수록 토큰 탈취 시 피해 범위도 커진다. ([Google Cloud](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift))

셋째, 고객 지원 데이터 안에 민감정보가 포함될 수 있었다. Cloudflare는 Salesforce Case 데이터가 고객 지원 티켓 본문과 연락처 정보를 포함하며, 일부 고객이 문제 해결 과정에서 토큰, 비밀번호, 로그 같은 민감정보를 입력했을 가능성이 있다고 설명했다. Cloudflare는 침해된 데이터 안에서 104개의 Cloudflare API 토큰을 발견했고, 예방 차원에서 모두 회전했다. ([The Cloudflare Blog](https://blog.cloudflare.com/response-to-salesloft-drift-incident/))

## 6. 대응 방법

Salesloft와 Salesforce는 2025년 8월 20일 Drift 애플리케이션의 활성 access token과 refresh token을 모두 폐기했다. 또한 Salesforce는 Drift 애플리케이션을 Salesforce AppExchange에서 제거했다. **Access token(현재 API 접근에 사용되는 토큰)** 은 실제 요청을 보낼 때 쓰이고, **refresh token(access token이 만료되었을 때 새 access token을 발급받기 위한 토큰)** 은 장기 접근을 가능하게 하기 때문에 둘 다 폐기하는 것이 중요하다. ([Google Cloud](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift))

이후 Salesforce는 2025년 8월 28일 Drift 애플리케이션과 Salesforce 간 연결을 비활성화했다. Salesforce 보안 공지에서도 Salesloft와 협력해 활성 access token 및 refresh token을 무효화하고 Drift를 연결 해제했다고 설명했다. ([Salesforce](https://help.salesforce.com/s/articleView?id=005134951&language=en_US&type=1&utm_source=chatgpt.com))

Cloudflare는 자체 대응 과정에서 Drift 통합을 비활성화하고, Salesloft 관련 소프트웨어와 브라우저 확장 프로그램을 제거했으며, Salesforce와 연결된 제3자 연동을 재검토하고 자격증명을 교체했다. 또한 고객 지원 티켓 안에 포함되었을 수 있는 토큰과 비밀번호를 찾기 위해 정규식, 엔트로피 분석, 패턴 매칭을 활용했다고 밝혔다. ([The Cloudflare Blog](https://blog.cloudflare.com/response-to-salesloft-drift-incident/))

## 7. 미흡했던 부분

이 사고에서 가장 아쉬운 부분은 제3자 앱 권한과 OAuth 토큰 관리다. OAuth 토큰은 사용자 편의성을 높이지만, 탈취되면 공격자가 비밀번호 없이도 API에 접근할 수 있다. 특히 refresh token까지 탈취되면 access token을 계속 새로 발급받을 수 있어 장기간 접근이 가능해진다.

또 다른 문제는 SaaS 연동이 “보이지 않는 공격 표면”이 된다는 점이다. 서버나 방화벽처럼 눈에 잘 보이는 자산은 보안팀이 관리하기 쉽지만, 각 부서가 업무 효율을 위해 연결한 SaaS 앱은 전체 목록조차 제대로 파악되지 않는 경우가 있다. 사용하지 않는 앱이 연결된 채 남아 있거나, 퇴사자 계정과 연결된 토큰이 유지되거나, 관리자 권한으로 연결된 앱이 과도한 범위를 갖고 있다면 큰 위험이 된다.

마지막으로 고객 지원 채널에 민감정보가 들어가는 관행도 문제다. 지원 티켓은 원래 문제 해결을 위한 공간이지만, 실제로는 로그, API 키, 토큰, 비밀번호, 내부 URL 등이 입력될 수 있다. 공격자가 지원 티켓 데이터를 탈취하면 단순 개인정보 유출을 넘어 2차 침해로 이어질 수 있다.

## 8. 재발 방지 대책

이번 사고를 막기 위해 가장 먼저 해야 할 일은 **연결된 앱 목록을 확인하는 것**이다. 회사에서 Salesforce, Drift, Slack, Google Workspace 같은 서비스를 함께 사용할 때는 서로 연결된 앱들이 생긴다. 문제는 사용하지 않는 앱이 계속 연결되어 있거나, 어떤 앱이 어떤 데이터에 접근하는지 모르는 경우가 있다는 점이다. 그래서 정기적으로 “우리 회사 계정에 어떤 외부 앱이 연결되어 있는지” 확인하고, 더 이상 쓰지 않는 앱은 삭제해야 한다.

두 번째로는 **권한을 필요한 만큼만 주는 것**이 중요하다. 예를 들어 어떤 앱이 고객 문의 내용만 확인하면 되는데, 전체 고객 정보나 영업 자료까지 볼 수 있게 해두면 위험하다. 이를 **최소 권한 원칙(필요한 기능을 수행하는 데 필요한 최소한의 권한만 주는 보안 원칙)** 이라고 한다. 권한이 작을수록 토큰이 유출되더라도 피해 범위가 줄어든다.

세 번째로는 **토큰을 주기적으로 바꾸고, 의심되면 바로 폐기하는 것**이다. OAuth 토큰은 쉽게 말해 “로그인 없이 서비스를 이용할 수 있게 해주는 출입증”과 비슷하다. 이 출입증이 공격자에게 넘어가면 비밀번호를 몰라도 데이터에 접근할 수 있다. 따라서 오래된 토큰은 그대로 두지 말고 주기적으로 새로 발급해야 하며, 사고가 의심되면 즉시 기존 토큰을 무효화해야 한다.

네 번째로는 **접속 기록을 확인하는 것**이다. 공격자는 훔친 토큰을 이용해 API를 호출하고 데이터를 가져간다. 따라서 평소보다 갑자기 많은 데이터가 조회되었는지, 낯선 IP 주소에서 접속했는지, 평소 사용하지 않던 도구로 API 요청이 들어왔는지 확인해야 한다. 쉽게 말해 “누가, 언제, 어디서, 얼마나 많은 데이터를 가져갔는지” 로그를 통해 살펴봐야 한다.

다섯 번째로는 **고객 지원 티켓에 민감정보를 남기지 않는 것**이다. 고객 문의나 기술 지원 과정에서 API 키, 비밀번호, 토큰, 서버 로그 같은 정보가 그대로 입력되는 경우가 있다. 이런 정보가 지원 티켓에 남아 있으면, 티켓 데이터가 유출됐을 때 2차 피해로 이어질 수 있다. 따라서 비밀번호나 토큰은 지원 티켓에 직접 적지 않도록 안내하고, 이미 입력된 민감정보는 마스킹하거나 삭제하는 정책이 필요하다.

정리하면, 이 사고를 막기 위한 핵심은 어렵지 않다. **사용하지 않는 연결은 끊고, 권한은 작게 주고, 토큰은 오래 방치하지 않고, 이상한 접속 기록을 확인하고, 중요한 정보는 지원 티켓에 남기지 않는 것**이다. OAuth나 SaaS 연동은 편리하지만, 연결된 서비스 하나가 뚫리면 다른 서비스의 데이터까지 위험해질 수 있기 때문에 꾸준한 관리가 필요하다.

## 9. 느낀 점

Salesloft Drift 사고는 클라우드 시대의 보안이 단순히 “우리 서버를 잘 지키는 것”만으로 끝나지 않는다는 점을 보여준다. 회사의 핵심 시스템이 직접 해킹되지 않았더라도, 신뢰하고 연결해 둔 외부 SaaS가 침해되면 내부 데이터에 접근할 수 있다.

특히 이 사건은 OAuth 토큰이 사실상 “비밀번호 없는 출입증”처럼 작동할 수 있다는 점을 보여준다. 토큰은 사용자가 직접 입력하는 비밀번호보다 눈에 덜 띄지만, API 접근 권한을 가지고 있기 때문에 유출되면 피해가 매우 크다.

결국 SaaS 보안의 핵심은 연결 관리다. 어떤 앱이 어떤 데이터에 접근하는지, 권한은 최소화되어 있는지, 오래된 토큰은 남아 있지 않은지, API 호출 로그는 탐지되고 있는지 계속 확인해야 한다. 이번 사건은 제3자 연동, API 권한, OAuth 토큰 관리가 현대 보안에서 필수 점검 항목이라는 사실을 잘 보여준다.

## 10. 참고 자료

- [Google Threat Intelligence Group, “Widespread Data Theft Targets Salesforce Instances via Salesloft Drift”](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift)
- [Cloudflare Blog, “The impact of the Salesloft Drift breach on Cloudflare and our customers”](https://blog.cloudflare.com/response-to-salesloft-drift-incident/)
- [Salesforce Security Advisory, “Drift App (Salesloft) Unauthorized Access Incident”](https://help.salesforce.com/s/articleView?id=005134951&type=1)
- [Salesforce Status, “Drift App (Salesloft) Connection Disabled”](https://status.salesforce.com/generalmessages/20000217)
- [Mandiant / Google Cloud Threat Intelligence, Salesloft Drift incident analysis](https://cloud.google.com/blog/topics/threat-intelligence/data-theft-salesforce-instances-via-salesloft-drift)
