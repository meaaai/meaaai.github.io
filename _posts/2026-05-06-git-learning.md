---
title: "[한 달-필수미션] Git 배우기"
date: 2026-05-06 00:00:00 +0900
categories: [Git]
tags: [git, github, branch, merge, rebase, cherry-pick, reset, revert]
pin: false
math: false
mermaid: false
---

## 들어가며

Learn Git Branching을 실습하면서 Git의 기본 명령어, 브랜치 이동, 작업 되돌리기, 체리픽, 인터랙티브 리베이스, 태그를 정리했다.

Git은 단순히 파일을 저장하는 도구가 아니라, 프로젝트의 변경 이력을 커밋 단위로 기록하고 필요한 시점으로 이동하거나 여러 작업 흐름을 합칠 수 있게 해주는 버전 관리 도구다.

## Git 기본 명령어

### 1. `git commit`

커밋은 현재 프로젝트 상태를 하나의 기록으로 남기는 것이다.

게임의 저장 지점처럼 커밋을 해두면 나중에 변경 내용을 확인하거나 이전 상태로 돌아갈 수 있다. Learn Git Branching에서는 `git commit` 명령어를 입력할 때마다 새로운 커밋 노드가 생성된다.

### 2. Git 브랜치

브랜치는 특정 커밋을 가리키는 이름표라고 볼 수 있다.

브랜치가 하나의 커밋을 가리키면, 그 커밋과 부모 커밋들을 따라가며 하나의 작업 내역이 만들어진다. 브랜치는 커밋을 추가하거나 위치를 옮기면 함께 움직일 수 있는 포인터다.

### 3. `git checkout [브랜치명]`

`git checkout [브랜치명]`은 현재 작업 위치를 해당 브랜치로 이동하는 명령어다.

실제 Git에서 새 브랜치를 만들면서 바로 이동하려면 보통 다음 명령어를 사용한다.

```bash
git checkout -b newImage
```

또는 최신 Git에서는 다음처럼 쓸 수 있다.

```bash
git switch -c newImage
```

Learn Git Branching에서는 브랜치와 커밋의 이동 관계를 시각적으로 확인할 수 있다.

![새 브랜치로 checkout하기](/assets/img/posts/git-learning/checkout-branch.png)

### 4. `git merge`

`git merge`는 브랜치를 합치는 명령어다.

주의할 점은 **현재 내가 서 있는 브랜치에 다른 브랜치를 합친다**는 것이다.

예를 들어 현재 `main` 브랜치에 있고 `bugFix` 브랜치를 합치고 싶다면 다음과 같이 입력한다.

```bash
git checkout main
git merge bugFix
```

### 5. `git rebase`

`merge`가 브랜치를 합치는 명령어라면, `rebase`는 브랜치의 시작 위치를 다른 커밋 뒤로 옮기는 명령어다.

결과적으로 다른 브랜치의 내용을 가져온다는 점은 비슷하지만, 커밋 기록이 남는 모양이 다르다. `rebase`를 사용하면 merge commit 없이 히스토리를 한 줄에 가깝게 정리할 수 있다.

현재 브랜치를 `main` 브랜치 위로 재배치하려면 다음처럼 쓴다.

```bash
git rebase main
```

또는 특정 브랜치를 기준 브랜치 위로 재배치하려면 다음처럼 쓸 수 있다.

```bash
git rebase [기준브랜치] [옮길브랜치]
```

### 6. Git의 상대 참조

Git에서는 커밋 해시를 직접 입력하지 않아도 상대 참조를 이용해 커밋 위치를 표현할 수 있다.

![Git 상대 참조](/assets/img/posts/git-learning/relative-ref.png)

- `^`: 부모 커밋으로 한 칸 이동
- `^^`: 부모의 부모 커밋으로 이동
- `~숫자`: 숫자만큼 부모 커밋을 따라 이동

예시는 다음과 같다.

```bash
git checkout main^
git checkout main~3
```

`main^`은 `main`이 가리키는 커밋의 부모 커밋을 의미하고, `main~3`은 `main`에서 세 칸 위의 조상 커밋을 의미한다.

### 7. `git branch -f [브랜치이름] [이동할커밋]`

`git branch -f`는 브랜치가 가리키는 위치를 강제로 옮기는 명령어다.

![git branch -f 예시](/assets/img/posts/git-learning/branch-force.png)

예를 들어 다음 명령어는 `main` 브랜치를 `C6` 커밋으로 이동시킨다.

```bash
git branch -f main C6
```

이 명령어는 `C6` 커밋을 새로 만드는 것이 아니다. 이미 존재하는 `C6` 커밋을 `main` 브랜치가 가리키도록 바꾸는 것이다.

따라서 원래 `main`이 가리키던 커밋은 그대로 남아 있지만, 더 이상 `main`이 그 커밋을 가리키지 않게 된다. Learn Git Branching에서는 현재 브랜치나 `HEAD`가 직접 가리키지 않는 커밋이 흐리게 표시된다.

### 8. Git에서 작업 되돌리기: `reset`과 `revert`

Git에서 작업을 되돌리는 대표적인 방법은 `git reset`과 `git revert`다.

#### `git reset`

`git reset`은 현재 브랜치가 가리키는 커밋 위치를 이전 커밋으로 되돌리는 명령어다.

예를 들어 현재 커밋 기록이 다음과 같다고 하자.

```text
C0 - C1 - C2 - C3 ← main
```

여기서 다음 명령어를 실행하면,

```bash
git reset HEAD~1
```

`main` 브랜치가 한 칸 이전 커밋인 `C2`로 이동한다.

```text
C0 - C1 - C2 ← main
```

즉, `git reset`은 새 커밋을 만드는 것이 아니라 브랜치의 위치를 과거 커밋으로 되돌리는 명령어다.

#### `git revert`

`git revert`는 되돌리는 내용을 새로운 커밋으로 기록하는 명령어다.

예를 들어 `git revert HEAD`를 실행하면, 현재 `HEAD` 커밋의 변경 사항을 반대로 적용한 새 커밋이 만들어진다. 이 방식은 히스토리를 지우지 않기 때문에 다른 사람과 공유하는 브랜치에서도 안전하게 사용할 수 있다.

로컬에서 혼자 작업하는 브랜치라면 `reset`을 사용할 수 있지만, 이미 원격 저장소에 올렸거나 다른 사람과 함께 쓰는 브랜치라면 `revert`를 사용하는 편이 안전하다.

![Git 기본 단계 완료](/assets/img/posts/git-learning/git-basic-complete.png)

## 코드 이리저리 옮기기

### 9. `git cherry-pick`

`git cherry-pick`은 특정 커밋을 골라 현재 브랜치로 가져오는 명령어다.

![git cherry-pick 예시](/assets/img/posts/git-learning/cherry-pick.png)

예를 들어 현재 `main` 브랜치에 있고 `C3`, `C4`, `C7` 커밋만 가져오고 싶다면 다음처럼 입력한다.

```bash
git cherry-pick C3 C4 C7
```

현재 서 있는 브랜치에 커밋들이 복사되어 새 커밋으로 붙는다. 원하는 커밋을 하나씩 고를 수 있어서 이해하기 쉽지만, 가져와야 할 커밋이 많아지면 명령어가 길어질 수 있다.

### 10. `git rebase -i`

`git rebase -i`는 인터랙티브 리베이스를 실행하는 명령어다.

여기서 `-i`는 `interactive`의 줄임말로, 사용자가 직접 커밋 목록을 수정할 수 있다는 뜻이다.

일반적인 `git rebase`가 브랜치의 시작 위치를 옮기는 명령어라면, `git rebase -i`는 커밋 기록을 직접 편집하는 명령어에 가깝다.

![인터랙티브 리베이스 설명](/assets/img/posts/git-learning/interactive-rebase.png)

인터랙티브 리베이스에서는 다음 작업을 할 수 있다.

- 커밋 순서 바꾸기
- 필요 없는 커밋 빼기
- 커밋 메시지 수정하기
- 여러 커밋 합치기

예를 들어 최근 3개의 커밋을 대상으로 인터랙티브 리베이스를 실행하려면 다음처럼 입력한다.

```bash
git rebase -i HEAD~3
```

### 11. `git commit --amend`

`git commit --amend`는 새 커밋을 하나 더 만들지 않고, 방금 만든 마지막 커밋을 고쳐서 다시 만드는 명령어다.

`--amend`는 항상 `HEAD`가 가리키는 가장 마지막 커밋만 수정할 수 있다. 그래서 중간 커밋을 수정하려면 `git rebase -i`로 해당 커밋을 마지막 위치로 옮긴 뒤 `git commit --amend`를 사용하고, 다시 `git rebase -i`로 순서를 정리할 수 있다.

![git rebase와 git commit --amend 예시](/assets/img/posts/git-learning/amend-rebase.png)

### 12. `git tag [태그이름] [커밋이름]`

Git에서 태그는 특정 커밋에 붙이는 고정된 이름표다.

브랜치는 새로운 커밋이 생기면 움직일 수 있는 이름표이고, 태그는 보통 특정 버전이나 릴리즈 지점을 표시하기 위해 고정해 두는 이름표다.

![git tag 예시](/assets/img/posts/git-learning/tag.png)

예시는 다음과 같다.

```bash
git tag v1 C1
```

이 명령어는 `C1` 커밋에 `v1`이라는 태그를 붙인다.

### 13. `git describe [커밋이름]`

`git describe`는 현재 커밋이 가장 가까운 태그로부터 얼마나 떨어져 있는지 알려주는 명령어다.

![git describe 설명](/assets/img/posts/git-learning/describe.png)

형태는 다음과 같다.

```bash
git describe <ref>
```

`<ref>`에는 커밋 해시, 브랜치 이름, 태그 이름, `HEAD` 등을 넣을 수 있다. 생략하면 현재 `HEAD`를 기준으로 실행된다.

출력은 보통 다음 형태로 나타난다.

```text
<tag>_<numCommits>_g<hash>
```

- `<tag>`: 가장 가까운 부모 태그
- `<numCommits>`: 해당 태그로부터 몇 커밋 떨어져 있는지
- `<hash>`: 현재 커밋의 해시 일부

## 문제풀이 정리

### 문제풀이 1. 여러 번 리베이스 하기

브랜치를 `rebase`하면 그 브랜치가 가리키는 마지막 커밋 하나만 옮겨지는 것이 아니다.

공통 조상 이후 그 브랜치에 속한 커밋들이 함께 옮겨진다. 그래서 브랜치 구조를 잘 보면 명령어 횟수를 줄일 수 있다.

### 문제풀이 2. 브랜치 스파게티

![브랜치 스파게티 문제](/assets/img/posts/git-learning/branch-spaghetti.png)

이 문제는 `cherry-pick`과 `rebase -i` 두 가지 방식으로 해결할 수 있다.

#### `cherry-pick` 방식

`cherry-pick`은 원하는 커밋을 하나씩 골라 현재 브랜치에 가져오는 방식이다. 이해하기 쉽지만 가져와야 할 커밋이 많으면 명령어가 길어진다.

```bash
git checkout one
git cherry-pick C4 C3 C2

git checkout two
git cherry-pick C5 C4 C3 C2

git branch -f three C2
```

#### `rebase -i` 방식

`rebase -i`는 여러 커밋의 순서를 한 번에 바꿀 수 있다. 커밋들을 원하는 순서로 재배치한 뒤, `git branch -f`로 브랜치가 가리키는 위치만 조정하면 된다.

```bash
git rebase -i C1 C4
git branch -f one HEAD

git rebase -i C1 C5
git branch -f two HEAD

git branch -f three C2
```

단, `rebase -i`를 여러 번 사용하거나 브랜치 위치를 반복해서 수정하면 오히려 `cherry-pick`보다 복잡해질 수 있다. 따라서 문제 구조가 단순하고 가져올 커밋이 명확하면 `cherry-pick`이 편하고, 커밋 순서를 한 번에 정리해야 한다면 `rebase -i`가 더 적합하다.

## 마무리

Git을 처음 배울 때는 명령어 자체보다 **브랜치가 커밋을 가리키는 포인터**라는 감각을 익히는 것이 중요하다.

`commit`은 기록을 만들고, `branch`는 기록의 위치를 가리키고, `checkout`은 내가 작업할 위치를 바꾸고, `merge`와 `rebase`는 서로 다른 작업 흐름을 합치거나 재배치한다.

이번 실습을 통해 Git 명령어가 단순한 암기 대상이 아니라 커밋 그래프를 움직이는 도구라는 점을 이해할 수 있었다.

---

참고 및 실습 출처: [Learn Git Branching](https://learngitbranching.js.org/)
