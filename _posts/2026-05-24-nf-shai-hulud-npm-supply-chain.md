---
layout: post
title: "[NF] Shai-Hulud npm 공급망 공격 분석"
date: 2026-05-24 21:20:00 +0900
categories: [Security, Incident-Analysis]
tags: [nf, security, incident-analysis, shai-hulud, npm, supply-chain, malware, open-source, ci-cd, token]
permalink: /posts/nf-shai-hulud-npm-supply-chain/
description: "Shai-Hulud npm 공급망 공격의 감염 방식, 원인, 대응 방법을 정리한 글입니다."
---

## 1. 사건 개요

2025년 9월, JavaScript 생태계에서 널리 사용되는 **npm(Node Package Manager, Node.js에서 외부 라이브러리와 패키지를 설치·관리하는 패키지 관리자)** 을 대상으로 한 대규모 공급망 공격이 공개됐다. 이 공격에 사용된 악성코드는 **Shai-Hulud**라고 불렸으며, 단순히 하나의 패키지를 감염시키는 수준이 아니라 스스로 다른 패키지로 퍼질 수 있는 **자가 전파형 웜(worm, 사용자의 직접 실행 없이 스스로 복제·확산되는 악성코드)** 형태를 띠고 있었다.

GitHub는 2025년 9월 14일 Shai-Hulud 공격을 통보받았고, 이 공격이 손상된 npm maintainer 계정(compromised maintainer account, 패키지 배포 권한을 가진 개발자 계정이 탈취된 상태)을 통해 인기 JavaScript 패키지에 악성 `postinstall` 스크립트가 삽입되는 방식이었다고 설명했다. GitHub는 대응 과정에서 500개 이상의 손상된 패키지를 npm 레지스트리에서 제거했다고 밝혔다.

CISA도 2025년 9월 23일 npm 생태계에 영향을 주는 광범위한 공급망 침해를 경고했다. CISA는 공격자가 손상된 개발자 계정으로 npm 레지스트리에 인증한 뒤, 다른 패키지에 악성 코드를 주입하는 자동화 과정을 사용했다고 설명했다.

## 2. 공격 방식

이 공격의 핵심은 **패키지 설치 과정에서 자동 실행되는 스크립트**였다. npm 패키지는 설치될 때 `preinstall`, `install`, `postinstall` 같은 **라이프사이클 스크립트(lifecycle script, 패키지 설치·빌드·배포 과정의 특정 시점에 자동 실행되는 명령)** 를 실행할 수 있다. Shai-Hulud 1차 공격에서는 주로 `postinstall` 스크립트가 악용됐다. `postinstall`은 패키지가 설치된 직후 실행되는 스크립트로, 정상적으로는 빌드 준비나 환경 설정에 사용되지만 공격자가 악성 명령을 넣으면 패키지를 설치한 순간 악성코드가 실행될 수 있다. CERT/CC도 이 공격에서 `postinstall` 스크립트가 악성 `bundle.js` 실행으로 이어졌다고 설명했다.

공격 흐름은 다음과 같이 정리할 수 있다.

1. 공격자가 npm maintainer 계정 또는 npm 토큰을 탈취한다.
2. 공격자는 정상 npm 패키지에 악성 스크립트를 삽입한 새 버전을 배포한다.
3. 개발자나 CI/CD 환경이 해당 패키지를 설치한다.
4. 설치 과정에서 악성 `postinstall` 스크립트가 실행된다.
5. 악성코드는 `.npmrc` 파일, 환경변수, GitHub 토큰, 클라우드 API 키 등을 찾는다.
6. 탈취한 인증정보를 이용해 다른 패키지에도 악성 코드를 삽입하고 다시 배포한다.

Unit 42 분석에 따르면 악성코드는 `.npmrc` 파일(npm 인증 토큰이 저장될 수 있는 설정 파일), 환경변수(environment variables, 운영체제나 CI/CD가 프로그램에 전달하는 설정값), GitHub PAT(Personal Access Token, 비밀번호 대신 API 접근에 쓰이는 개인 인증 토큰), AWS·GCP·Azure 같은 클라우드 서비스 키를 수집하려 했다. 또한 훔친 npm 토큰을 이용해 피해 개발자가 관리하는 다른 패키지를 찾아 악성 코드를 주입하고 새 버전으로 배포하는 방식으로 확산됐다.

## 3. 기술적으로 중요한 특징

Shai-Hulud가 위험했던 이유는 단순한 정보 탈취형 악성코드가 아니라 **공급망 내부에서 스스로 전파되는 구조**였기 때문이다. 일반적인 악성 패키지는 설치한 사용자 한 명만 감염시키는 경우가 많지만, Shai-Hulud는 탈취한 npm 토큰(npm registry에 패키지를 배포하거나 수정할 수 있는 인증 토큰)을 이용해 다른 정상 패키지까지 오염시킬 수 있었다.

또한 이 공격은 개발자의 로컬 환경뿐 아니라 **CI/CD(Continuous Integration/Continuous Deployment, 코드 변경 사항을 자동으로 빌드·테스트·배포하는 개발 자동화 환경)** 에서 특히 위험했다. CI/CD 서버에는 GitHub 토큰, npm 토큰, 클라우드 배포 키, API 키 등이 환경변수로 저장되는 경우가 많다. 감염된 패키지가 CI/CD 환경에서 설치되면 실제 서비스 배포 권한이나 클라우드 접근 권한까지 탈취될 수 있다.

Trend Micro는 Shai-Hulud가 GitHub API를 통해 피해자의 저장소 권한을 확인하고, 접근 가능한 저장소를 나열하며, TruffleHog를 내려받아 비밀값을 추가로 탐색했다고 분석했다. 여기서 **TruffleHog(코드 저장소나 파일에서 API 키, 토큰, 비밀번호 같은 secret을 찾는 도구)** 는 원래 보안 점검에 쓰이는 도구지만, 공격자는 이를 악용해 더 많은 인증정보를 수집했다.

## 4. Shai-Hulud 2.0

2025년 11월에는 더 공격적인 변종인 Shai-Hulud 2.0도 보고됐다. Unit 42는 2.0 캠페인이 25,000개 이상의 악성 GitHub 저장소와 약 350명의 고유 사용자를 포함할 정도로 더 넓은 범위에서 관찰됐다고 설명했다.

1차 공격과 비교했을 때 2.0은 실행 시점과 파괴성이 달라졌다. 1차 공격은 주로 `postinstall` 단계에서 실행됐지만, 2.0은 `preinstall` 단계에서 실행되는 방식이 보고됐다. **preinstall(패키지가 실제로 설치되기 전에 실행되는 npm 스크립트)** 은 설치 초기에 실행되기 때문에 더 많은 빌드 서버와 자동화 환경에 영향을 줄 수 있다. Unit 42는 2.0에서 `setup_bun.js`, `bun_environment.js` 같은 새 payload 파일이 사용됐고, 인증정보 탈취나 GitHub 저장소 생성에 실패하면 사용자의 홈 디렉터리를 삭제하려는 fallback 동작도 있었다고 분석했다.

## 5. 근본 원인

이 사건의 근본 원인은 오픈소스 생태계의 **신뢰 기반 구조**에 있다. 개발자는 npm에서 패키지를 설치할 때 패키지 이름, 다운로드 수, 기존 평판을 믿고 사용하는 경우가 많다. 하지만 maintainer 계정이 탈취되거나 npm 토큰이 유출되면 공격자는 기존에 신뢰받던 패키지 이름 그대로 악성 버전을 배포할 수 있다.

특히 다음 세 가지가 핵심 문제였다.

첫째, **장기 토큰(long-lived token, 오랫동안 유효한 인증 토큰)** 이 위험했다. 토큰이 한 번 유출되면 공격자는 사용자의 비밀번호를 몰라도 패키지를 배포하거나 수정할 수 있다.

둘째, 패키지 설치 스크립트가 너무 강력했다. `postinstall`이나 `preinstall`은 정상 기능이지만, 패키지를 설치하는 것만으로 로컬 명령이 실행된다는 점에서 공격자에게 좋은 실행 지점이 된다.

셋째, CI/CD 환경에 많은 비밀값이 모여 있었다. 자동 배포를 위해 토큰과 API 키가 저장되어 있는데, 감염된 패키지가 이 환경에서 실행되면 단순 개발자 PC 감염을 넘어 조직 전체의 배포·클라우드 인프라로 피해가 확대될 수 있다.

## 6. 대응 방법

GitHub는 Shai-Hulud 공격 이후 500개 이상의 손상된 패키지를 npm 레지스트리에서 제거했고, 악성코드의 IoC(Indicators of Compromise, 침해 여부를 식별할 수 있는 파일명·해시·도메인·IP·행위 패턴 등 침해 지표)를 포함한 새 패키지 업로드를 차단했다고 밝혔다.

또한 GitHub는 npm 보안을 강화하기 위해 classic token을 폐지하고, 2FA(Two-Factor Authentication, 비밀번호 외 추가 인증을 요구하는 이중 인증)를 더 강하게 적용하며, granular token(권한과 유효기간을 세분화한 토큰)의 수명을 7일로 제한하고, trusted publishing을 확대하겠다고 발표했다. 여기서 **trusted publishing(신뢰 기반 배포, 장기 토큰을 저장하지 않고 GitHub Actions 같은 신뢰된 CI/CD 제공자와 npm을 연결해 패키지를 배포하는 방식)** 은 빌드 시스템 안에 npm 토큰을 오래 저장하지 않아도 된다는 장점이 있다.

개발자와 조직 입장에서는 다음과 같은 조치가 필요하다.

- 감염된 npm 패키지 버전이 사용됐는지 `package-lock.json` 또는 `yarn.lock`을 확인한다.
- 의심되는 npm 토큰, GitHub PAT, 클라우드 키, API 키를 즉시 폐기하고 재발급한다.
- CI/CD 환경변수에 저장된 secret을 점검한다.
- 패키지 설치 시 자동 실행되는 스크립트 사용을 제한한다.
- npm 계정에 2FA를 적용하고, 가능하면 WebAuthn/FIDO2(보안키나 생체인증 기반의 피싱 저항 인증 방식)를 사용한다.
- 장기 토큰 대신 trusted publishing 또는 짧은 수명의 세분화된 토큰을 사용한다.

## 7. 미흡했던 부분

가장 큰 문제는 npm maintainer 계정과 배포 토큰 관리였다. 오픈소스 패키지는 수많은 프로젝트가 의존하기 때문에, 인기 패키지 하나가 감염되면 그 패키지를 직접 설치한 프로젝트뿐 아니라 이를 다시 의존하는 하위 프로젝트까지 영향을 받을 수 있다. 이것이 **공급망 공격(supply chain attack, 직접 대상 시스템을 공격하지 않고 그 시스템이 신뢰하는 외부 소프트웨어·서비스·업체를 침해해 피해를 주는 공격)** 의 위험성이다.

또한 많은 프로젝트가 lockfile(lockfile, 설치되는 의존성의 정확한 버전을 고정해 재현 가능한 설치를 돕는 파일)을 제대로 검토하지 않고 자동 업데이트에 의존한다. 이 경우 악성 버전이 배포되면 개발자나 CI/CD가 이를 빠르게 받아 실행할 수 있다.

마지막으로, 패키지 설치 스크립트에 대한 경계가 부족했다. npm의 설치 스크립트는 편리하지만, 외부 패키지가 내 컴퓨터나 빌드 서버에서 명령을 실행할 수 있다는 뜻이기도 하다. 따라서 설치 스크립트는 단순한 편의 기능이 아니라 보안상 매우 중요한 실행 권한으로 봐야 한다.

## 8. 재발 방지 대책

Shai-Hulud 같은 공격을 막기 위해서는 개발자, maintainer, 조직이 각각 다른 관점에서 대응해야 한다.

개발자는 패키지를 설치할 때 무조건 최신 버전을 따라가기보다 lockfile 변경 내역을 확인해야 한다. 특히 갑자기 새 버전이 배포됐거나, maintainer가 변경됐거나, 설치 스크립트가 추가된 경우에는 주의해야 한다.

Maintainer는 npm 계정에 강력한 2FA를 적용하고, 장기 npm 토큰 사용을 줄여야 한다. 가능하면 trusted publishing을 사용해 CI/CD에 npm 토큰을 직접 저장하지 않는 구조로 바꾸는 것이 좋다.

조직은 CI/CD 환경의 secret을 최소 권한으로 발급해야 한다. 예를 들어 빌드 서버에서 읽기 권한만 필요한데 배포·삭제 권한까지 가진 토큰을 넣어두면, 감염 시 피해가 훨씬 커진다. 또한 secret scanning(secret scanning, 코드나 로그에서 토큰·비밀번호·API 키 같은 민감정보를 자동 탐지하는 점검)을 정기적으로 수행해야 한다.

## 9. 느낀 점

Shai-Hulud 사건은 오픈소스 생태계가 얼마나 편리하면서도 위험할 수 있는지를 보여준다. 개발자는 매일 npm 패키지를 설치하고 업데이트하지만, 그 과정에서 실행되는 코드가 항상 안전하다고 보장할 수는 없다.

특히 이번 사건은 “내가 작성한 코드만 안전하면 된다”는 생각이 부족하다는 점을 보여준다. 현대 개발 환경에서는 내가 직접 작성한 코드보다 외부 라이브러리와 CI/CD, 클라우드 토큰, GitHub 권한이 더 큰 공격 표면이 될 수 있다.

결국 공급망 보안은 선택이 아니라 필수다. 패키지 이름과 다운로드 수만 믿기보다, 어떤 권한으로 설치되고 실행되는지, 어떤 토큰이 노출될 수 있는지, 감염됐을 때 어디까지 확산될 수 있는지를 함께 생각해야 한다.

## 10. 참고 자료

- [GitHub Blog, “Our plan for a more secure npm supply chain”](https://github.blog/security/supply-chain-security/our-plan-for-a-more-secure-npm-supply-chain/)
- [CISA, “Widespread Supply Chain Compromise Impacting npm Ecosystem”](https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem)
- [Unit 42, “Shai-Hulud Worm Compromises npm Ecosystem in Supply Chain Attack”](https://unit42.paloaltonetworks.com/npm-supply-chain-attack/)
- [CERT/CC, “NPM supply chain compromise exposes challenges to securing the ecosystem”](https://www.kb.cert.org/vuls/id/534320)
- [Trend Micro, “What We Know About the NPM Supply Chain Attack”](https://www.trendmicro.com/en_us/research/25/i/npm-supply-chain-attack.html)
