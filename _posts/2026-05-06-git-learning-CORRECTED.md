---
title: "Git 배우기"
date: 2026-05-06
categories: [Git]
tags: [git, github, learn-git-branching, branch, rebase, remote]
---

Learn Git Branching으로 Git의 기본 명령어부터 원격 저장소 명령어까지 학습한 내용을 정리했다.

## git 기본 명령어

### 1. git commit

커밋(commit)은 Git에서 현재 프로젝트 상태를 하나의 기록으로 남기는 것이다.

게임의 저장 지점처럼 커밋을 해두면 나중에 변경 내용을 확인하거나 이전 상태로 돌아갈 수 있다. Learn Git Branching에서는 `git commit` 명령어를 입력할 때마다 새로운 커밋 노드가 생성된다.

### 2. git branch

브랜치는 하나의 커밋과 그 부모 커밋들을 포함하는 작업 내역이다. 쉽게 말하면 커밋에 붙어 있는 움직이는 이름표라고 볼 수 있다.

```text
브랜치 = 특정 커밋을 가리키는 이름표
```

### 3. git checkout [브랜치명]

`git checkout`은 현재 위치를 다른 브랜치나 커밋으로 이동시키는 명령어이다.

```bash
git checkout [브랜치명]
```

변경분을 커밋하기 전에 새 브랜치로 이동하면, 이후 커밋은 이동한 브랜치에 기록된다.

![새 브랜치로 이동하는 예시](/assets/img/git-learning/checkout-branch.png)

### 4. git merge

`git merge`는 브랜치를 합치는 명령어이다.

중요한 점은 **현재 내가 서 있는 브랜치에 다른 브랜치를 합친다**는 것이다.

```bash
git checkout main
git merge feature
```

위 명령어는 `feature` 브랜치의 내용을 `main` 브랜치에 합친다.

### 5. git rebase [기준브랜치] [옮길 브랜치]

`merge`가 브랜치를 합치는 명령어라면, `rebase`는 브랜치의 시작 위치를 옮기는 명령어이다.

```bash
git rebase main feature
```

`rebase`는 현재 브랜치의 커밋들을 다른 브랜치의 최신 커밋 뒤로 다시 배치한다. `merge commit` 없이 커밋 흐름을 한 줄로 정리할 수 있다는 장점이 있다.

```text
merge  = 브랜치를 합친다
rebase = 브랜치의 시작 위치를 옮긴다
```

### 6. git의 상대 참조

Git에서는 커밋 해시를 직접 외우지 않아도 상대 참조를 이용해 커밋을 이동할 수 있다.

![상대 참조 예시](/assets/img/git-learning/relative-ref.png)

```text
^  = 부모 커밋으로 한 칸 이동
^^ = 부모의 부모 커밋으로 이동
~n = n개 부모 커밋 위로 이동
```

예를 들어 다음 명령어는 `main`의 부모 커밋으로 이동한다.

```bash
git checkout main^
```

다음 명령어는 `main`에서 두 커밋 위로 이동한다.

```bash
git checkout main~2
```

### 7. git branch -f [브랜치이름] [이동할커밋]

`git branch -f`는 브랜치가 가리키는 위치를 강제로 옮기는 명령어이다.

```bash
git branch -f main C6
```

![git branch -f 예시](/assets/img/git-learning/branch-force.png)

이 명령어는 `C6` 커밋을 새로 만드는 것이 아니라, 이미 존재하는 `C6` 커밋으로 `main` 브랜치를 강제로 이동시키는 명령어이다.

따라서 원래 `main`이 가리키던 커밋은 그대로 남아 있지만, 더 이상 `main`이 그 커밋을 가리키지 않게 된다.

Learn Git Branching에서는 각 레벨의 연습 목적에 맞게 여러 커밋이 미리 만들어져 있다. 그래서 어떤 단계에서는 새 커밋을 만드는 것이 아니라 `HEAD`, `main`, `bugFix` 같은 포인터를 원하는 커밋으로 이동시키는 연습을 한다.

### 8. git에서 작업 되돌리기: reset과 revert

Git에서 작업을 되돌리는 대표적인 방법은 `git reset`과 `git revert`이다.

#### git reset

`git reset`은 현재 브랜치가 가리키는 커밋 위치를 이전 커밋으로 되돌리는 명령어이다.

예를 들어 현재 커밋 기록이 다음과 같다고 하자.

```text
C0 - C1 - C2 - C3 ← main
```

여기서 다음 명령어를 실행하면:

```bash
git reset HEAD~1
```

`main` 브랜치가 한 칸 이전 커밋인 `C2`로 이동한다.

```text
C0 - C1 - C2 ← main
```

즉, `git reset`은 새 커밋을 만드는 것이 아니라 브랜치의 위치를 과거 커밋으로 되돌리는 명령어이다.

#### git revert

`git revert`는 기존 커밋을 직접 지우는 대신, 그 커밋의 변경 내용을 반대로 적용하는 새 커밋을 만든다.

```bash
git revert HEAD
```

`reset`은 히스토리를 고쳐 쓰기 때문에 혼자 작업하는 로컬 브랜치에서는 사용할 수 있지만, 다른 사람과 공유하는 리모트 브랜치에서는 조심해야 한다. 공유된 변경 내역을 되돌릴 때는 보통 `revert`를 사용한다.

![git 기본 단계 완료](/assets/img/git-learning/git-basic-complete.png)

## 코드 이리저리 옮기기

### 9. git cherry-pick

`git cherry-pick`은 특정 커밋을 골라 현재 브랜치로 가져오는 명령어이다.

```bash
git cherry-pick C3 C4 C7
```

![git cherry-pick 예시](/assets/img/git-learning/cherry-pick.png)

현재 서 있는 브랜치가 `main`이라면, 위 명령어는 `C3`, `C4`, `C7` 커밋을 골라 `main` 브랜치 위에 복사한다.

### 10. git rebase -i

`git rebase -i`는 인터랙티브 리베이스를 실행하는 명령어이다.

```bash
git rebase -i HEAD~3
```

여기서 `-i`는 `interactive`의 줄임말로, 사용자가 직접 커밋 목록을 수정할 수 있다는 뜻이다.

일반적인 `git rebase`가 브랜치의 시작 위치를 옮기는 명령어라면, `git rebase -i`는 커밋 기록을 직접 편집하는 명령어이다.

할 수 있는 작업은 다음과 같다.

```text
커밋 순서 변경
커밋 선택
커밋 삭제
커밋 수정
```

![인터랙티브 리베이스 예시](/assets/img/git-learning/interactive-rebase.png)

### 11. git commit --amend

`git commit --amend`는 새 커밋을 하나 더 만들지 않고, 방금 만든 마지막 커밋을 고쳐서 다시 만드는 명령어이다.

```bash
git commit --amend
```

`--amend`는 항상 `HEAD`가 가리키는 가장 마지막 커밋만 수정할 수 있다.

그래서 중간 커밋을 수정하려면 `git rebase -i`로 수정하고 싶은 커밋을 맨 끝으로 옮긴 뒤 `git commit --amend`를 하고, 다시 `git rebase -i`로 순서를 정리할 수 있다.

![rebase와 amend 예시](/assets/img/git-learning/amend-rebase.png)

### 12. git tag

`git tag`는 특정 커밋에 이름표를 붙이는 명령어이다.

```bash
git tag v1 C1
```

브랜치는 계속 움직이는 이름표이고, 태그는 보통 고정된 이름표이다.

![git tag 예시](/assets/img/git-learning/tag.png)

### 13. git describe

`git describe`는 현재 커밋이 가장 가까운 태그로부터 몇 번째 커밋인지 알려주는 명령어이다.

```bash
git describe [커밋이름]
```

출력 형태는 보통 다음과 같다.

```text
<tag>_<numCommits>_g<hash>
```

![git describe 예시](/assets/img/git-learning/describe.png)

## 문제풀이

### 여러 번 리베이스하기

브랜치를 `rebase`하면 그 브랜치가 가리키는 마지막 커밋 하나만 옮기는 것이 아니라, 공통 조상 이후 그 브랜치에 속한 커밋들이 함께 옮겨진다.

그래서 커밋을 하나씩 옮기지 않아도 브랜치 단위로 작업 흐름을 정리할 수 있다.

### 브랜치 스파게티

![브랜치 스파게티 예시](/assets/img/git-learning/branch-spaghetti.png)

이 문제는 `cherry-pick`과 `rebase -i` 두 가지 방식으로 해결할 수 있다.

`cherry-pick`은 원하는 커밋을 하나씩 골라 현재 브랜치에 가져오는 방식이라 이해하기 쉽다. 하지만 가져와야 할 커밋이 많으면 명령어가 길어질 수 있다.

반면 `rebase -i`는 여러 커밋의 순서를 한 번에 바꿀 수 있다. 커밋들을 원하는 순서로 재배치한 뒤 `git branch -f`로 브랜치가 가리키는 위치만 조정하면 된다.

예시 답안은 다음과 같다.

```bash
git rebase -i C1 C4
git branch -f one HEAD
git rebase -i C1 C5
git branch -f two HEAD
git branch -f three C2
```

또는 `cherry-pick`으로도 해결할 수 있다.

```bash
git checkout one
git cherry-pick C4 C3 C2
git checkout two
git cherry-pick C5 C4 C3 C2
git branch -f three C2
```

---

## 원격 저장소

### 14. git clone

`git clone`은 원격 저장소에 있는 프로젝트를 내 컴퓨터로 복사해 오는 명령어이다.

```bash
git clone [원격 저장소 주소]
```

GitHub에 있는 저장소를 처음 내 컴퓨터에서 작업하려면 먼저 `git clone`을 사용한다.

```bash
git clone https://github.com/meaaai/meaaai.github.io.git
```

`git clone`을 하면 원격 저장소의 파일과 커밋 기록이 내 컴퓨터로 복사된다. 또한 Git은 기본적으로 원격 저장소에 `origin`이라는 이름을 붙인다.

```text
origin = 내가 clone해 온 원격 저장소의 기본 이름
```

그리고 원격 저장소의 브랜치는 내 컴퓨터에서 `origin/main` 같은 형태로 보인다. Learn Git Branching에서는 화면에 맞추기 위해 `origin/main`을 `o/main`처럼 줄여서 표시한다.

```text
main        = 내 로컬 브랜치
origin/main = 원격 저장소의 main을 따라가는 원격 추적 브랜치
```

### 15. 원격 브랜치

원격 브랜치는 `origin/main`, `origin/foo`처럼 원격 저장소의 브랜치 상태를 내 컴퓨터가 기억해 둔 브랜치이다.

Learn Git Branching에서는 이를 `o/main`, `o/foo`처럼 표시한다.

![원격 브랜치 참고](/assets/img/git-learning/remote-branch-reference.png)

원격 브랜치를 직접 체크아웃하면 보통 분리된 `HEAD` 상태가 된다. Git은 원격 추적 브랜치에서 직접 작업하도록 두지 않기 때문이다.

```text
o/main = 원격 저장소 main의 마지막 확인 상태
main   = 내가 직접 작업하는 로컬 브랜치
```

새로운 커밋을 만들어도 `o/main`이 바로 갱신되지는 않는다. `o/main`은 원격 저장소와 통신하는 `fetch`, `pull`, `push` 같은 작업을 할 때 갱신된다.

### 16. git fetch

`git fetch`는 원격 저장소의 최신 정보를 가져오는 명령어이다.

```bash
git fetch
```

![git fetch와 git clone의 차이 1](/assets/img/git-learning/fetch-vs-clone-1.png)

![git fetch와 git clone의 차이 2](/assets/img/git-learning/fetch-vs-clone-2.png)

`git fetch`가 하는 일은 크게 두 가지이다.

```text
1. 원격 저장소에는 있지만 로컬에는 없는 커밋을 다운로드한다.
2. origin/main 같은 원격 추적 브랜치를 최신 상태로 업데이트한다.
```

중요한 점은 `git fetch`를 해도 내가 작업 중인 파일이나 로컬 브랜치는 바로 바뀌지 않는다는 것이다.

```text
git fetch 이후 바뀌는 것   = origin/main 같은 원격 추적 브랜치
git fetch 이후 안 바뀌는 것 = 내 로컬 main, 작업 파일
```

### 17. git pull

`git pull`은 원격 저장소의 변경 내용을 가져와 현재 브랜치에 합치는 명령어이다.

```bash
git pull
```

본질적으로는 다음 두 작업을 한 번에 실행하는 단축 명령어이다.

```text
git pull = git fetch + git merge
```

따라서 `git pull`만 사용하면 원격 커밋과 내 로컬 커밋을 합치기 위해 merge commit이 생길 수 있다.

![git pull을 사용했을 때의 merge 예시](/assets/img/git-learning/pull-merge-explanation.png)

### 18. git pull --rebase

`git pull --rebase`는 원격 저장소의 최신 커밋을 가져온 뒤, 내가 로컬에서 만든 커밋을 그 뒤로 다시 배치하는 명령어이다.

```bash
git pull --rebase
```

일반 `git pull`과 비교하면 다음과 같다.

```text
git pull          = git fetch + git merge
git pull --rebase = git fetch + git rebase
```

원격 저장소가 먼저 업데이트되어 `git push`가 거절될 때, 다음 흐름으로 내 작업을 정리할 수 있다.

```bash
git pull --rebase
git push
```

이렇게 하면 원격 저장소의 최신 커밋 위에 내 로컬 커밋을 다시 올린 뒤 push할 수 있다.

### 19. git fakeTeamwork

`git fakeTeamwork`는 실제 Git 명령어가 아니라 Learn Git Branching에서 사용하는 학습용 명령어이다.

다른 사람이 원격 저장소에 커밋을 올린 상황을 가짜로 만들어준다.

```bash
git fakeTeamwork
```

커밋 개수나 브랜치를 지정할 수도 있다.

```bash
git fakeTeamwork 3
git fakeTeamwork side 5
```

실제 터미널에서는 사용할 수 없다.

### 20. git push

`git push`는 내 로컬 저장소의 커밋을 원격 저장소에 반영하는 명령어이다.

```bash
git push
```

`git pull`이 원격 저장소의 변경 내용을 내 로컬로 가져오는 명령어라면, `git push`는 반대로 내 로컬에서 만든 변경 내용을 원격 저장소로 올리는 명령어이다.

```text
git pull = 원격 저장소 → 로컬 저장소
git push = 로컬 저장소 → 원격 저장소
```

push를 하지 못하고 원격 저장소가 먼저 업데이트된 경우에는 다음처럼 작업을 원격 브랜치의 최신 상태를 기반으로 다시 정리한 뒤 push한다.

```bash
git fetch
git rebase origin/main
git push
```

또는 짧게 다음처럼 쓸 수 있다.

```bash
git pull --rebase
git push
```

### 21. 실수로 main 브랜치에서 커밋했을 때

원래는 새 브랜치를 만들어 작업한 뒤, 그 브랜치를 원격 저장소에 push하고 Pull Request를 만들어야 한다.

하지만 실수로 `main` 브랜치에서 바로 커밋을 해버리는 경우가 있다.

이때 바로 `main`을 push하려고 하면 문제가 생길 수 있다. `main` 브랜치는 보통 직접 push하는 브랜치가 아니라, Pull Request를 통해 변경 사항을 반영하는 중심 브랜치이기 때문이다.

이 상황에서는 실수로 만든 커밋을 새 브랜치에 보관하고, `main` 브랜치는 다시 원격 저장소의 `main` 상태로 되돌린다.

```bash
git branch feature
git reset origin/main
git checkout feature
git push
```

결과적으로 `main`은 원격 저장소와 같은 깨끗한 상태가 되고, 내가 만든 작업은 `feature` 브랜치에 남게 된다.

```text
main    = 원격 main과 같은 상태
feature = 내가 작업한 커밋을 가진 브랜치
```

Pull Request는 내가 작업한 브랜치의 변경 내용을 `main` 브랜치에 합치기 전에 검토하고 승인받기 위해 만드는 요청이다.

### 22. git push 인자 사용하기

일반적으로 `git push`는 현재 브랜치의 작업을 연결된 원격 브랜치로 올린다.

하지만 다음과 같이 작성하면, 로컬의 특정 브랜치나 커밋을 원격의 다른 브랜치로 push할 수 있다.

```bash
git push origin <source>:<destination>
```

여기서 `:` 앞은 로컬에서 보낼 대상이고, `:` 뒤는 원격 저장소에서 업데이트할 브랜치이다.

```text
source      = 로컬에서 보낼 커밋 또는 브랜치
destination = 원격 저장소에서 갱신할 브랜치
```

예를 들어 다음 명령어는 로컬 `foo` 브랜치를 원격 `main` 브랜치로 push한다.

```bash
git push origin foo:main
```

다음 명령어는 로컬 `main`의 부모 커밋을 원격 `foo` 브랜치로 push한다.

```bash
git push origin main^:foo
```

### 23. git fetch 인자 사용하기

`git fetch`도 source와 destination을 직접 지정할 수 있다.

```bash
git fetch origin <source>:<destination>
```

`push`와 방향이 반대라고 생각하면 된다.

```text
git push origin A:B  = 로컬 A를 원격 B로 보냄
git fetch origin A:B = 원격 A를 로컬 B로 가져옴
```

예를 들어 다음 명령어는 원격의 `C6` 커밋을 가져와 로컬 `main` 브랜치가 가리키게 만든다.

```bash
git fetch origin C6:main
```

### 24. Source가 없는 fetch와 push

`source`를 비워서 사용하는 경우도 있다.

#### push에서 source가 없는 경우

```bash
git push origin :foo
```

`push`에서 `:` 앞이 비어 있으면 원격 브랜치를 삭제한다.

```text
아무것도 없는 값을 원격 foo에 push한다
= 원격 foo를 삭제한다
```

#### fetch에서 source가 없는 경우

```bash
git fetch origin :bar
```

`fetch`에서 `:` 앞이 비어 있으면 로컬에 새 브랜치를 만든다.

```text
원격 저장소의 기본 위치를 가져와 로컬 bar 브랜치를 만든다
```

즉, `git branch` 명령어가 비활성화된 상황에서도 `fetch`를 이용해 새 로컬 브랜치를 만들 수 있다.

### 25. git pull 인자 사용하기

`git pull`도 인자를 사용할 수 있다.

```bash
git pull origin <source>:<destination>
```

이 명령어는 원격의 `source`를 가져와 로컬의 `destination` 브랜치를 만들거나 갱신한 뒤, 그 브랜치를 현재 브랜치에 merge한다.

예를 들어 다음 명령어는 원격의 `C3` 커밋을 로컬 `foo` 브랜치로 가져오고, 현재 브랜치에 `foo`를 merge한다.

```bash
git pull origin C3:foo
```

다음 명령어는 원격의 `C2` 커밋을 로컬 `side` 브랜치로 가져오고, 현재 브랜치에 `side`를 merge한다.

```bash
git pull origin C2:side
```

이렇게 하면 커밋을 내려받고, 새 브랜치를 만들고, 그 브랜치를 현재 브랜치로 병합하는 과정을 비교적 짧은 명령어로 처리할 수 있다.

---

## 인증

![main 단계 완료](/assets/img/git-learning/main-complete.png)

![원격 단계 완료](/assets/img/git-learning/remote-complete.png)

---

참고 및 실습 출처: Learn Git Branching
