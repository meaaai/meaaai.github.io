---
layout: post
title: "[BugBounty] Fuzzing과 Fuzzer 정리"
date: 2026-05-28 00:00:00 +0900
categories: [BugBounty]
tags: [fuzzing, fuzzer, bugbounty]
---

# Fuzzing과 Fuzzer 정리

처음에는 fuzzing이라는 단어가 단순히 “랜덤 값을 많이 넣어 보는 테스트"라고 생각했으나, 공부해 보니 실제로는 프로그램의 공격 표면을 **자동**으로 두드려 보면서 버그를 찾아내는 꽤 실전적인 보안 테스트 기법이었다.

특히 레드팀 관점에서 보면 퍼징은 단순한 품질 테스트가 아니라, 외부 입력을 받는 지점에서 프로그램이 어디까지 버티고 어디서 무너지는지 확인하는 방법에 가깝다. 공격자가 직접 입력을 조작할 수 있는 파일, 요청, 패킷, 파라미터, API 바디 같은 것들은 모두 fuzzing의 대상이 될 수 있다.

---

## 1. Fuzzing이란?

Fuzzing은 프로그램에 비정상적이거나 예상하지 못한 입력을 자동으로 많이 넣어 보면서 crash, hang, memory error, assertion failure, logic bug 같은 이상 동작을 찾는 테스트 기법이다.

예를 들어 이미지 파일을 읽는 프로그램이 있다고 하면, fuzzer는 정상 PNG 파일을 그대로 넣는 것에서 끝이 아니라, 파일 헤더를 깨뜨리거나, 길이 값을 말도 안 되게 바꾸거나, 중간 데이터를 잘라내거나, 특정 바이트를 반복해서 넣는 식으로 입력을 계속 변형한다. 그 결과 프로그램이 죽거나 멈추면 “이 입력은 뭔가 문제를 일으켰다”고 판단하고 저장한다.

간단히 말하면 fuzzing은 다음과 같은 과정이다.

```text
1. 테스트할 프로그램을 정한다.
2. 입력값을 준비한다.
3. 입력값을 자동으로 변형하거나 생성한다.
4. 프로그램에 계속 넣어 본다.
5. 크래시, 멈춤, 메모리 오류 같은 이상 동작을 찾는다.
6. 문제가 발생한 입력을 저장하고 재현한다.
```

여기서 중요한 점은 fuzzing이 완전히 무작위만은 아니라는 것이다. 초창기에는 랜덤에 가까운 방식도 많았지만, 현재 많이 쓰이는 fuzzer는 code coverage를 확인하면서 더 많은 코드 경로에 도달하는 입력을 우선적으로 발전시킨다. 즉, “아무거나 넣어 보는 것”이 아니라 “프로그램 내부에서 새로운 길을 열어 주는 입력을 찾아가는 것”에 가깝다.

---

## 2. Fuzzer란?

Fuzzer는 fuzzing을 자동으로 수행하는 도구다. fuzzing이 방법이라면, fuzzer는 그 방법을 실제로 실행하는 프로그램이다.

fuzzer가 하는 일은 크게 다섯 가지로 나눌 수 있다.

| 역할 | 설명 |
| --- | --- |
| 입력 생성 | 랜덤 입력이나 변형된 입력을 만든다. |
| 실행 | 대상 프로그램에 입력을 넣고 실행한다. |
| 관찰 | crash, hang, timeout, sanitizer report 등을 확인한다. |
| 피드백 수집 | 어떤 코드 경로가 실행됐는지 coverage를 확인한다. |
| 저장 | 의미 있는 입력, crash input, 재현 파일을 저장한다. |

예를 들어 AFL++ 같은 fuzzer는 seed input을 기반으로 입력을 조금씩 바꾸고, 새로운 code coverage를 만들어낸 입력은 corpus에 저장한다. 이후 그 입력을 다시 변형해서 더 깊은 코드 경로를 찾는다. 이런 방식 때문에 fuzzing은 오래 돌릴수록 점점 더 흥미로운 입력을 발견할 가능성이 생긴다.

---

## 3. Fuzzing에서 자주 나오는 용어

처음 fuzzing을 보면 용어가 꽤 낯설다. 기본 용어를 먼저 잡아 두면 fuzzer 문서를 읽을 때 훨씬 편하다.

| 용어 | 의미 |
| --- | --- |
| Target | 테스트 대상 프로그램, 라이브러리, 함수 |
| Seed | fuzzer에게 처음 제공하는 기본 입력 |
| Corpus | fuzzing에 사용할 입력 파일 모음 |
| Mutation | 기존 입력을 조금씩 변형하는 것 |
| Coverage | 프로그램 실행 중 도달한 코드 범위 |
| Crash | 프로그램이 비정상 종료되는 현상 |
| Hang | 프로그램이 멈추거나 너무 오래 실행되는 현상 |
| Harness | 특정 함수나 프로그램을 fuzzer가 호출할 수 있게 감싼 테스트 코드 |
| Sanitizer | 메모리 오류, undefined behavior 등을 잡는 동적 분석 도구 |
| Reproducer | 버그를 재현할 수 있는 입력 파일 |
| Triage | 발견된 crash가 어떤 원인인지 분류하고 우선순위를 매기는 과정 |

특히 sanitizer는 fuzzing에서 매우 중요하다. 프로그램이 겉으로는 죽지 않아도 내부에서는 buffer overflow, use-after-free, memory leak 같은 문제가 발생할 수 있다. AddressSanitizer, UndefinedBehaviorSanitizer 같은 도구를 함께 사용하면 이런 문제를 더 잘 잡을 수 있다.

---

## 4. Fuzzing으로 찾을 수 있는 문제

Fuzzing은 다양한 유형의 버그를 찾을 수 있다.

| 유형 | 설명 | 보안 관점 |
| --- | --- | --- |
| Crash | 프로그램이 비정상 종료됨 | DoS 가능성 |
| Hang | 입력 처리 중 무한 루프나 긴 지연 발생 | 서비스 마비 가능성 |
| Buffer Overflow | 버퍼 범위를 넘어 데이터가 쓰임 | RCE 가능성 |
| Use-After-Free | 해제된 메모리를 다시 사용함 | 메모리 손상, RCE 가능성 |
| Integer Overflow | 정수 계산이 범위를 넘음 | 크기 계산 오류, 메모리 오류 가능성 |
| Parser Bug | 파일/프로토콜 해석 과정 오류 | 입력 기반 취약점 가능성 |
| Assertion Failure | 내부 검증 조건 실패 | 로직 오류 또는 예외 처리 문제 |

레드팀 입장에서는 crash가 났다고 바로 exploit이 되는 것은 아니다. 하지만 crash는 “이 입력을 처리하는 과정에 안전하지 않은 지점이 있다”는 신호가 될 수 있다. 그래서 crash input을 줄이고, 재현하고, 디버거로 원인을 확인하는 과정이 중요하다.

---

## 5. Fuzzing의 종류

### 5.1 Black-box fuzzing

Black-box fuzzing은 프로그램 내부 구조를 모르는 상태에서 입력과 출력만 보고 테스트하는 방식이다.

소스코드가 없어도 시도할 수 있다는 장점이 있다. 대신 어떤 코드 경로가 실행됐는지 알기 어렵기 때문에 효율이 떨어질 수 있다. 상용 프로그램, 폐쇄형 바이너리, 외부 서비스처럼 내부를 알 수 없는 대상을 테스트할 때 쓰인다.

레드팀 관점에서는 실제 공격자가 소스코드를 모르는 경우가 많기 때문에 black-box 접근이 현실적인 경우도 많다.

### 5.2 White-box fuzzing

White-box fuzzing은 소스코드나 내부 구조를 알고 있는 상태에서 테스트하는 방식이다.

프로그램의 조건문, 경로, 제약 조건을 분석해 더 정교한 입력을 만들 수 있다. 이론적으로는 깊은 코드 경로까지 도달하기 좋지만, 분석 비용이 크고 속도가 느려질 수 있다.

### 5.3 Gray-box fuzzing

Gray-box fuzzing은 black-box와 white-box의 중간 방식이다.

프로그램 내부를 완전히 분석하지는 않지만, code coverage 같은 피드백을 이용한다. 새로운 코드 경로를 실행한 입력은 가치 있는 입력으로 보고 저장한 뒤, 다시 변형한다. AFL++, libFuzzer, Honggfuzz 같은 도구들이 대표적으로 이 방식에 가깝다.

현실적으로 가장 많이 쓰이는 방식이기도 하다. 내부를 전부 이해하지 않아도 되고, 단순 랜덤보다 훨씬 효율적이기 때문이다.

### 5.4 Mutation-based fuzzing

Mutation-based fuzzing은 기존 입력을 조금씩 바꿔서 새로운 입력을 만드는 방식이다.

예를 들어 정상 PNG 파일 하나를 seed로 넣으면, fuzzer가 일부 바이트를 바꾸거나 삭제하거나 반복하거나 길이를 바꾼다.

```text
normal.png
→ 헤더 일부 변경
→ 길이 필드 변경
→ 특정 바이트 반복
→ 중간 데이터 삭제
→ 매우 큰 값 삽입
```

장점은 시작하기 쉽다는 것이다. 정상 입력 몇 개만 있어도 fuzzing을 시작할 수 있다. 단점은 입력 형식이 복잡할수록 초반 파서에서 바로 걸러질 가능성이 높다는 점이다.

### 5.5 Generation-based fuzzing

Generation-based fuzzing은 입력을 처음부터 생성하는 방식이다.

예를 들어 JSON, XML, SQL, JavaScript처럼 구조가 있는 입력은 아무렇게나 깨뜨리면 문법 오류로 바로 탈락할 수 있다. 이때 문법을 어느 정도 알고 있는 fuzzer가 구조에 맞는 입력을 생성하면 더 깊은 코드까지 도달할 수 있다.

### 5.6 Structure-aware / Grammar-based fuzzing

Structure-aware fuzzing은 입력의 구조를 이해하고 fuzzing하는 방식이다.

예를 들어 PDF, PNG, JavaScript, HTTP 요청처럼 형식이 복잡한 데이터는 내부 구조를 유지해야 의미 있는 테스트가 된다. Grammar-based fuzzing은 입력 문법을 정의해 두고, 그 문법에 맞는 변형을 만든다.

단순히 랜덤한 바이트를 던지는 것보다 준비 과정은 더 필요하지만, 복잡한 parser나 interpreter를 테스트할 때 훨씬 유리할 수 있다.

### 5.7 Coverage-guided fuzzing

Coverage-guided fuzzing은 현재 가장 많이 언급되는 방식 중 하나다.

핵심은 “새로운 코드 경로를 실행한 입력을 더 가치 있게 본다”는 것이다. fuzzer는 입력을 넣은 뒤 어떤 코드가 실행됐는지 확인하고, 기존에 가보지 못한 경로를 실행한 입력을 저장한다. 이후 그 입력을 다시 변형해서 더 깊은 경로로 들어간다.

이 방식은 레드팀과도 잘 맞는다. 단순히 프로그램이 죽는지만 보는 것이 아니라, 입력을 통해 도달 가능한 내부 기능과 예외 처리 지점을 점점 넓혀 갈 수 있기 때문이다.

---

## 6. 여러 fuzzer 비교

### 6.1 AFL++

AFL++는 가장 유명한 coverage-guided fuzzer 중 하나다. 기존 AFL을 발전시킨 도구로, C/C++ 프로그램이나 파일 입력 기반 프로그램을 fuzzing할 때 자주 언급된다.

AFL++의 기본 흐름은 다음과 같다.

```text
1. afl-cc 같은 컴파일러 wrapper로 대상 프로그램을 빌드한다.
2. seed input 디렉터리를 준비한다.
3. afl-fuzz를 실행한다.
4. fuzzer가 입력을 변형하면서 대상 프로그램을 반복 실행한다.
5. crash와 hang이 발견되면 output 디렉터리에 저장된다.
```

AFL++의 장점은 입문 자료가 많고, coverage-guided fuzzing의 기본 구조를 이해하기 좋다는 점이다. 소스코드가 있는 대상은 계측해서 fuzzing할 수 있고, binary-only target에 대해서도 QEMU mode, FRIDA mode 같은 선택지가 있다.

레드팀 관점에서는 파일 파서, CLI 도구, 네이티브 바이너리의 입력 처리 부분을 테스트할 때 특히 유용해 보였다.

### 6.2 libFuzzer

libFuzzer는 LLVM 프로젝트에서 제공하는 in-process coverage-guided fuzzer다.

AFL++가 프로그램을 반복 실행하는 느낌에 가깝다면, libFuzzer는 테스트 대상 라이브러리와 fuzzer를 함께 링크해서 특정 함수에 입력을 계속 넣는 방식에 가깝다. 보통 `LLVMFuzzerTestOneInput`이라는 entrypoint를 작성한다.

간단한 형태는 다음과 같다.

```c
#include <stdint.h>
#include <stddef.h>

int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    // data와 size를 이용해 테스트 대상 함수를 호출한다.
    return 0;
}
```

libFuzzer는 라이브러리 함수나 parser 함수를 직접 테스트할 때 좋다. 속도도 빠른 편이고 sanitizer와 함께 쓰기 좋다. 대신 대상 함수를 fuzzing하기 위해 harness를 직접 작성해야 하는 경우가 많다.

레드팀 관점에서는 오픈소스 라이브러리의 특정 함수나 parser를 깊게 테스트할 때 활용하기 좋아 보였다.

### 6.3 Jackalope

Jackalope는 Google Project Zero에서 공개한 binary coverage-guided fuzzer다. 특징은 black-box binary를 대상으로 할 수 있다는 점이다.

공식 설명에 따르면 Jackalope는 customizable, distributed, coverage-guided fuzzer이며 Windows, macOS, Linux, Android의 binary fuzzing을 지원한다. 내부적으로 TinyInst 기반 instrumentation을 활용한다.

Jackalope가 흥미로운 이유는 소스코드가 없는 프로그램을 대상으로 할 때다. 레드팀이나 취약점 연구에서는 항상 소스코드가 있는 환경만 만나는 것이 아니기 때문에, binary-only fuzzing 도구는 실전성이 크다.

### 6.4 Honggfuzz

Honggfuzz는 security-oriented, feedback-driven, evolutionary fuzzer다.

코드 커버리지를 기반으로 버그를 찾고, multi-process, multi-threaded 실행과 persistent fuzzing을 지원한다. AFL++나 libFuzzer만큼 입문 자료가 많이 보이는 편은 아니지만, 보안 연구 목적의 fuzzer로 꾸준히 언급된다.

레드팀 관점에서는 다양한 실행 방식과 분석 옵션을 활용해 fuzzing 환경을 직접 조정하고 싶을 때 후보가 될 수 있다.

### 6.5 OSS-Fuzz

OSS-Fuzz는 fuzzer 하나라기보다는 continuous fuzzing infrastructure에 가깝다.

오픈소스 프로젝트를 대상으로 fuzzing을 지속적으로 수행하고, 발견된 문제를 관리하는 인프라다. 내부적으로 libFuzzer, AFL++, Honggfuzz 같은 도구들과 sanitizer를 함께 사용할 수 있다.

즉, AFL++나 libFuzzer가 “직접 실행하는 도구”라면 OSS-Fuzz는 “오픈소스 프로젝트를 계속 fuzzing하는 플랫폼”에 가깝다.

레드팀 관점에서는 당장 공격 도구라기보다는, 실제 보안 생태계에서 fuzzing이 어떻게 대규모로 운영되는지 이해하는 데 의미가 있다. 오픈소스 구성 요소를 많이 사용하는 환경에서는 이런 지속적 fuzzing 결과가 공급망 보안과도 연결된다.

### 6.6 google/fuzzing

미션 관련 링크에 있는 `google/fuzzing`은 fuzzer 자체라기보다는 fuzzing 학습 자료 모음에 가깝다.

Introduction to fuzzing, structure-aware fuzzing, fuzz target 작성법, glossary 같은 문서들이 있어 fuzzing을 처음 공부할 때 유용하다. 여러 fuzzer를 실습하기 전에 기본 개념을 잡는 용도로 보면 된다.

---

## 7. Fuzzer별 차이 정리

| 도구 | 유형 | 주요 특징 | 잘 맞는 상황 |
| --- | --- | --- | --- |
| AFL++ | Coverage-guided fuzzer | AFL 기반, 다양한 mode, crash/hang 관리 | C/C++ 프로그램, 파일 파서, CLI 도구 |
| libFuzzer | In-process coverage-guided fuzzer | LLVM 기반, 함수 단위 fuzzing, sanitizer와 궁합 좋음 | 라이브러리 함수, parser API 테스트 |
| Jackalope | Binary coverage-guided fuzzer | black-box binary 대상, TinyInst 기반 | 소스코드 없는 바이너리 분석 |
| Honggfuzz | Feedback-driven evolutionary fuzzer | 보안 지향, coverage 기반, multi-process/thread | 커스터마이징이 필요한 보안 연구 |
| OSS-Fuzz | Continuous fuzzing infrastructure | 오픈소스 프로젝트 장기 fuzzing | 대규모 지속 fuzzing, 공급망 보안 |
| google/fuzzing | 학습 자료 저장소 | 튜토리얼, 용어, 예제 문서 | fuzzing 개념 학습 |

---

## 8. Fuzzing을 실제로 한다면 어떤 순서로 할까?

이번에 개념을 정리하면서 fuzzing은 도구 이름보다 흐름이 더 중요하다고 느꼈다.

```text
1. Target 선정
   - 어떤 프로그램이나 함수를 테스트할지 정한다.

2. 입력 지점 파악
   - 파일, stdin, socket, API body, command-line argument 등 입력이 들어가는 곳을 찾는다.

3. Seed corpus 준비
   - 정상 입력 파일을 몇 개 준비한다.

4. Harness 작성
   - 필요한 경우 fuzzer가 대상 함수를 호출할 수 있게 감싼다.

5. Instrumentation / Sanitizer 적용
   - coverage와 메모리 오류를 관찰할 수 있게 빌드한다.

6. Fuzzer 실행
   - AFL++, libFuzzer, Honggfuzz 등 적절한 도구를 선택한다.

7. Crash 수집
   - crash, hang, sanitizer report를 확인한다.

8. Reproducer 최소화
   - 문제를 일으키는 입력을 줄이고 재현성을 확인한다.

9. Root cause 분석
   - 디버거, 로그, sanitizer report를 통해 원인을 찾는다.

10. 보고서 작성 또는 패치 검증
   - BugBounty나 내부 보안 테스트라면 영향도와 재현 절차를 정리한다.
```

레드팀에서는 1번과 2번이 특히 중요하다고 생각한다. 아무 프로그램이나 fuzzing하는 것보다, 실제 공격자가 조작할 수 있는 입력 지점을 먼저 찾는 것이 더 실전적이다. 예를 들어 업로드 기능, 파일 변환 기능, 압축 해제 기능, 이미지 처리 기능, 문서 미리보기 기능, 네트워크 패킷 처리 기능은 모두 흥미로운 fuzzing 대상이 될 수 있다.

---

## 9. 레드팀 관점에서 본 Fuzzing

레드팀은 단순히 도구를 실행하는 역할이 아니라, 공격자가 어떤 경로로 시스템을 건드릴 수 있는지 생각해야 한다. 그런 점에서 fuzzing은 공격 표면을 넓게 훑어보는 데 도움이 된다.

내가 이해한 레드팀 관점의 fuzzing 활용은 다음과 같다.

첫째, 입력 검증이 약한 지점을 찾을 수 있다. 사용자가 직접 조작할 수 있는 파일, 파라미터, 요청 값, 프로토콜 메시지를 계속 변형하면 개발자가 예상하지 못한 예외 상황이 드러날 수 있다.

둘째, crash를 통해 취약점 후보를 찾을 수 있다. 모든 crash가 보안 취약점은 아니지만, memory corruption과 연결된다면 영향도가 커질 수 있다. 특히 C/C++ 기반 프로그램에서는 buffer overflow, use-after-free 같은 문제가 실제 exploit 가능성으로 이어질 수 있다.

셋째, 취약점 재현 능력을 기를 수 있다. BugBounty나 레드팀 보고서에서는 “어쩌다 한 번 죽었다”가 아니라, 같은 입력으로 반복 재현되고 원인이 설명되어야 한다. fuzzing은 crash input을 남겨 주기 때문에 재현 중심의 분석 습관을 들이기 좋다.

넷째, 자동화된 공격 표면 탐색과 연결된다. 레드팀 업무에서는 수동 분석도 중요하지만, 반복 가능한 자동화도 중요하다. fuzzing은 사람이 직접 만들기 어려운 수많은 입력을 자동으로 생성한다는 점에서 자동화된 취약점 탐색의 기본기라고 느꼈다.

다섯째, 방어 관점까지 같이 이해할 수 있다. 레드팀은 공격만 아는 것보다, 왜 이런 버그가 생기고 어떻게 줄일 수 있는지도 알아야 한다. fuzzing 결과를 보고 sanitizer, input validation, parser hardening, CI 기반 지속 테스트 같은 방어 방법까지 연결할 수 있다.

---

## 10. 미션 관련 링크 정리

### AFL++

- 링크: https://github.com/AFLplusplus/AFLplusplus
- 의미: 대표적인 coverage-guided fuzzer
- 핵심: 입력 변형, coverage 피드백, crash/hang 저장
- 공부 포인트: `afl-cc`, `afl-fuzz`, seed corpus, crashes/hangs 디렉터리

### Jackalope

- 링크: https://github.com/googleprojectzero/Jackalope
- 의미: Google Project Zero의 binary coverage-guided fuzzer
- 핵심: black-box binary fuzzing, TinyInst 기반 instrumentation
- 공부 포인트: 소스코드 없는 바이너리를 대상으로 fuzzing할 수 있다는 점

### google/fuzzing

- 링크: https://github.com/google/fuzzing/tree/master
- 의미: fuzzing 학습 자료 저장소
- 핵심: fuzzing 개념, fuzz target 작성법, structure-aware fuzzing, glossary
- 공부 포인트: 도구 사용 전에 기본 개념을 잡기 좋음

---

## 11. 느낀점

처음에는 fuzzing을 “이상한 값을 많이 넣어서 운 좋게 버그를 찾는 방식” 정도로 생각했다. 그런데 정리하면서 보니 실제 fuzzing은 생각보다 체계적이었다. 특히 coverage-guided fuzzing은 단순한 랜덤 테스트가 아니라, 프로그램 내부에서 새롭게 도달한 경로를 기준으로 입력을 계속 발전시키는 방식이어서 어렵고 흥미로웠다.

레드팀 관점에서도 fuzzing은 꽤 매력적인 기술이라고 느꼈다. 웹 취약점처럼 요청 하나를 보고 바로 결과를 확인하는 방식과는 다르지만, 프로그램이 입력을 처리하는 깊은 부분까지 자동으로 파고들 수 있다는 점이 좋았다. 특히 파일 파서, 압축 해제, 이미지 처리, 문서 변환처럼 외부 입력을 받지만 내부 동작이 복잡한 기능에서는 사람이 직접 모든 케이스를 생각하기 어렵기 때문에 fuzzing이 강력해질 수밖에 없다고 느꼈다.

또 하나 인상 깊었던 점은 crash를 찾는 것보다 그 이후가 더 중요하다는 점이다. crash input을 얻었다고 끝나는 것이 아니라, 그 입력이 왜 문제를 일으켰는지 재현하고, 줄이고, 디버거로 원인을 확인하고, 실제 보안 영향도를 판단해야 한다. 이 과정은 레드팀 보고서 작성 능력과도 바로 연결된다고 생각한다.

이번 미션을 통해 fuzzing은 단순한 도구 사용법이 아니라 “입력 기반으로 프로그램을 무너뜨리는 사고방식”에 가깝다는 생각이 들었다. 레드팀을 목표로 한다면 웹 요청만 보는 것이 아니라, 파일·프로토콜·바이너리·라이브러리처럼 다양한 입력 표면을 볼 수 있어야 하고, fuzzing은 그 감각을 기르는 데 좋은 출발점이 될 것 같다.

---

## 12. 앞으로의 계획

먼저 google/fuzzing 문서를 읽으면서 fuzz target, corpus, sanitizer, coverage 같은 기본 개념을 더 정확히 익힐 계획이다. 용어가 익숙해져야 AFL++나 libFuzzer 문서를 읽을 때 흐름이 끊기지 않을 것 같다.

그다음에는 작은 C 프로그램을 직접 만들어 AFL++로 fuzzing해 보고 싶다. 예를 들어 파일에서 문자열을 읽어 특정 형식을 파싱하는 간단한 프로그램을 만들고, seed input을 넣은 뒤 crash가 발생하는 과정을 확인해 보는 식이다. 이때 AddressSanitizer를 함께 켜서 단순 crash뿐 아니라 메모리 오류가 어떻게 보고되는지도 확인해 볼 계획이다.

이후에는 libFuzzer로 함수 단위 fuzzing을 실습해 보고 싶다. AFL++가 프로그램 전체를 대상으로 하는 느낌이라면, libFuzzer는 특정 parser 함수나 라이브러리 함수를 직접 테스트하는 데 좋아 보였기 때문이다. harness를 작성하는 연습도 같이 할 수 있을 것 같다.

마지막으로는 레드팀 관점에서 fuzzing 결과를 보고서로 정리하는 연습을 해 보고 싶다. 단순히 “크래시가 났다”가 아니라, 입력 조건, 재현 방법, crash 원인, 영향도, 완화 방안까지 정리해야 실제 BugBounty나 보안 테스트 산출물에 가까워질 것 같다.

---

## 13. 참고 자료

- AFL++ GitHub: https://github.com/AFLplusplus/AFLplusplus
- AFL++ Binary-only Targets: https://aflplus.plus/docs/fuzzing_binary-only_targets/
- Jackalope GitHub: https://github.com/googleprojectzero/Jackalope
- TinyInst GitHub: https://github.com/googleprojectzero/TinyInst
- Google fuzzing docs: https://github.com/google/fuzzing/tree/master
- Google fuzzing - Intro to fuzzing: https://github.com/google/fuzzing/blob/master/docs/intro-to-fuzzing.md
- Google fuzzing - Structure-aware fuzzing: https://github.com/google/fuzzing/blob/master/docs/structure-aware-fuzzing.md
- LLVM libFuzzer docs: https://llvm.org/docs/LibFuzzer.html
- Honggfuzz GitHub: https://github.com/google/honggfuzz
- OSS-Fuzz docs: https://google.github.io/oss-fuzz/
