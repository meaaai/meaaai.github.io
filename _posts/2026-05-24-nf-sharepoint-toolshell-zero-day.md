---
layout: post
title: "[NF] Microsoft SharePoint ToolShell 제로데이 분석"
date: 2026-05-24 21:00:00 +0900
categories: [Security, Incident-Analysis]
tags: [nf, security, incident-analysis, microsoft, sharepoint, toolshell, zero-day, cve, rce, webshell]
permalink: /posts/nf-sharepoint-toolshell-zero-day/
description: "Microsoft SharePoint ToolShell 제로데이 사고의 공격 방식, 원인, 대응 방법을 정리한 글입니다."
---

## 1. 사건 개요

2025년 7월, Microsoft SharePoint Server에서 **ToolShell**이라고 불리는 취약점 체인이 실제 공격에 악용됐다. 여기서 **SharePoint Server(기업이나 기관이 내부 문서, 협업 자료, 업무 데이터를 관리하기 위해 사용하는 Microsoft의 서버형 협업 플랫폼)** 는 회사 내부 자료가 모이는 시스템이기 때문에 공격자 입장에서 매우 가치가 높은 대상이다.

이번 사고에서 중요한 점은 모든 SharePoint가 영향을 받은 것은 아니라는 점이다. Microsoft는 이번 취약점이 **온프레미스 SharePoint Server(기업이 직접 설치하고 운영하는 서버형 SharePoint)** 에 영향을 주며, Microsoft 365에서 제공되는 **SharePoint Online(마이크로소프트가 클라우드에서 관리하는 SharePoint 서비스)** 은 영향을 받지 않는다고 밝혔다.

Microsoft는 공격자가 CVE-2025-49704, CVE-2025-49706, CVE-2025-53770, CVE-2025-53771 등을 이용했다고 설명했다. 특히 CVE-2025-53770은 공격자가 인증 없이 네트워크를 통해 코드를 실행할 수 있는 취약점으로 알려졌다. NVD도 이 취약점을 “온프레미스 Microsoft SharePoint Server에서 신뢰할 수 없는 데이터 역직렬화로 인해 인증되지 않은 공격자가 네트워크를 통해 코드를 실행할 수 있는 취약점”으로 설명한다.

또한 Microsoft는 Linen Typhoon, Violet Typhoon, Storm-2603 등 중국 기반 위협 행위자들이 이 취약점을 악용한 정황을 관찰했다고 밝혔다. 특히 Storm-2603은 이후 랜섬웨어 배포에도 이 취약점을 사용한 것으로 언급됐다.

## 2. 공격 방식

이번 공격의 핵심은 **인증 우회와 원격 코드 실행**이다.

**인증 우회(authentication bypass, 원래는 로그인이나 권한 확인을 거쳐야 하는 기능에 공격자가 정상 인증 없이 접근하는 것)** 가 가능해지면 공격자는 계정 정보 없이도 서버의 특정 기능을 건드릴 수 있다. 여기에 **원격 코드 실행(RCE, Remote Code Execution, 공격자가 멀리 떨어진 곳에서 서버에 명령어나 코드를 실행시키는 공격)** 이 결합되면 서버를 장악할 수 있다.

공격자는 취약한 SharePoint 서버에 조작된 요청을 보낸 뒤, 서버 안에 악성 파일을 업로드했다. Microsoft는 실제 공격에서 `spinstall0.aspx`라는 악성 스크립트가 사용됐다고 설명했다. 이 파일은 **웹셸(web shell, 공격자가 웹 요청을 통해 서버에 명령을 내릴 수 있게 해주는 악성 파일)** 로 사용될 수 있으며, 공격자는 이를 통해 서버 내부 명령을 실행하거나 민감한 정보를 빼낼 수 있었다.

`spinstall0.aspx`가 특히 위험했던 이유는 단순히 명령 실행만 하는 것이 아니라, **MachineKey(ASP.NET 애플리케이션에서 데이터 검증과 암호화에 사용하는 서버 측 암호화 키)** 정보를 가져오는 기능도 포함하고 있었기 때문이다. MachineKey가 탈취되면 공격자는 패치 이후에도 정상 요청처럼 보이는 악성 요청을 만들거나, 서버에 다시 접근할 가능성이 생긴다. Microsoft도 `spinstall0.aspx`가 MachineKey 데이터를 가져와 공격자에게 전달할 수 있었다고 설명했다.

## 3. 기술적으로 중요한 특징

ToolShell 사고에서 가장 중요한 부분은 “서버에 파일 하나가 생겼다” 정도로 끝나는 문제가 아니라는 점이다. SharePoint는 기업 내부 문서, 계정 정보, 업무 시스템과 연결되어 있는 경우가 많다. 따라서 공격자가 SharePoint 서버를 장악하면 단순히 웹사이트 하나가 뚫리는 것이 아니라 내부망 침투의 출발점이 될 수 있다.

Unit 42 분석에 따르면 공격자는 먼저 SharePoint 서버가 취약한지 확인하는 정찰 활동을 했고, 이후 취약한 서버를 대상으로 공격을 시도했다. 일부 공격에서는 웹셸을 심거나 PowerShell 명령을 실행하는 방식이 관찰됐다.

여기서 **PowerShell(Windows에서 시스템 관리와 자동화에 사용하는 명령어 도구)** 은 원래 관리자들이 서버를 관리할 때 쓰는 정상 도구다. 하지만 공격자가 서버에 접근한 뒤 PowerShell을 사용하면 파일 다운로드, 추가 악성코드 실행, 계정 정보 수집, 보안 기능 비활성화 같은 행동을 할 수 있다.

또한 Kaspersky는 CVE-2025-53770과 CVE-2025-53771을 함께 악용하면 인증되지 않은 공격자가 SharePoint 서버를 장악할 수 있고, 서버 안의 정보에 접근하거나 내부 인프라로 공격을 확장할 수 있다고 분석했다.

## 4. 근본 원인

이번 사고의 근본 원인은 크게 세 가지로 볼 수 있다.

첫 번째는 **외부에 노출된 온프레미스 서버**다. SharePoint Server를 회사 내부에서 직접 운영하면서 인터넷에 노출해 두면, 공격자는 전 세계 어디서든 취약한 서버를 찾고 공격을 시도할 수 있다. 특히 SharePoint처럼 내부 문서와 업무 정보가 모이는 서버는 공격자에게 좋은 표적이 된다.

두 번째는 **패치 지연**이다. 보안 업데이트가 나와도 실제 기업 환경에서는 바로 적용되지 않는 경우가 많다. 업무 중단 우려, 호환성 문제, 담당자 부족 등으로 패치가 늦어지면 공격자는 그 틈을 노린다. Kaspersky는 ToolShell 취약점이 낮은 노력으로 악용될 수 있고, 공개 익스플로잇이 나온 뒤에도 오랫동안 공격에 재사용될 가능성이 높다고 분석했다.

세 번째는 **기존 패치 우회**다. ToolShell은 이전에 공개된 취약점과 관련이 깊다. Kaspersky 분석에 따르면 CVE-2025-53770과 CVE-2025-53771은 앞서 패치된 CVE-2025-49704, CVE-2025-49706과 유사한 흐름을 보였고, 기존 패치가 충분하지 않아 우회가 가능했던 것으로 분석됐다.

## 5. 대응 방법

이번 사고에서 중요한 점은 패치만 하고 끝내면 안 된다는 것이다. 이미 공격자가 서버에 웹셸을 심었거나 MachineKey를 빼갔다면, 보안 업데이트를 적용한 뒤에도 다시 접근할 가능성이 남아 있기 때문이다.

Microsoft는 지원되는 SharePoint Server 버전에 최신 보안 업데이트를 즉시 적용하라고 안내했다. 또한 **AMSI(Antimalware Scan Interface, 애플리케이션에서 실행되는 스크립트나 명령을 백신·보안 제품이 검사할 수 있게 해주는 Microsoft 보안 인터페이스)** 를 활성화하고, Defender Antivirus 또는 비슷한 보안 제품을 SharePoint 서버에 적용하라고 권고했다.

또한 Microsoft는 최신 보안 업데이트 적용 후 **ASP.NET MachineKey를 교체하고 IIS를 재시작**하라고 안내했다. 여기서 **IIS(Internet Information Services, Windows 서버에서 웹 서비스를 운영할 때 사용하는 Microsoft 웹 서버)** 를 재시작해야 새 키가 제대로 적용된다.

CISA도 CVE-2025-53770을 2025년 7월 20일 **KEV(Known Exploited Vulnerabilities, 실제 공격에 악용된 것으로 확인된 취약점 목록)** 에 추가했다. 이는 이 취약점이 단순히 이론적인 위험이 아니라 실제 공격에 쓰였다는 의미다.

## 6. 미흡했던 부분

가장 아쉬운 부분은 외부에 노출된 온프레미스 서버 관리다. SharePoint처럼 내부 문서와 인증 정보가 모이는 시스템은 공격자 입장에서 매우 가치가 높다. 그런데 이런 서버가 인터넷에서 바로 접근 가능한 상태라면, 취약점이 공개되는 순간 공격 대상이 되기 쉽다.

물론 모든 서버를 완전히 내부망에만 둘 수는 없다. 하지만 최소한 VPN, 프록시, 인증 게이트웨이, 방화벽 정책 등을 통해 아무나 접근하지 못하게 제한했어야 한다. Microsoft도 AMSI를 사용할 수 없다면 서버를 인터넷에서 분리하거나, VPN·프록시·인증 게이트웨이로 인증되지 않은 트래픽을 제한하라고 권고했다.

또 하나의 문제는 패치를 “설치”로만 생각했다는 점이다. 취약점 공격이 이미 발생한 뒤라면 패치는 문을 잠그는 역할만 한다. 이미 집 안에 들어온 공격자를 내보내는 작업, 즉 웹셸 제거, 로그 분석, 키 교체, 계정 점검까지 함께 해야 한다.

## 7. 재발 방지 대책

이번 사고를 막기 위해 가장 먼저 해야 할 일은 **외부에서 바로 접속 가능한 SharePoint 서버를 줄이는 것**이다. SharePoint는 회사 내부 자료가 많이 모이는 시스템이기 때문에, 인터넷에 그대로 열어두면 공격자에게 너무 좋은 표적이 된다. 꼭 외부 접속이 필요하다면 VPN이나 인증 게이트웨이처럼 한 번 더 확인하는 장치를 두는 것이 좋다.

두 번째로는 **보안 업데이트를 빠르게 적용하는 것**이다. 특히 이번처럼 실제 공격에 사용된 취약점은 며칠만 늦어도 피해가 생길 수 있다. 보안 패치가 나오면 “언젠가 해야지”가 아니라, 우선순위를 높여 빠르게 적용해야 한다.

세 번째로는 **패치 후에도 침해 흔적을 확인하는 것**이다. 공격자가 이미 `spinstall0.aspx` 같은 웹셸을 심어두었다면 패치만으로는 부족하다. 서버 안에 수상한 파일이 생기지 않았는지, `w3wp.exe` 같은 IIS 프로세스가 이상한 명령을 실행하지 않았는지, PowerShell이 비정상적으로 실행된 기록이 있는지 확인해야 한다.

네 번째로는 **MachineKey를 교체하는 것**이다. MachineKey는 서버가 요청을 검증할 때 쓰는 중요한 키다. 만약 이 키가 공격자에게 넘어갔다면, 공격자는 나중에도 정상 요청처럼 보이게 접근을 시도할 수 있다. 그래서 패치 후에는 기존 키를 버리고 새 키로 바꾸는 과정이 필요하다.

다섯 번째로는 **서버 보안 제품을 켜두고 탐지 기능을 활용하는 것**이다. AMSI, Defender, EDR 같은 도구는 공격자가 웹셸을 실행하거나 PowerShell 명령을 사용할 때 이를 탐지하는 데 도움을 준다. 보안 제품을 설치만 해두는 것이 아니라, 실제로 켜져 있는지와 최신 상태인지 확인해야 한다.

정리하면 이번 사고의 재발 방지 핵심은 어렵지 않다. **서버를 아무에게나 열어두지 않고, 패치를 빨리 적용하고, 공격 흔적을 확인하고, 중요한 키를 바꾸고, 보안 탐지를 켜두는 것**이다. 취약점 대응은 패치 하나로 끝나는 작업이 아니라, 공격자가 이미 들어왔는지까지 확인하는 과정이라고 볼 수 있다.

## 8. 느낀 점

이 사건은 “패치만 하면 끝”이라는 생각이 얼마나 위험한지 보여준다. 취약점이 공개되고 패치가 나왔다고 해도, 그 전에 공격자가 이미 웹셸을 심어두었거나 MachineKey를 탈취했다면 문제는 계속 남아 있을 수 있다.

특히 SharePoint는 단순한 웹 서비스가 아니라 기업 내부 문서와 업무 정보가 모이는 시스템이다. 이런 서버가 뚫리면 하나의 서비스 장애로 끝나는 것이 아니라 내부망 침투, 계정 탈취, 랜섬웨어 배포로 이어질 수 있다.

이번 사례를 통해 취약점 대응은 **패치 → 로그 확인 → 웹셸 제거 → 키 교체 → 외부 노출 점검**까지 이어져야 한다는 점을 알 수 있었다. 보안에서 중요한 것은 “문을 잠그는 것”뿐만 아니라, 이미 누가 들어와 있지는 않은지 확인하는 일이다.

## 9. 참고 자료

- [Microsoft Security Blog, “Disrupting active exploitation of on-premises SharePoint vulnerabilities”](https://www.microsoft.com/en-us/security/blog/2025/07/22/disrupting-active-exploitation-of-on-premises-sharepoint-vulnerabilities/)
- [NVD, “CVE-2025-53770”](https://nvd.nist.gov/vuln/detail/CVE-2025-53770)
- [CISA, “Microsoft Releases Guidance on Exploitation of SharePoint Vulnerabilities”](https://www.cisa.gov/news-events/alerts/2025/07/20/update-microsoft-releases-guidance-exploitation-sharepoint-vulnerabilities)
- [Kaspersky Securelist, “ToolShell explained”](https://securelist.com/toolshell-explained/117045/)
- [Unit 42, “Active Exploitation of Microsoft SharePoint Vulnerabilities”](https://unit42.paloaltonetworks.com/microsoft-sharepoint-cve-2025-49704-cve-2025-49706-cve-2025-53770/)
