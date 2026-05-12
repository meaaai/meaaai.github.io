---
title: "[OverTheWire Bandit] 리눅스 기반 시스템 탐색 및 데이터 분석"
date: 2026-05-12 23:00:00 +0900
categories: [Linux, Bandit]
tags: [linux, ssh, bandit, overthewire, terminal]
---

## Level 0

**목표**: SSH를 사용하여 게임에 로그인하기

**풀이**

```bash
ssh -p 2220 bandit0@bandit.labs.overthewire.org
# password: bandit0
```

| 부분 | 의미 |
| --- | --- |
| `ssh` | 원격 서버 접속 |
| `-p 2220` | 2220번 포트 사용 |
| `bandit0` | 접속할 사용자 이름 |
| `bandit.labs.overthewire.org` | Bandit 서버 주소 |

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-01.png' | relative_url }})

위와 같이 환영 창이 뜨면 성공

## Level 0 → 1

**레벨 목표**

이번 레벨의 비밀번호는 홈 디렉터리에 위치한 readme라는 파일에 저장된다.

따라서 현재 디렉터리에 어떤 파일이 있는지 확인한 뒤, 해당 파일의 내용을 읽으면 된다.

**풀이**

문제에서 `readme` 파일이 홈 디렉터리에 있다고 했으므로, 먼저 현재 위치의 파일 목록을 확인했다.

실행 결과 `readme` 파일이 있는 것을 확인할 수 있었다.

이제 `cat` 명령어를 사용하여 `readme` 파일의 내용을 출력했다.

그 결과 다음 레벨인 `bandit1`에 접속할 때 필요한 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-02.png' | relative_url }})

| 명령어 | 설명 |
| --- | --- |
| `ssh` | 원격 서버에 접속 |
| `ls` | 현재 디렉터리의 파일 목록 확인 |
| `cat` | 파일 내용을 터미널에 출력 |

## Level 1 → 2

다음 레벨의 비밀번호는 홈 디렉터리에 있는 `-` 파일 안에 저장되어 있다.

처음에는 `-`가 기호처럼 보여서 헷갈릴 수 있지만, 이 문제에서는 실제 파일 이름이 `-`이다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-03.png' | relative_url }})

level 0 → 1에서와 같이 ls 명령어와 cat 명령어를 그대로 사용할 시, 위처럼 파일 내용이 보이지 않는다.

문제가 무엇인지 찾아본 결과 `-`는 리눅스 명령어에서 표준 입력을 의미할 수 있기 때문에 단순히 `cat -`로 입력하면 안 된다는 것을 알게 되었다.

따라서 현재 디렉터리에 있는 파일임을 명확히 하기 위해 `./-` 형태로 입력했다. 여기서 `./`는 **현재 디렉터리**라는 뜻이다.

## Level 2 → 3

다음 레벨의 비밀번호는 홈 디렉터리에 있는 `spaces in this filename` 파일 안에 저장되어 있다.

파일명이 공백을 포함하고 있었고, `--`로 시작했기 때문에 단순히 `cat "--spaces in this filename--"`를 입력하면 `cat`이 파일명을 옵션으로 해석했다.

이를 해결하기 위해 현재 디렉터리의 파일임을 명확히 나타내는 `./`를 붙여 실행했다.

```bash
cat "./--spaces in this filename--"
```

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-04.png' | relative_url }})

## Level 3 → 4

이번 문제에서는 다음 레벨의 비밀번호가 홈 디렉터리에 바로 있는 파일이 아니라, `inhere`라는 디렉터리 안의 숨겨진 파일에 저장되어 있었다.

이전 레벨들에서는 파일이 현재 위치, 즉 홈 디렉터리에 바로 있었기 때문에 `cat` 명령어로 바로 파일 내용을 확인할 수 있었다.

하지만 이번 문제에서는 파일이 `inhere` 디렉터리 안에 있었기 때문에, 먼저 해당 디렉터리로 이동해야 했다.

`cd`는 change directory의 줄임말로, 현재 작업 위치를 다른 디렉터리로 이동할 때 사용하는 명령어이다.

`cd inhere`를 입력하면 현재 위치가 홈 디렉터리에서 `inhere` 디렉터리 안으로 바뀐다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-05.png' | relative_url }})

```bash
bandit3@bandit:~/inhere$
```

위 상태는 홈 디렉터리 안의 `inhere` 디렉터리로 이동했다는 뜻이다.

## Level 4 → 5

이번 문제에서는 다음 레벨의 비밀번호가 `inhere` 디렉터리 안에 있는 유일하게 사람이 읽을 수 있는 파일에 저장되어 있었다.

먼저 `ls` 명령어를 사용해 현재 위치에 어떤 파일이나 디렉터리가 있는지 확인했고, `inhere` 디렉터리가 있는 것을 확인했다. 이전 문제와 마찬가지로 비밀번호가 홈 디렉터리에 바로 있는 것이 아니라 특정 디렉터리 안에 있었기 때문에, `cd inhere` 명령어를 사용해 해당 디렉터리로 이동했다.

`inhere` 디렉터리 안에는 여러 개의 파일이 있었기 때문에, 어떤 파일이 사람이 읽을 수 있는 파일인지 확인해야 했다. 이를 위해 `file ./-*` 명령어를 사용했다. `file` 명령어는 파일의 종류를 확인할 때 사용하며, 실행 결과 중 `ASCII text`로 표시되는 파일이 사람이 읽을 수 있는 파일이다.

이후 해당 파일을 `cat` 명령어로 출력하여 다음 레벨의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-06.png' | relative_url }})

## Level 5 → 6

이번 문제에서는 다음 레벨의 비밀번호가 `inhere` 디렉터리 아래 어딘가에 있는 파일에 저장되어 있었다. 이전 문제와 달리 파일이 바로 보이는 위치에 있는 것이 아니라 여러 하위 디렉터리 중 하나에 있었고, 문제에서 제시한 조건에 맞는 파일을 찾아야 했다.

조건은 사람이 읽을 수 있고, 크기가 1033바이트이며, 실행할 수 없는 파일이었다. 여러 디렉터리 안에서 특정 조건을 만족하는 파일을 찾아야 했기 때문에 `find` 명령어를 사용했다.

```bash
find . -type f -size 1033c ! -executable
```

| 부분 | 의미 |
| --- | --- |
| `find .` | 현재 디렉터리 아래에서 찾기 |
| `-type f` | 일반 파일만 찾기 |
| `-size 1033c` | 크기가 1033바이트인 것 찾기 |
| `! -executable` | 실행 가능한 파일은 제외 |

명령어 실행 결과 조건에 맞는 파일 경로를 확인할 수 있었고, 해당 파일을 `cat`으로 출력하여 다음 레벨의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-07.png' | relative_url }})

## Level 6 → 7

이번 문제에서는 다음 레벨의 비밀번호가 서버의 어딘가에 저장되어 있다고 했다. 이전 문제에서는 `inhere` 디렉터리 아래에서만 파일을 찾으면 됐지만, 이번에는 특정 디렉터리가 주어지지 않았기 때문에 서버 전체를 대상으로 검색해야 했다.

문제에서 제시한 조건은 소유자가 `bandit7`, 그룹이 `bandit6`, 크기가 33바이트인 파일이었다. 따라서 `find /`를 사용해 서버 전체에서 조건에 맞는 파일을 검색했다.

```bash
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

| 부분 | 의미 |
| --- | --- |
| `find /` | 서버 전체에서 검색 |
| `-type f` | 일반 파일만 검색 |
| `-user bandit7` | 소유자가 `bandit7`인 파일 검색 |
| `-group bandit6` | 그룹이 `bandit6`인 파일 검색 |
| `-size 33c` | 크기가 33바이트인 파일 검색 |
| `2>/dev/null` | 권한 오류 메시지를 화면에 출력하지 않음 |

명령어 실행 결과 조건에 맞는 파일 경로를 확인할 수 있었고, 해당 파일을 `cat` 명령어로 출력하여 다음 레벨의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-08.png' | relative_url }})

## Level 7 → 8

이번 문제에서는 다음 레벨의 비밀번호가 `data.txt` 파일 안에 저장되어 있고, `millionth`라는 단어 옆에 있다고 했다.

이전 문제들에서는 파일의 위치나 조건을 기준으로 비밀번호 파일을 찾았다면, 이번에는 이미 파일 이름이 주어져 있었기 때문에 파일 안에서 특정 단어를 검색하면 되는 문제였다.

`data.txt` 파일 전체를 직접 확인하면 내용이 많아 원하는 줄을 찾기 어렵기 때문에, 문자열 검색에 사용하는 `grep` 명령어를 사용했다.

```bash
grep "millionth" data.txt
```

`grep`은 파일 안에서 특정 문자열이 포함된 줄을 찾아 출력하는 명령어이다.

실행 결과 `millionth`가 포함된 줄이 출력되었고, 그 옆에 있는 문자열을 통해 다음 레벨의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-09.png' | relative_url }})

## Level 8 → 9

이번 문제에서는 다음 레벨의 비밀번호가 `data.txt` 파일 안에 저장되어 있으며, 그중 한 번만 나타나는 유일한 텍스트 라인을 찾아야 했다.

이전 문제에서는 특정 단어가 포함된 줄을 찾기 위해 `grep`을 사용했지만, 이번 문제에서는 특정 단어가 주어진 것이 아니라 중복되지 않는 줄을 찾아야 했다. 따라서 파일 내용을 정렬한 뒤, 한 번만 등장하는 줄을 출력하는 방식으로 접근했다.

```bash
sort data.txt | uniq -u
```

`sort`는 파일 내용을 정렬하는 명령어이다. `uniq`는 중복된 줄을 처리할 때 사용하는 명령어인데, `uniq -u`를 사용하면 중복되지 않고 한 번만 나타나는 줄만 출력할 수 있다.

단, `uniq`는 연속된 중복 줄을 기준으로 동작하기 때문에 먼저 `sort`로 같은 내용의 줄들을 서로 붙게 만들어야 한다.

명령어 실행 결과 한 줄의 문자열이 출력되었고, 이 값이 다음 레벨의 비밀번호였다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-10.png' | relative_url }})

## Level 9 → 10

이번 문제에서는 다음 레벨의 비밀번호가 `data.txt` 파일 안에 저장되어 있었지만, 파일 안에는 사람이 바로 읽기 어려운 내용이 많이 포함되어 있었다.

문제에서 비밀번호는 몇 안 되는 사람이 읽을 수 있는 문자열 중 하나이며, 앞에 여러 개의 `=` 문자가 붙는다고 했다. 따라서 먼저 `strings` 명령어를 사용해 `data.txt` 안에서 사람이 읽을 수 있는 문자열만 추출했다.

```bash
strings data.txt | grep "="
```

`strings`는 파일 안에서 사람이 읽을 수 있는 문자열만 골라 출력하는 명령어이다. 여기에 `grep "="`을 함께 사용하여 문제에서 힌트로 준 `=` 문자가 포함된 줄만 확인했다.

실행 결과 `=` 문자가 포함된 줄을 찾을 수 있었고, 그 뒤에 있는 문자열을 통해 다음 레벨의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-11.png' | relative_url }})

## Level 10 → 11

이번 문제에서는 다음 레벨의 비밀번호가 `data.txt` 파일 안에 저장되어 있었지만, 일반 텍스트가 아니라 Base64로 인코딩된 형태였다.

이전 문제들에서는 파일 안에서 특정 문자열을 찾거나 사람이 읽을 수 있는 문자열을 추출하는 방식이었다면, 이번에는 주어진 데이터 자체를 원래 형태로 변환해야 했다.

이를 위해 `base64 -d data.txt` 명령어를 사용했다. `base64`는 Base64 형식의 데이터를 다룰 때 사용하는 명령어이고, `-d` 옵션은 decode의 의미로 인코딩된 데이터를 다시 원래 내용으로 변환한다.

명령어 실행 결과 인코딩되어 있던 문자열이 디코딩되었고, 그 결과로 다음 레벨의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-12.png' | relative_url }})

## Level 11 → 12

이번 문제에서는 다음 레벨의 비밀번호가 `data.txt` 파일 안에 저장되어 있었지만, 모든 소문자와 대문자가 알파벳 기준으로 13자리씩 회전된 상태였다.

이 방식은 ROT13이라고 부르며, 알파벳을 13칸씩 밀어서 다른 문자로 바꾸는 방식이다. 예를 들어 `a`는 `n`으로, `n`은 다시 `a`로 변환된다. ROT13은 같은 변환을 한 번 더 적용하면 원래 문자열로 돌아오는 특징이 있다.

따라서 `data.txt`의 내용을 읽은 뒤, `tr` 명령어를 사용해 알파벳을 ROT13 방식으로 다시 변환했다.

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

`tr`은 문자를 다른 문자로 변환할 때 사용하는 명령어이다. 이번 문제에서는 대문자와 소문자를 모두 ROT13 규칙에 맞게 변환하기 위해 `'A-Za-z'`를 `'N-ZA-Mn-za-m'`로 바꾸었다.

명령어 실행 결과 회전되어 있던 문자열이 원래 형태로 변환되었고, 다음 레벨의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-13.png' | relative_url }})

## Level 12 → 13

이번 문제에서는 다음 레벨의 비밀번호가 `data.txt` 파일에 저장되어 있었지만, 단순한 텍스트 파일이 아니라 반복적으로 압축된 파일의 헥스덤프 형태였다.

먼저 홈 디렉터리에서 바로 작업하지 않고 `/tmp` 아래에 임시 작업 디렉터리를 만들었다. 이번 문제는 파일을 복사하고, 이름을 바꾸고, 압축을 여러 번 해제해야 했기 때문에 원본 파일이 있는 홈 디렉터리에서 바로 작업하는 것보다 별도의 작업 공간을 만드는 것이 안전했다.

```bash
mktemp -d
cd /tmp/tmp.Wa7kHBsujA
```

`mktemp -d`는 임시 디렉터리를 자동으로 생성해 주는 명령어이다. 직접 디렉터리 이름을 정할 수도 있지만, `mktemp -d`를 사용하면 다른 사용자와 겹치기 어려운 이름의 디렉터리를 만들 수 있다.

이후 홈 디렉터리에 있던 `data.txt` 파일을 현재 작업 디렉터리로 복사했다.

```bash
cp ~/data.txt .
```

문제에서 `data.txt`가 헥스덤프라고 했기 때문에, 먼저 `xxd -r` 명령어를 사용해 헥스덤프를 원래 바이너리 파일 형태로 복구했다.

```bash
xxd -r data.txt > data
```

`xxd -r`에서 `-r`은 reverse의 의미로, 헥스덤프 형태의 데이터를 원래 파일로 되돌릴 때 사용한다. 복구된 파일은 바로 읽을 수 없었기 때문에 `file` 명령어로 파일 형식을 확인했다.

```
file data
```

확인 결과 처음에는 gzip 압축 파일이었다. 따라서 파일 이름을 `data.gz`로 바꾼 뒤 `gzip -d`로 압축을 해제했다.

```bash
mv data data.gz
gzip -d data.gz
```

압축을 한 번 풀어도 바로 비밀번호가 나오지 않았기 때문에, 다시 `file` 명령어로 새로 나온 파일의 형식을 확인했다. 이후에는 파일 형식에 따라 gzip이면 `gzip -d`, bzip2이면 `bzip2 -d`, tar 아카이브이면 `tar -xf`를 사용해 계속 압축을 해제했다.

진행 중 한 번은 `file` 결과가 gzip이었는데 실수로 bzip2 파일처럼 이름을 바꾸고 `bzip2 -d`를 실행했다. 이때 `not a bzip2 file` 오류가 발생했고, 다시 파일명을 `.gz`로 바꾼 뒤 `gzip -d`를 사용해 올바르게 압축을 풀었다.

전체적으로 진행한 압축 해제 흐름은 다음과 같다.

```bash
gzip -d data.gz
bzip2 -d data.bz2
gzip -d data.gz
tar -xf data.tar
tar -xf data5.tar
bzip2 -d data6.bz2
tar -xf data6.tar
gzip -d data8.gz
```

즉, 헥스덤프 복구를 제외하고 압축 또는 아카이브 해제는 총 8번 진행했다.

마지막으로 `file data8`을 실행했을 때 `ASCII text`로 표시되었다. 이는 사람이 읽을 수 있는 텍스트 파일이라는 의미이므로, `cat` 명령어로 내용을 출력했다.

```bash
cat data8
```

그 결과 다음 레벨인 `bandit13`에 접속할 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-14.png' | relative_url }})

## Level 13 → 14

이번 문제에서는 다음 레벨의 비밀번호를 직접 찾는 것이 아니라, `bandit14` 계정으로 로그인할 수 있는 개인 SSH 키를 제공받는 방식이었다.

문제 설명에 따르면 `bandit14`의 비밀번호는 `/etc/bandit_pass/bandit14`에 저장되어 있지만, 이 파일은 `bandit14` 사용자만 읽을 수 있다. 따라서 먼저 `bandit14` 계정으로 접속해야 했고, 이를 위해 홈 디렉터리에 있는 `sshkey.private` 파일을 사용해야 했다.

먼저 `bandit13`에 접속한 뒤 파일 목록을 확인했다.

```bash
ls -al
```

실행 결과 `HINT` 파일과 `sshkey.private` 파일이 있는 것을 확인할 수 있었다.

`sshkey.private`는 `bandit14`로 접속할 때 사용할 개인 SSH 키이고, `HINT` 파일에는 이 레벨을 진행할 때 주의해야 할 내용이 들어 있었다.

처음에는 `bandit13` 서버 안에서 바로 `bandit14`로 접속하려고 했다.

```bash
ssh -i ./sshkey.private -p 2220 bandit14@localhost
```

하지만 다음과 같은 오류가 발생했다.

```
You are trying to log into this SSH server with a password on port 2220 from localhost.
Connecting from localhost is blocked to conserve resources.
```

처음에는 포트 문제인지, 키 파일 문제인지 헷갈렸기 때문에 AI에게 오류 메시지의 의미를 물어보며 원인을 확인했다. 오류 메시지를 해석해 보니, 현재 OverTheWire 서버에서는 한 레벨에서 다른 레벨로 `localhost`를 통해 직접 접속하는 방식이 막혀 있다는 것을 알 수 있었다.

이후 포트 번호를 빼고 접속도 시도해 보았다.

```bash
ssh -i ./sshkey.private bandit14@localhost
```

하지만 이번에는 22번 포트로 접속하려고 하면서 다음과 같은 안내가 나왔다.

```
You are trying to log into this SSH server on port 22, which is not intended.
```

즉, Bandit 서버에 접속할 때는 2220번 포트를 사용해야 하지만, 서버 내부에서 `localhost`로 직접 다음 레벨에 접속하는 방식은 현재 차단되어 있었다. 그래서 `HINT` 파일의 안내처럼 로그아웃한 뒤, 키 파일을 로컬 환경으로 가져와서 접속하는 방식으로 해결했다.

먼저 Bandit 서버에서 나와 내 MacBook 터미널로 돌아왔다.

```
exit
```

그다음 `scp` 명령어를 사용해 Bandit 서버에 있는 `sshkey.private` 파일을 내 MacBook으로 복사했다.

```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .
```

`scp`는 원격 서버와 로컬 컴퓨터 사이에서 파일을 복사할 때 사용하는 명령어이다.

주의할 점은 `ssh`에서는 포트 옵션으로 소문자 `-p`를 사용하지만, `scp`에서는 대문자 `-P`를 사용한다는 것이다.

키 파일을 가져온 뒤에는 SSH에서 사용할 수 있도록 권한을 변경했다.

```bash
chmod 600 sshkey.private
```

개인 SSH 키는 권한이 너무 열려 있으면 보안상 SSH에서 사용을 거부할 수 있기 때문에, `chmod 600`으로 소유자만 읽고 쓸 수 있도록 설정했다.

마지막으로 MacBook 터미널에서 가져온 키 파일을 사용해 `bandit14`에 접속했다.

```bash
ssh -i ./sshkey.private -p 2220 bandit14@bandit.labs.overthewire.org
```

이번에는 정상적으로 `bandit14` 계정에 접속할 수 있었다.

접속 후에는 `bandit14` 사용자만 읽을 수 있는 비밀번호 파일을 확인했다.

```bash
cat /etc/bandit_pass/bandit14
```

그 결과 `bandit14`의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-15.png' | relative_url }})

이번 문제를 통해 비밀번호 대신 SSH 개인 키를 사용해 로그인하는 방법을 배웠고, `localhost`가 상황에 따라 의미하는 대상이 달라질 수 있다는 점도 알게 되었다. 또한 오류 메시지를 그냥 넘기지 않고 읽어 보면, 왜 접속이 실패했는지 해결 방향을 찾을 수 있다는 것을 경험했다.

| 명령어 | 설명 |
| --- | --- |
| `ls -al` | 현재 디렉터리의 파일을 자세히 확인 |
| `cat HINT` | 힌트 파일 내용 확인 |
| `ssh -i ./sshkey.private -p 2220 bandit14@localhost` | 개인 키로 `localhost` 접속을 시도했지만 차단됨 |
| `exit` | Bandit 서버 접속 종료 |
| `scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .` | 서버의 개인 키 파일을 로컬 MacBook으로 복사 |
| `chmod 600 sshkey.private` | 개인 키 파일 권한 설정 |
| `ssh -i ./sshkey.private -p 2220 bandit14@bandit.labs.overthewire.org` | 개인 키를 사용해 `bandit14`로 접속 |
| `cat /etc/bandit_pass/bandit14` | `bandit14` 비밀번호 파일 확인 |

## Level 14 → 15

이번 문제에서는 다음 레벨의 비밀번호를 파일에서 직접 찾는 것이 아니라, 현재 레벨의 비밀번호를 `localhost`의 30000번 포트에 제출해야 했다.

먼저 `cat /etc/bandit_pass/bandit14` 명령어로 현재 레벨인 `bandit14`의 비밀번호를 확인했다. 이 비밀번호를 30000번 포트로 전송하면 다음 레벨의 비밀번호를 받을 수 있다.

이를 위해 `nc` 명령어를 사용했다.

```bash
nc localhost 30000
```

`nc`는 netcat의 줄임말로, 특정 호스트와 포트에 연결해 데이터를 주고받을 때 사용하는 명령어이다. 여기서 `localhost`는 현재 접속 중인 Bandit 서버 자기 자신을 의미하고, `30000`은 문제에서 지정한 포트 번호이다.

`nc localhost 30000`을 실행한 뒤 현재 레벨의 비밀번호를 입력하자, 서버가 응답으로 다음 레벨인 `bandit15`의 비밀번호를 출력했다.

또는 파이프를 사용해 다음과 같이 한 번에 처리할 수도 있다.

```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

이 명령어는 `bandit14`의 비밀번호를 출력한 뒤, 그 결과를 바로 `nc`로 넘겨 30000번 포트에 제출하는 방식이다.

| 명령어 | 설명 |
| --- | --- |
| `cat /etc/bandit_pass/bandit14` | 현재 레벨인 `bandit14`의 비밀번호를 확인한다. |
| `nc localhost 30000` | `localhost`의 30000번 포트에 연결한다. |
| `cat /etc/bandit_pass/bandit14 | nc localhost 30000` | 현재 레벨의 비밀번호를 30000번 포트로 바로 전송한다. |

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-16.png' | relative_url }})

## Level 15 → 16

이번 문제에서는 다음 레벨의 비밀번호를 얻기 위해 현재 레벨의 비밀번호를 `localhost`의 30001번 포트에 제출해야 했다.

이전 문제에서는 `nc` 명령어를 사용해 포트에 데이터를 전송했지만, 이번 문제에서는 SSL/TLS 암호화를 사용해야 한다는 조건이 추가되었다. 따라서 일반적인 TCP 연결이 아니라 SSL/TLS 연결을 만들 수 있는 `openssl s_client` 명령어를 사용했다.

먼저 현재 레벨의 비밀번호를 확인했다.

```bash
cat /etc/bandit_pass/bandit15
```

이후 다음 명령어로 `localhost`의 30001번 포트에 SSL/TLS 방식으로 연결했다.

```bash
openssl s_client -connect localhost:30001
```

연결 후 현재 레벨의 비밀번호를 입력하자 다음 레벨인 `bandit16`의 비밀번호가 출력되었다.

또는 파이프를 사용해 현재 레벨의 비밀번호를 바로 제출할 수도 있다.

```bash
cat /etc/bandit_pass/bandit15 | openssl s_client -connect localhost:30001 -quiet
```

`openssl s_client`는 SSL/TLS를 사용하는 서버에 접속할 때 사용하는 명령어이다. 이번 문제에서는 30001번 포트가 SSL/TLS 연결을 요구했기 때문에 `nc` 대신 이 명령어를 사용해야 했다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-17.png' | relative_url }})

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-18.png' | relative_url }})

## Level 16 → 17

이번 문제에서는 다음 레벨의 자격 증명을 얻기 위해 현재 레벨의 비밀번호를 `localhost`의 특정 포트에 제출해야 했다.
다만 포트 번호가 바로 주어지지 않고, `31000`번부터 `32000`번 사이에서 직접 찾아야 했다.

먼저 어떤 포트가 열려 있는지 확인하기 위해 `nmap` 명령어를 사용했다.

```bash
nmap -sV -p 31000-32000 localhost
```

`nmap`은 포트 스캔을 할 때 사용하는 명령어이다. 이번 문제에서는 단순히 열려 있는 포트만 찾는 것이 아니라, 그중 SSL/TLS를 사용하는 포트도 구분해야 했기 때문에 `-sV` 옵션을 함께 사용했다. `-sV`는 열린 포트에서 어떤 서비스가 동작하는지 확인하는 옵션이다.

스캔 결과 여러 포트가 열려 있었고, 그중 `31790` 포트가 `ssl/unknown`으로 표시되었다.

```
31046/tcp open  echo
31518/tcp open  ssl/echo
31691/tcp open  echo
31790/tcp open  ssl/unknown
31960/tcp open  echo
```

`echo` 서버는 내가 보낸 값을 그대로 돌려주는 서버이므로 정답이 아니었다. 반면 `31790` 포트는 SSL/TLS를 사용하면서, 잘못된 입력에 대해 `Wrong! Please enter the correct current password.`라는 응답을 반환했다. 따라서 이 포트가 현재 비밀번호를 제출해야 하는 포트라고 판단했다.

이후 `openssl s_client`를 사용해 현재 레벨의 비밀번호를 `31790` 포트에 제출했다.

```bash
cat /etc/bandit_pass/bandit16 | openssl s_client -connect localhost:31790 -quiet
```

그 결과 다음 레벨의 비밀번호가 아니라, `bandit17`로 접속할 수 있는 private key가 출력되었다. 처음에는 출력된 키를 직접 복사해 `bandit17.key` 파일로 만들려고 했지만, 키의 일부만 저장되어 접속이 실패했다. SSH private key는 반드시 `-----BEGIN RSA PRIVATE KEY-----`부터 `-----END RSA PRIVATE KEY-----`까지 전체가 저장되어야 한다.

그래서 이번에는 Bandit 서버 안에서 키 전체를 임시 파일로 저장한 뒤, 로컬 MacBook으로 가져오는 방식으로 해결했다.

먼저 임시 디렉터리를 만들었다.

```bash
tmpdir=$(mktemp -d)
echo $tmpdir
```

생성된 경로는 다음과 같았다.

```
/tmp/tmp.NBKvclqxOB
```

이후 `openssl s_client`의 출력 결과 중 private key 부분만 추출해 임시 디렉터리에 저장했다.

```bash
cat /etc/bandit_pass/bandit16 | openssl s_client -connect localhost:31790 -quiet 2>/dev/null | sed -n '/-----BEGIN RSA PRIVATE KEY-----/,/-----END RSA PRIVATE KEY-----/p' >"$tmpdir/bandit17.key"
```

여기서 `sed -n '/-----BEGIN RSA PRIVATE KEY-----/,/-----END RSA PRIVATE KEY-----/p'`는 출력 내용 중 private key의 시작 줄부터 끝 줄까지만 골라내는 역할을 한다.

또한 `2>/dev/null`은 `openssl` 실행 중 출력되는 불필요한 오류 메시지를 숨기기 위해 사용했다.

저장된 키가 정상적인지 확인했다.

```bash
head -n 1 "$tmpdir/bandit17.key"
tail -n 1 "$tmpdir/bandit17.key"
```

첫 줄이 `-----BEGIN RSA PRIVATE KEY-----`, 마지막 줄이 `-----END RSA PRIVATE KEY-----`로 확인되면 private key가 정상적으로 저장된 것이다.

이후 Bandit 서버에서 나와 MacBook 터미널에서 `scp` 명령어를 사용해 키 파일을 로컬로 복사했다.

```bash
scp -P 2220 bandit16@bandit.labs.overthewire.org:/tmp/tmp.NBKvclqxOB/bandit17.key .
```

`scp`는 원격 서버와 로컬 컴퓨터 사이에서 파일을 복사할 때 사용하는 명령어이다. `ssh`에서는 포트 옵션으로 소문자 `-p`를 사용하지만, `scp`에서는 대문자 `-P`를 사용한다.

키 파일을 가져온 뒤에는 SSH에서 사용할 수 있도록 권한을 변경했다.

```bash
chmod 600 bandit17.key
```

개인 키 파일은 권한이 너무 열려 있으면 SSH에서 사용을 거부할 수 있기 때문에, `chmod 600`으로 소유자만 읽고 쓸 수 있도록 설정했다.

마지막으로 가져온 키 파일을 사용해 `bandit17`에 접속했다.

```bash
ssh -i ./bandit17.key -p 2220 bandit17@bandit.labs.overthewire.org
```

정상적으로 접속되면 프롬프트가 `bandit17@bandit:~$` 형태로 바뀐다.

이번 문제를 통해 포트 범위를 스캔해 열려 있는 서비스를 찾는 방법, SSL/TLS 포트에 데이터를 제출하는 방법, 그리고 출력된 private key를 파일로 저장해 다음 레벨 접속에 사용하는 방법을 배웠다.

| 명령어 | 설명 |
| --- | --- |
| `nmap -sV -p 31000-32000 localhost` | 31000~32000 범위에서 열려 있는 포트와 서비스 정보를 확인 |
| `cat /etc/bandit_pass/bandit16` | 현재 레벨인 `bandit16`의 비밀번호 확인 |
| `openssl s_client -connect localhost:31790 -quiet` | SSL/TLS를 사용하는 31790번 포트에 연결 |
| `mktemp -d` | 임시 작업 디렉터리 생성 |
| `echo $tmpdir` | 생성된 임시 디렉터리 경로 확인 |
| `sed -n '/-----BEGIN RSA PRIVATE KEY-----/,/-----END RSA PRIVATE KEY-----/p'` | 출력 내용 중 private key 부분만 추출 |
| `2>/dev/null` | 오류 메시지를 화면에 출력하지 않도록 처리 |
| `head -n 1 파일명` | 파일의 첫 줄 확인 |
| `tail -n 1 파일명` | 파일의 마지막 줄 확인 |
| `scp -P 2220 ...` | Bandit 서버의 파일을 로컬 MacBook으로 복사 |
| `chmod 600 bandit17.key` | SSH 개인 키 파일 권한 설정 |
| `ssh -i ./bandit17.key -p 2220 bandit17@bandit.labs.overthewire.org` | private key를 사용해 `bandit17` 접속 |

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-19.png' | relative_url }})

## Level 17 → 18

이번 문제에서는 홈 디렉터리에 `passwords.old`와 `passwords.new` 두 개의 파일이 주어졌다. 다음 레벨의 비밀번호는 `passwords.new` 파일 안에 있으며, 두 파일을 비교했을 때 변경된 유일한 줄이라고 했다.

이전 문제들에서는 특정 파일을 찾거나 포트에 비밀번호를 제출하는 방식이었다면, 이번 문제에서는 두 파일의 차이를 비교해야 했다. 이를 위해 `diff` 명령어를 사용했다.

```bash
diff passwords.old passwords.new
```

`diff`는 두 파일의 차이점을 비교할 때 사용하는 명령어이다. 실행 결과에서 `<`로 시작하는 줄은 첫 번째 파일인 `passwords.old`에 있던 내용이고, `>`로 시작하는 줄은 두 번째 파일인 `passwords.new`에 있는 내용이다.

문제에서 비밀번호는 `passwords.new`에 있다고 했으므로, 출력 결과 중 `>`로 표시된 줄을 확인했다. 해당 줄이 다음 레벨인 `bandit18`에 접속할 비밀번호였다.

문제 참고 사항에 따르면 이 레벨을 해결한 뒤 `bandit18`에 로그인할 때 `Byebye!` 메시지가 보일 수 있는데, 이는 다음 레벨인 `bandit19`와 관련된 내용이다.

| 명령어 | 설명 |
| --- | --- |
| `ls` | 홈 디렉터리에 `passwords.old`, `passwords.new` 파일이 있는지 확인 |
| `diff passwords.old passwords.new` | 두 파일을 비교해 서로 다른 줄을 출력 |
| `ssh -p 2220 bandit18@bandit.labs.overthewire.org` | 찾은 비밀번호로 다음 레벨 접속 |

## Level 18 → 19

이번 문제에서는 다음 레벨의 비밀번호가 홈 디렉터리의 `readme` 파일에 저장되어 있었다. 하지만 `bandit18` 계정은 `.bashrc` 파일이 수정되어 있어서, 일반적인 방식으로 SSH 로그인하면 접속 직후 바로 로그아웃되었다.

이전 단계에서 `bandit18`에 접속했을 때 환영 문구가 출력된 직후 `Byebye!` 메시지와 함께 연결이 종료되어 당황했다. 처음에는 비밀번호가 틀렸거나 접속에 실패한 줄 알았지만, 문제 설명을 다시 보니 `.bashrc`가 수정되어 로그인할 때 자동으로 로그아웃되도록 되어 있다는 내용이 있었다.

따라서 이번에는 일반적으로 접속해서 쉘에 들어가는 방식이 아니라, SSH 접속과 동시에 명령어를 실행하는 방식을 사용했다.

```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org "cat readme"
```

이 명령어는 `bandit18`에 접속한 뒤 대화형 쉘을 여는 대신, 바로 `cat readme` 명령어만 실행한다. `cat readme`는 홈 디렉터리에 있는 `readme` 파일의 내용을 출력하는 명령어이므로, 자동 로그아웃되기 전에 파일 내용을 확인할 수 있었다.

명령어 실행 결과 `readme` 파일의 내용이 출력되었고, 이를 통해 다음 레벨인 `bandit19`의 비밀번호를 확인할 수 있었다.

## Level 19 → 20

이번 문제에서는 다음 레벨의 비밀번호를 얻기 위해 홈 디렉터리에 있는 setuid 바이너리 파일을 사용해야 했다.

먼저 `ls -l` 명령어로 홈 디렉터리의 파일 목록과 권한을 확인했다. 실행 결과 `bandit20-do`라는 실행 파일이 있었고, 이 파일이 이번 문제에서 사용해야 하는 setuid 바이너리였다.

문제에서 인수 없이 실행해 보라고 했기 때문에 먼저 다음과 같이 실행했다.

```bash
./bandit20-do
```

실행 결과 이 프로그램은 특정 명령어를 다른 사용자 권한으로 실행할 수 있게 해주는 파일이라는 것을 알 수 있었다.

setuid는 실행 파일에 설정되는 특수 권한으로, 파일을 실행한 사용자가 아니라 파일 소유자의 권한으로 프로그램이 실행되도록 한다. 이번 문제에서는 `bandit20-do`를 사용하면 `bandit20` 권한으로 명령어를 실행할 수 있었다.

따라서 일반적인 비밀번호 저장 위치인 `/etc/bandit_pass`에서 `bandit20`의 비밀번호 파일을 읽었다.

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

이 명령어를 실행하자 `bandit20`의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-20.png' | relative_url }})

## Level 20 → 21

이번 문제에서는 홈 디렉터리에 있는 `suconnect`라는 setuid 바이너리를 사용해야 했다. 이전 레벨에서는 setuid 바이너리를 이용해 직접 비밀번호 파일을 읽었다면, 이번에는 포트를 열고 통신하는 방식으로 다음 레벨의 비밀번호를 받아야 했다.

문제 설명에 따르면 `suconnect`는 인수로 받은 포트 번호에 대해 `localhost`로 접속한 뒤, 그 연결에서 한 줄의 텍스트를 읽는다. 그리고 그 텍스트가 현재 레벨인 `bandit20`의 비밀번호와 일치하면, 다음 레벨인 `bandit21`의 비밀번호를 보내준다.

먼저 홈 디렉터리에서 파일을 확인했다.

```bash
ls -l
```

실행 결과 `suconnect` 파일이 있는 것을 확인했다. 이 파일을 사용하려면 먼저 `suconnect`가 접속할 수 있는 포트를 열어 두어야 했다. 이를 위해 `nc` 명령어를 사용했다.

```bash
nc -l -p 12345
```

`nc`는 네트워크 연결을 만들거나 특정 포트에서 대기할 때 사용하는 명령어이다. 여기서는 `-l` 옵션을 사용해 12345번 포트에서 연결을 기다리도록 했다.

이후 다른 터미널에서 같은 `bandit20` 계정으로 접속한 뒤, `suconnect`를 실행했다.

```bash
./suconnect 12345
```

이 명령어를 실행하면 `suconnect`가 `localhost`의 12345번 포트로 접속한다. 그다음 첫 번째 터미널에서 현재 레벨의 비밀번호를 입력하면, `suconnect`가 해당 비밀번호를 확인한다.

비밀번호가 맞으면 `suconnect`가 다음 레벨인 `bandit21`의 비밀번호를 전송한다. 따라서 첫 번째 터미널에서 응답으로 출력된 문자열을 통해 다음 레벨의 비밀번호를 확인할 수 있었다.

이번 문제를 통해 단순히 파일을 읽는 방식뿐만 아니라, `nc`를 이용해 포트를 열고 프로그램과 통신하는 방식도 사용할 수 있다는 것을 알게 되었다.

## 정리

1. 내가 localhost의 특정 포트에서 서버처럼 대기한다.
2. suconnect가 그 포트로 접속한다.
3. 내가 bandit20의 비밀번호를 보내준다.
4. suconnect가 비밀번호가 맞는지 확인한다.
5. 맞으면 bandit21의 비밀번호를 보내준다.
    
    ![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-21.png' | relative_url }})
    

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-22.png' | relative_url }})

## Level 21 → 22

이번 문제에서는 프로그램이 `cron`에 의해 정기적으로 실행된다고 했다.

이전 문제들에서는 직접 파일을 찾거나 명령어를 실행해서 비밀번호를 얻었다면, 이번 문제에서는 자동으로 실행되는 작업의 설정을 확인해야 했다.

먼저 cron 작업 설정이 저장되어 있는 `/etc/cron.d/` 디렉터리를 확인했다.

```bash
ls /etc/cron.d/
```

그중 `bandit22`와 관련된 설정 파일을 확인했다.

```bash
cat /etc/cron.d/cronjob_bandit22
```

이 파일을 통해 cron이 `/usr/bin/cronjob_bandit22.sh` 스크립트를 정기적으로 실행한다는 것을 알 수 있었다.

따라서 실제로 어떤 작업이 실행되는지 확인하기 위해 해당 스크립트의 내용을 확인했다.

```bash
cat /usr/bin/cronjob_bandit22.sh
```

스크립트 내용을 확인해 보니, `bandit22`의 비밀번호가 저장된 `/etc/bandit_pass/bandit22` 파일의 내용을 `/tmp` 아래의 특정 파일로 복사하고 있었다.

`cron`은 리눅스에서 명령어나 스크립트를 정해진 시간마다 자동으로 실행하는 작업 스케줄러이다. 이번 문제에서는 cron 설정 파일과 실행되는 스크립트를 차례대로 확인함으로써, 비밀번호가 복사되는 위치를 찾을 수 있었다.

마지막으로 스크립트에 적혀 있던 `/tmp` 경로의 파일을 `cat` 명령어로 출력하여 다음 레벨인 `bandit22`의 비밀번호를 확인할 수 있었다.

| 명령어 | 설명 |
| --- | --- |
| `ls /etc/cron.d/` | cron 작업 설정 파일 목록 확인 |
| `cat /etc/cron.d/cronjob_bandit22` | `bandit22`와 관련된 cron 설정 확인 |
| `cat /usr/bin/cronjob_bandit22.sh` | cron이 실행하는 스크립트 내용 확인 |
| `cat /tmp/특정파일명` | 스크립트가 비밀번호를 저장한 파일 내용 확인 |

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-23.png' | relative_url }})

## Level 22 → 23

이번 문제에서도 `cron`에 의해 정기적으로 실행되는 작업을 확인해야 했다. 이전 레벨과 마찬가지로 먼저 `/etc/cron.d/` 디렉터리에서 `bandit23`과 관련된 cron 설정 파일을 확인했다.

```bash
cat /etc/cron.d/cronjob_bandit23
```

설정 파일을 확인해 보니 `/usr/bin/cronjob_bandit23.sh` 스크립트가 주기적으로 실행되고 있었다. 따라서 실제로 어떤 작업이 수행되는지 확인하기 위해 해당 스크립트의 내용을 확인했다.

```bash
cat /usr/bin/cronjob_bandit23.sh
```

스크립트에서는 `whoami` 명령어로 현재 사용자의 이름을 구하고, `echo I am user $myname`의 결과를 `md5sum`으로 해시 처리한 뒤, 그 값을 `/tmp` 아래의 파일 이름으로 사용하고 있었다. 그리고 `/etc/bandit_pass/$myname`의 내용을 해당 `/tmp` 파일로 복사하고 있었다.

중요한 점은 이 스크립트가 cron에 의해 `bandit23` 사용자 권한으로 실행된다는 것이다. 따라서 스크립트 안의 `$myname`은 `bandit23`이 되고, 결국 `bandit23`의 비밀번호가 `/tmp` 아래의 특정 해시값 파일에 저장된다.

해당 파일 이름을 구하기 위해 다음 명령어를 사용했다.

```bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
```

`md5sum`은 입력값의 MD5 해시값을 계산하는 명령어이고, `cut -d ' ' -f 1`은 결과 중 해시값 부분만 추출하기 위해 사용했다.

마지막으로 구한 해시값을 이용해 `/tmp`에 생성된 파일을 읽었다.

```bash
cat /tmp/$(echo I am user bandit23 | md5sum | cut -d ' ' -f 1)
```

`$(...)`는 괄호 안의 명령어를 먼저 실행하고, 그 결과를 바깥 명령어에 넣어 사용할 때 쓰는 방식이다. 이를 통해 해시값을 직접 파일 경로에 넣어 다음 레벨인 `bandit23`의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-24.png' | relative_url }})

## Level 23 → 24

이번 문제에서도 `cron`에 의해 정기적으로 실행되는 작업을 확인해야 했다. 이전 레벨에서는 cron 스크립트가 이미 비밀번호를 특정 위치에 복사해 두었기 때문에 그 위치를 찾아 읽으면 됐지만, 이번 문제에서는 직접 쉘 스크립트를 작성해야 했다.

먼저 `/etc/cron.d/cronjob_bandit24` 파일을 확인해 cron이 어떤 스크립트를 실행하는지 확인했다.

```bash
cat /etc/cron.d/cronjob_bandit24
```

그 결과 `/usr/bin/cronjob_bandit24.sh`가 정기적으로 실행된다는 것을 알 수 있었다. 이후 해당 스크립트의 내용을 확인했다.

```bash
cat /usr/bin/cronjob_bandit24.sh
```

스크립트를 확인해 보니 `/var/spool/bandit24/foo` 디렉터리 안에 있는 스크립트들을 실행한 뒤 삭제하는 구조였다. 문제 참고 사항에서 실행된 쉘 스크립트가 제거된다고 한 이유가 바로 이 부분이었다. 따라서 원본 스크립트는 따로 보관하고, 복사본을 해당 디렉터리에 넣어야 했다.

먼저 `/tmp` 아래에 작업용 디렉터리를 만들고 권한을 설정했다.

```
mkdir /tmp/bandit24_bora
chmod 777 /tmp/bandit24_bora
cd /tmp/bandit24_bora
```

이번에는 직접 쉘 스크립트를 만들어야 했기 때문에 `getpass.sh` 파일을 작성했다.

```bash
nano getpass.sh
```

스크립트 내용은 다음과 같이 작성했다.

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/bandit24_bora/password
chmod 666 /tmp/bandit24_bora/password
```

`#!/bin/bash`는 이 파일을 bash 쉘로 실행하겠다는 의미이다. 스크립트 안에서는 `bandit24`의 비밀번호 파일을 읽어 `/tmp/bandit24_bora/password` 파일로 저장하도록 했다. 또한 나중에 `bandit23` 사용자가 해당 파일을 읽을 수 있도록 `chmod 666`으로 권한을 변경했다.

작성한 스크립트에 실행 권한을 부여한 뒤, cron이 확인하는 디렉터리로 복사했다.

```bash
chmod +x getpass.sh
cp getpass.sh /var/spool/bandit24/foo/
```

cron은 일정 시간마다 자동으로 실행되기 때문에 잠시 기다린 뒤, `/tmp`에 저장된 결과 파일을 확인했다.

```bash
cat /tmp/bandit24_bora/password
```

그 결과 다음 레벨인 `bandit24`의 비밀번호를 확인할 수 있었다. 이번 문제를 통해 cron이 실행하는 스크립트 구조를 분석하는 방법과, 직접 쉘 스크립트를 작성해 원하는 명령을 실행시키는 방법을 배웠다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-25.png' | relative_url }})

## Level 24 → 25

이번 문제에서는 `localhost`의 30002번 포트에 `bandit24`의 비밀번호와 4자리 PIN 코드를 함께 제출해야 했다.

이전 레벨들에서는 파일을 찾거나 스크립트를 작성하는 방식이었다면, 이번에는 가능한 모든 PIN 조합을 시도하는 brute-force 방식이 필요했다.

문제에서 PIN 코드는 4자리 숫자라고 했으므로 가능한 범위는 `0000`부터 `9999`까지이다. 따라서 총 10000개의 조합을 만들어야 했다. 먼저 현재 레벨의 비밀번호를 변수에 저장했다.

```bash
password=$(cat /etc/bandit_pass/bandit24)
```

이후 `seq -w 0000 9999`를 사용해 4자리 PIN 후보를 생성했다. `seq`는 연속된 숫자를 출력하는 명령어이고, `-w` 옵션은 숫자의 자릿수를 맞춰 `0000`처럼 앞에 0이 붙은 형태로 출력해 준다.

```bash
for pin in $(seq -w 0000 9999); do echo "$password$pin"; done | nc localhost 30002
```

`for` 반복문을 사용해 각 PIN마다 `bandit24`의 비밀번호와 PIN을 한 줄로 만들어 전송했다. 문제에서 매번 새로운 연결을 만들 필요가 없다고 했기 때문에, 모든 시도 값을 한 번에 만들어 `nc localhost 30002`로 전달했다.

출력 결과에는 틀린 PIN에 대한 `Wrong` 메시지가 많이 포함되므로, 보기 쉽게 하기 위해 `grep -v "Wrong"`을 사용할 수도 있다.

```bash
for pin in $(seq -w 0000 9999); do echo "$password$pin"; done | nc localhost 30002 | grep -v "Wrong"
```

`grep -v`는 특정 문자열이 포함되지 않은 줄만 출력하는 옵션이다. 이를 통해 실패 메시지를 제외하고, 정답 PIN을 찾았을 때 출력되는 `Correct!` 메시지와 다음 레벨의 비밀번호를 확인할 수 있었다.

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-26.png' | relative_url }})

![스크린샷]({{ '/assets/images/linux-system-exploration-bandit/img-27.png' | relative_url }})
