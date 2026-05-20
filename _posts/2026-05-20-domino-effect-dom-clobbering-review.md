---
title: "[논문 리뷰] The DOMino Effect: DOM Clobbering Gadget 자동 탐지와 Exploit 생성"
date: 2026-05-20 18:30:00 +0900
categories: [논문/컨퍼런스]
tags: [paper-review, web-security, dom-clobbering, xss, usenix-security, hulk]
---

# 관심 분야 논문 리뷰 - The DOMino Effect

이번에 리뷰할 논문은 2025년 USENIX Security Symposium에 발표된 **「The DOMino Effect: Detecting and Exploiting DOM Clobbering Gadgets via Concolic Execution with Symbolic DOM」**이다.

이 논문은 웹 환경에서 발생할 수 있는 **DOM Clobbering** 취약점을 자동으로 탐지하고, 실제 exploit 가능성까지 분석하는 연구이다. 논문에서 제안한 도구의 이름은 **Hulk**이며, 공격자가 제어할 수 있는 HTML 구조를 **Symbolic DOM**으로 모델링해 exploit 가능한 HTML markup을 생성하는 것이 핵심이다.

## 논문 정보

| 구분 | 내용 |
| --- | --- |
| 제목 | The DOMino Effect: Detecting and Exploiting DOM Clobbering Gadgets via Concolic Execution with Symbolic DOM |
| 학회 | 34th USENIX Security Symposium (USENIX Security 25) |
| 연도 | 2025 |
| 저자 | Zhengyu Liu, Theo Lee, Jianjia Yu, Zifeng Kang, Yinzhi Cao |
| 소속 | Johns Hopkins University |
| 키워드 | DOM Clobbering, Web Security, XSS, Gadget Detection, Concolic Execution, Symbolic DOM, Hulk |
| 논문 링크 | <https://www.usenix.org/conference/usenixsecurity25/presentation/liu-zhengyu> |

## 논문을 고른 이유

나는 취약점 연구원과 레드팀 분야에 관심이 있기 때문에, 단순히 방어 기술만 다루는 논문보다는 실제 취약점이 어떻게 발견되고, 어떤 방식으로 공격 가능성이 검증되는지를 보여주는 논문을 읽어보고 싶었다.

레드팀이나 취약점 연구원은 시스템을 공격자의 관점에서 바라보고, 보안상 약점이 실제로 악용될 수 있는지 판단하는 능력이 중요하다고 생각한다. 이 논문은 DOM Clobbering이라는 웹 취약점을 단순히 개념적으로 설명하는 데 그치지 않고, 실제 웹사이트에서 취약한 지점을 자동으로 탐지하고 exploit 가능성까지 분석한다는 점에서 흥미로웠다.

처음에는 DOM Clobbering이라는 개념이 조금 낯설었다. XSS나 SQL Injection처럼 자주 들어본 취약점은 아니었기 때문이다. 그런데 내용을 살펴보니 HTML과 JavaScript가 서로 맞물리는 방식에서 생기는 문제였고, 공격자가 직접 `<script>`를 삽입하지 않아도 기존 JavaScript 코드의 흐름을 이용해 공격으로 이어질 수 있다는 점이 흥미로웠다.

특히 레드팀이나 취약점 연구에서는 “취약점이 존재한다”에서 끝나는 것이 아니라, 그 취약점이 실제 환경에서 어떤 조건을 만족해야 악용될 수 있는지까지 판단해야 한다고 생각한다. 이 논문은 그런 과정을 자동화하려는 연구라서, 내가 관심 있는 진로와도 잘 맞는다고 느꼈다.

## 문제 정의

DOM Clobbering은 DOM 요소의 `id`나 `name` 속성이 JavaScript 변수나 객체 속성 조회와 충돌하면서 발생할 수 있는 웹 공격 기법이다. 공격자는 겉으로 보기에는 단순한 HTML markup을 삽입하지만, 브라우저와 JavaScript가 이를 해석하는 과정에서 기존 코드의 흐름이 바뀔 수 있다. 논문에서는 이러한 흐름이 XSS나 CSRF 같은 보안 문제로 이어질 수 있다고 설명한다.

이 공격에서 중요한 개념이 **gadget**이다. gadget은 이미 웹사이트나 라이브러리 안에 존재하는 JavaScript 코드 조각이다. 공격자는 새로운 악성 스크립트를 직접 넣는 대신, 기존 코드 중 공격에 이용될 수 있는 부분을 찾아서 자신의 HTML markup이 위험한 sink까지 흘러가도록 만든다.

기존 연구인 **TheThing**은 DOM Clobbering gadget을 찾기 위한 도구였지만, 미리 정의된 HTML payload 목록에 많이 의존했다. 그래서 복잡한 조건을 만족해야 하는 새로운 gadget은 놓칠 수 있었다. 또한 JavaScript는 동적으로 코드가 생성되거나 객체 alias, 즉 서로 다른 변수 이름이 같은 객체를 가리키는 상황이 자주 발생하는데, 정적 분석만으로는 이런 흐름을 따라가기 어렵다는 한계가 있었다.

읽으면서 느낀 점은, 이 문제가 단순히 “이름이 겹쳐서 위험하다” 정도로 끝나는 문제가 아니라는 것이다. 실제 공격이 되려면 HTML 구조, JavaScript 실행 흐름, 조건문, sink 도달 여부까지 전부 맞아야 한다. 결국 DOM Clobbering은 웹 구조를 꽤 깊게 이해해야 분석할 수 있는 취약점이라는 생각이 들었다.

## 핵심 아이디어

이 논문의 핵심 아이디어는 공격자가 삽입할 수 있는 HTML을 단순 문자열로 보지 않고, 브라우저가 해석하는 **DOM Tree 구조**로 모델링하는 것이다. 저자들은 이를 **Symbolic DOM**이라고 부른다.

기존 방식처럼 미리 만들어 둔 payload를 하나씩 대입하는 것이 아니라, gadget 안에서 요구되는 DOM 관련 조건을 수집하고, 그 조건을 만족하는 HTML markup을 자동으로 생성한다.

Hulk는 크게 세 가지 흐름으로 동작한다.

### 1. Gadget Detection

웹 페이지의 JavaScript를 실행하면서 공격자가 조작할 수 있는 DOM 값이 위험한 sink까지 흘러가는지 추적한다. 이때 **dynamic taint tracking**을 사용해서 값의 흐름을 따라간다.

정적 분석만으로는 동적으로 생성되는 코드나 실행 중에 바뀌는 객체 관계를 따라가기 어렵기 때문에, 논문은 실제 실행을 기반으로 한 동적 분석을 사용한다.

### 2. Exploit Generation

단순히 “source에서 sink까지 흐름이 있다”고 판단하는 것만으로는 충분하지 않다. DOM Clobbering exploit은 일반적인 문자열 payload와 다르게, 태그 이름, 속성, DOM 요소 간 관계, JavaScript에서 접근되는 방식까지 맞아야 한다.

이 단계에서 Hulk는 **Symbolic DOM**과 **concolic execution**을 사용해 실제 조건을 만족하는 HTML markup을 생성한다.

### 3. Gadget Verification

생성된 HTML markup을 실제 페이지에 주입해 보고, 공격자가 의도한 payload가 sink까지 도달하는지 확인한다. 즉, Hulk는 탐지, exploit 생성, 검증까지 하나의 흐름으로 연결한다.

이 부분이 인상적이었다. 많은 취약점 분석 도구는 “의심 지점”을 많이 찾아내는 데 그치는 경우가 있는데, 이 논문은 실제 exploit 가능성까지 확인하려고 한다. 보안 분석에서 오탐이 많으면 결국 사람이 다시 확인해야 하기 때문에, 검증 단계까지 자동화하려는 시도가 현실적인 문제를 잘 건드렸다고 느꼈다.

## 방법론 / 시스템 설명

Hulk의 첫 번째 단계는 JavaScript 코드에서 DOM Clobbering source와 sink 사이의 흐름을 찾는 것이다. source는 공격자가 영향을 줄 수 있는 DOM 요소나 속성이고, sink는 XSS나 CSRF처럼 실제 보안 문제로 이어질 수 있는 위험한 함수 호출 지점이다.

이 과정에서 Hulk는 JavaScript 프로그램을 실제로 실행하면서 taint tracking을 수행한다. 논문에서 제시한 Google API client library 사례에서도 기존 정적 분석 도구는 복잡한 조건과 동적 값 흐름 때문에 gadget을 찾지 못했지만, Hulk는 실행 중 값을 추적해 이를 해결했다.

두 번째 단계에서는 수집한 조건을 바탕으로 HTML markup을 생성한다. DOM Clobbering exploit은 단순히 문자열 하나를 맞추는 것이 아니라, 태그 이름, 속성, DOM 요소 간 관계, JavaScript에서 접근되는 방식까지 맞아야 한다. 이 때문에 저자들은 Symbolic DOM을 이용해 DOM 구조 자체를 제약 조건으로 표현한다.

세 번째 단계에서는 생성된 markup이 실제로 sink까지 도달하는지 검증한다. 이 검증 과정이 있기 때문에 Hulk는 단순 후보를 많이 던지는 도구라기보다는, 실제 exploit 가능한 gadget을 확인하는 도구에 가깝다.

개인적으로 이 흐름이 취약점 연구와 잘 맞는다고 느꼈다. 취약점 연구는 단순히 “여기 이상해 보인다”에서 끝나는 것이 아니라, 그 이상한 지점이 실제로 어떤 조건에서 공격으로 이어지는지를 끝까지 확인해야 한다. Hulk는 그 과정을 자동화하려고 했다는 점에서 의미가 크다.

## 평가 및 결과 해석

논문은 Hulk를 **Tranco Top 5,000 웹사이트**에 적용해 평가했다.

평가 결과는 다음과 같다.

| 항목 | 결과 |
| --- | --- |
| 식별한 DOM Clobbering source | 310,163개 |
| 식별한 sink function call | 485,102개 |
| taint flow | 34,040개 |
| 생성한 후보 HTML markup | 1,741,254개 |
| 최종 검증된 unique gadget | 497개 |

최종적으로 검증된 497개의 gadget 중 378개는 XSS, 90개는 CSRF, 26개는 open redirection, 3개는 storage manipulation으로 이어질 수 있었다.

숫자만 봐도 규모가 꽤 크지만, 더 인상적인 부분은 단순히 연구용 예제에서만 동작한 것이 아니라는 점이다. Hulk는 Google Client API, Google Closure, MathJax, Webpack 같은 널리 쓰이는 client-side library에서도 zero-day gadget을 찾았다.

논문은 Hulk가 발견한 결과 중 일부가 패치되었고, end-to-end exploit 평가를 통해 Jupyter Notebook/JupyterLab, Canvas LMS, Hackmd.io, Cocalc 같은 실제 웹 애플리케이션에서 HTML injection이 stored XSS로 이어질 수 있음을 보였다. 또한 이 연구 결과로 19개의 CVE가 부여되었다고 설명한다.

TheThing과 비교한 결과도 중요하다.

| 데이터셋 | Hulk | TheThing |
| --- | --- | --- |
| Tranco Top 500 | 33개 gadget 검증 | 6개 gadget 검증 |
| Known Gadgets dataset | 12개 중 5개 탐지 | 12개 중 4개 탐지 |

두 도구 모두 검증 단계를 거치기 때문에 false positive는 없었지만, Hulk가 false negative를 줄이는 데 더 좋은 결과를 보였다.

처음에는 “497개를 찾았다”는 결과가 가장 눈에 띄었는데, 읽다 보니 더 중요한 점은 따로 있었다. 이 논문은 단순히 많이 찾은 것이 아니라, 기존 방식이 왜 놓쳤는지를 설명하고 그 원인을 해결하려고 했다. 미리 정해진 payload에 의존하면 복잡한 조건을 만족하는 gadget을 놓칠 수 있고, 정적 분석만으로는 JavaScript의 동적 특성을 따라가기 어렵다. Hulk는 이 두 문제를 Symbolic DOM과 dynamic taint tracking으로 풀어낸다.

## 한계점

물론 이 논문이 모든 문제를 해결한 것은 아니다.

첫 번째 한계는 **taint tracking의 범위**이다. Hulk는 직접 접근되는 DOM 요소나 client-side storage를 통한 흐름은 추적할 수 있지만, 간접 접근까지 모두 다루지는 못한다. 예를 들어 taint된 요소가 다른 non-tainted 요소 안에 들어가 있고, 나중에 `innerHTML` 같은 방식으로 간접 접근되는 경우에는 taint가 전파되지 않을 수 있다. 논문은 이 부분을 성능과 효과 사이의 trade-off로 설명한다.

두 번째 한계는 **WebAssembly를 통한 흐름을 추적하지 못한다는 점**이다. Hulk가 사용하는 Jalangi2가 WebAssembly를 지원하지 않기 때문이다. 웹 환경에서 WebAssembly 사용이 늘어나고 있다는 점을 생각하면, 이 부분은 앞으로 더 중요해질 수 있다.

세 번째는 **constraint solving의 한계**이다. Hulk는 Z3 solver를 사용하는데, 정규표현식이나 `replace`, `indexOf`, `split` 같은 문자열 관련 연산이 복잡하게 얽히면 제한 시간 안에 해를 찾지 못할 수 있다. DOM Clobbering exploit은 HTML 구조와 문자열 조건이 함께 얽히는 경우가 많기 때문에, 이 부분은 실제 탐지 성능에도 영향을 줄 수 있다고 생각한다.

마지막으로, **DOM Clobbering 자체의 완화가 쉽지 않다**는 점도 인상적이었다. 단순히 `id`나 `name` 속성을 전부 제거하면 보안상으로는 안전해질 수 있지만, 실제 웹 기능이 깨질 수 있다. 논문에서도 DOM Clobbering 기능을 완전히 비활성화하면 상당수 웹 페이지가 깨질 수 있고, sanitizer 역시 기능성과 보안 사이에서 균형을 잡기 어렵다고 설명한다.

이 부분을 보면서 웹 보안은 “위험한 기능을 없애면 된다”로 끝나지 않는다는 생각이 들었다. 실제 서비스에서는 보안, 기능, 호환성을 동시에 고려해야 하기 때문에 방어가 더 어려워진다.

## 느낀 점

이 논문을 읽으면서 가장 크게 느낀 점은, 취약점 연구가 단순히 공격 코드를 만드는 일이 아니라는 것이다. 실제로는 브라우저가 HTML을 어떻게 해석하는지, JavaScript가 객체와 속성을 어떻게 조회하는지, 데이터가 어떤 흐름으로 sink까지 도달하는지를 하나씩 따라가야 한다.

특히 DOM Clobbering은 겉으로 보기에는 단순한 HTML 삽입처럼 보일 수 있지만, 실제로는 기존 JavaScript 코드의 흐름을 이용하는 공격이다. 그래서 공격자가 새로운 스크립트를 직접 넣지 않아도, 웹 애플리케이션 안에 이미 존재하는 gadget을 이용해 XSS나 CSRF로 이어질 수 있다. 이 점이 꽤 흥미로웠다.

또 하나 인상적이었던 점은 “취약점 탐지”와 “exploit 가능성 검증”을 분리하지 않았다는 것이다. 보안 도구가 취약한 지점을 많이 알려줘도 실제로 공격이 가능한지 확인할 수 없다면 분석자의 부담은 여전히 크다. Hulk는 탐지한 흐름에 대해 실제 HTML markup을 생성하고 검증까지 수행한다는 점에서, 취약점 연구자나 레드팀 관점에서 더 실용적인 접근이라고 느꼈다.

읽기 전에는 DOM Clobbering을 잘 몰랐기 때문에 조금 막연했지만, 읽고 나서는 웹 취약점 분석에서 DOM과 JavaScript 동작 원리를 이해하는 것이 얼마나 중요한지 알게 되었다. 앞으로 웹 보안을 공부할 때 XSS, CSRF 같은 대표적인 취약점만 보는 것이 아니라, 브라우저 내부 동작이나 클라이언트 사이드 라이브러리의 구조도 함께 봐야겠다고 생각했다.

## 앞으로 해볼 것

### 더 읽어볼 논문

이 논문을 읽고 나서 가장 먼저 이어서 읽어보고 싶은 것은 **TheThing 관련 연구**이다. Hulk가 계속 비교 대상으로 삼는 기존 도구이기 때문에, TheThing이 어떤 방식으로 DOM Clobbering gadget을 탐지했고 왜 predefined payload 방식에 한계가 생겼는지 직접 확인해 보고 싶다.

또한 DOM Clobbering과 연결되는 **HTML sanitizer 관련 연구**도 읽어보고 싶다. 논문에서 언급된 것처럼 HTML injection을 막기 위해 sanitizer를 사용하더라도 `id`나 `name` 속성이 남으면 DOM Clobbering이 가능할 수 있다. 그래서 sanitizer가 어떤 기준으로 HTML을 허용하거나 제거하는지 공부하면, 웹 취약점 분석을 더 잘 이해할 수 있을 것 같다.

### 직접 재현해 보고 싶은 것

바로 실제 서비스에 테스트하는 것은 위험하므로, 먼저 로컬 환경에서 아주 작은 HTML/JavaScript 예제를 만들어 DOM Clobbering이 어떻게 발생하는지 확인해 보고 싶다. 예를 들어 `document.x` 같은 속성 조회가 특정 DOM 요소의 `name`이나 `id`와 충돌할 때 어떤 값이 반환되는지 직접 실험해 보면 개념이 더 잘 잡힐 것 같다.

그 다음에는 간단한 sink를 만들어서, 공격자가 넣은 HTML 요소가 어떤 조건을 만족해야 위험한 함수까지 도달하는지 확인해 보고 싶다. 논문에서는 Symbolic DOM과 concolic execution으로 이 과정을 자동화했지만, 처음에는 손으로 작은 예제를 따라가 보는 것이 이해에 더 도움이 될 것 같다.

### 프로젝트로 발전시킨다면

이 내용을 프로젝트로 확장한다면, 간단한 DOM Clobbering 탐지 도구를 만들어 보고 싶다. 처음부터 Hulk처럼 복잡한 taint tracking이나 Symbolic DOM을 구현하기는 어렵겠지만, 특정 패턴의 `window` 또는 `document` 속성 접근과 위험한 sink를 찾아주는 작은 정적 분석 도구부터 시작할 수 있을 것 같다.

또 다른 방향으로는 웹 개발자 입장에서 DOM Clobbering을 줄이기 위한 체크리스트를 만들어 볼 수 있을 것 같다. 예를 들어 전역 객체 접근을 피하고, DOM 요소를 가져올 때 명확한 API를 사용하고, sanitizer 설정에서 `id`와 `name` 속성을 어떻게 처리해야 하는지 정리하는 식이다. 공격 기법을 이해하는 것도 중요하지만, 결국 실무에서는 개발자가 실수하지 않도록 안전한 가이드를 제공하는 것도 중요하다고 생각한다.

마지막으로, 레드팀 관점에서는 “어떤 웹 기능이 DOM Clobbering에 노출되기 쉬운가”를 따로 정리해 보고 싶다. markdown editor, rich-text editor, notebook 서비스처럼 사용자가 HTML과 비슷한 입력을 넣을 수 있는 기능은 특히 더 주의해서 봐야 할 것 같다.

이 논문을 계기로 웹 취약점 분석을 단순한 입력값 테스트가 아니라, 브라우저와 JavaScript 실행 흐름을 **함께** 보는 관점으로 확장해 보고 싶다.

## 참고 링크

- USENIX Security 25 논문 페이지: <https://www.usenix.org/conference/usenixsecurity25/presentation/liu-zhengyu>
- Hulk GitHub Repository: <https://github.com/jackfromeast/TheHulk>
- DOM Clobbering Gadget Collection: <https://github.com/jackfromeast/dom-clobbering-collection>
