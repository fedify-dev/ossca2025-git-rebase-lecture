OSSCA 2025 Git 리베이스 실습
============================

이 저장소는 OSSCA 2025 멘티들을 위한 Git 리베이스 실습 교보재입니다.


리베이스란?
-----------

Git에서 **리베이스**(rebase)란 이름 그대로 커밋들의 기반을 바꾸는 것을 뜻합니다.
예를 들어 다음과 같은 두 개의 브랜치가 있을 때:

~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: a
---
gitGraph BT:
  commit id: "공통 조상 1"
  commit id: "공통 조상 2"
  commit id: "공통 조상 3"
  branch b
  commit id: "B 커밋 1"
  commit id: "B 커밋 2"
  checkout a
  commit id: "A 커밋 1"
  commit id: "A 커밋 2"
  commit id: "A 커밋 3"
~~~~

다음과 같이 선형화하는 것을 뜻합니다:

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: a
---
gitGraph BT:
  commit id: "공통 조상 1"
  commit id: "공통 조상 2"
  commit id: "공통 조상 3"
  commit id: "B 커밋 1"
  commit id: "B 커밋 2"
  commit id: "A 커밋 1"
  commit id: "A 커밋 2"
  commit id: "A 커밋 3"
~~~~


충돌 없는 경우
--------------

가장 먼저 실습할 리베이스는 충돌이 없는 경우입니다. 우선, 다음과 같이 공통된
과거 커밋을 가진 두 개의 브랜치가 있습니다 (위로 갈 수록 최신):

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: no-conflict-a
---
gitGraph BT:
    commit id: "첫 커밋"
    branch no-conflict-b
    checkout no-conflict-b
    commit id: "Flex"
    checkout no-conflict-a
    commit id: "제목에 에모지 추가"
~~~~

일단 우리는 `no-conflict-a` 브랜치에 있다고 가정합시다:

~~~~ sh
git switch no-conflict-a
~~~~

만일 우리가 리베이스를 하지 않고 머지를 한다면(`git merge no-conflict-b`),
다음과 같은 커밋 그래프가 생성 될 겁니다:

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: no-conflict-a
---
gitGraph BT:
    commit id: "첫 커밋"
    branch no-conflict-b
    checkout no-conflict-b
    commit id: "Flex"
    checkout no-conflict-a
    commit id: "제목에 에모지 추가"
    merge no-conflict-b id: "머지 커밋"
~~~~

하지만 이는 우리가 원하는 게 아닙니다. 우리의 목표는 리베이스를 하여 다음과 같은
커밋 그래프를 만드는 것입니다:

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: no-conflict-a
---
gitGraph BT:
    commit id: "첫 커밋"
    commit id: "Flex" tag: "no-conflict-b"
    commit id: "제목에 에모지 추가"
~~~~

이를 위해서는 `no-conflict-b` 브랜치를 `no-conflict-a` 브랜치에 리베이스하면
됩니다:

~~~~ sh
git rebase origin/no-conflict-b
~~~~

리베이스는 커밋 역사를 파괴적으로 수정하기 때문에, 푸시하려고 하면 오류가
납니다:

~~~~ sh
git push origin no-conflict-a
~~~~

~~~~
To github.com:YOUR-NAME/ossca2025-git-rebase-lecture.git
 ! [rejected]        no-conflict-a -> no-conflict-a (non-fast-forward)
error: failed to push some refs to 'github.com:YOUR-NAME/ossca2025-git-rebase-lecture.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
~~~~

따라서 `-f`/`--force` 옵션을 함께 줘야 푸시할 수 있습니다:

~~~~ sh
git push --force origin no-conflict-a
~~~~


충돌 있는 경우
--------------

다음으로 실습할 리베이스는 충돌이 있는 경우입니다. 우선, 다음과 같이 공통된
과거 커밋을 가진 두 개의 브랜치가 있습니다:

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: w-conflict-a
---
gitGraph BT:
    commit id: "첫 커밋"
    branch w-conflict-b
    checkout w-conflict-b
    commit id: "'모두들' 추가"
    checkout w-conflict-a
    commit id: "Flex"
~~~~

일단 우리는 `w-conflict-a` 브랜치에 있다고 가정합시다:

~~~~ sh
git switch w-conflict-a
~~~~

이번 목표는 리베이스를 하여 다음과 같은 커밋 그래프를 만드는 것입니다:

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: w-conflict-a
---
gitGraph BT:
    commit id: "첫 커밋"
    commit id: "'모두들' 추가" tag: "w-conflict-b"
    commit id: "Flex"
~~~~

아까와 마찬가지로 `git rebase` 커맨드를 이용하면 됩니다:

~~~~ sh
git rebase origin/w-conflict-b
~~~~

하지만 *README.md* 파일에 충돌이 있기 때문에 다음과 같은 메시지가 뜰 겁니다:

~~~~
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
error: could not apply 920b76b... Flex
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
hint: Disable this message with "git config set advice.mergeConflict false"
Could not apply 920b76b... # Flex
~~~~

현재 어떤 상황인지 파악하기 위해 `git status` 커맨드를 실행해 봅니다:

~~~~
interactive rebase in progress; onto f67d9aa
Last command done (1 command done):
   pick 920b76b # Flex
No commands remaining.
You are currently rebasing branch 'w-conflict-a' on 'f67d9aa'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
	both modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
~~~~

사실 위 메시지 안에 여러분이 어떤 조치를 취해야 할 지 다 써져 있습니다.
`both modified:   README.md`라는 메시지는 *README.md* 파일이 `w-conflict-a` 및
`w-conflict-b` 브랜치 양쪽에서 수정되었기 때문에 충돌한다는 표시입니다.

이 때는 *README.md* 파일을 열어 수동으로 충돌을 해결해 주어야 합니다.
*README.md* 파일을 열어보면 파일 내용이 다음과 같이 되어 있는 것을 볼 수
있습니다:

~~~~ markdown
Git 리베이스 실습
=================

이 파일은 Git 리베이스 실습을 위한 것입니다.

이 실습을 통해 Git 리베이스의 기본 개념과 사용법을 익힐 수 있습니다.

<<<<<<< HEAD
모두들 화이팅!
||||||| parent of 920b76b (Flex)
화이팅!
=======
화이팅! 💪
>>>>>>> 920b76b (Flex)
~~~~

 -  **`<<<<<<<`부터 `|||||||`까지**가 리베이스 하려고 했던 `w-conflict-b`에서
    만들어진 변경이고,
 -  **`|||||||`부터 `=======`까지**가 `w-conflict-a`와 `w-conflict-b`의 공통
    부모 커밋일 때의 내용, 즉 양쪽으로부터 아무런 변경이 적용되기 이전의
    내용이고,
 -  **`=======`부터 `>>>>>>>`까지가 리베이스 되려고 하는 `w-conflict-a`,
    즉 현재 브랜치에서 변경된 내용입니다.

이를 정리하면 다음과 같습니다:

 -  **`w-conflict-a` 브랜치**(현재 브랜치)에서는 `화이팅!`이 `화이팅! 💪`으로
    바뀌었습니다.
 -  **`w-conflict-b` 브랜치**에서는 `화이팅!`이 `모두들 화이팅!`으로
    바뀌었습니다.

즉, 양쪽의 변경을 모두 합치면 `모두들 화이팅! 💪` 정도가 될 것입니다. 따라서
*README.md* 파일의 내용을 다음과 같이 변경해야 합니다:

~~~~ markdown
Git 리베이스 실습
=================

이 파일은 Git 리베이스 실습을 위한 것입니다.

이 실습을 통해 Git 리베이스의 기본 개념과 사용법을 익힐 수 있습니다.

모두들 화이팅! 💪
~~~~

이렇게 양쪽의 변경을 사람이 직접 합치는 작업을 **충돌 해결**(conflict
resolution)이라고 합니다. 충돌을 해결했으면, Git에게 충돌한 파일이 해결되었다고
표시해 주어야 합니다. `git add` 커맨드로 해결되었다는 표시를 해 줄 수 있습니다:

~~~~ sh
git add README.md
~~~~

충돌을 해결했으면 다시 `git status` 커맨드로 현재 상황을 파악해 봅니다:

~~~~
interactive rebase in progress; onto f67d9aa
Last command done (1 command done):
   pick 920b76b # Flex
No commands remaining.
You are currently rebasing branch 'w-conflict-a' on 'f67d9aa'.
  (all conflicts fixed: run "git rebase --continue")

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   README.md

~~~~

이제는 `modified:   README.md`로 상태가 바뀌어 있는 것을 볼 수 있습니다.
충돌이 해결되었으니 이제 리베이스를 재개해야 합니다. 다음 커맨드로 재개할 수
있습니다:

~~~~ sh
git rebase --continue
~~~~

위 커맨드를 입력하면 커밋 메시지를 입력하는 편집기가 뜹니다. 커밋 메시지는 이미
채워져 있기 때문에 대부분 그대로 두면 됩니다.

이제 리베이스가 완료되었습니다. `git log` 커맨드나 Git 역사를 GUI로 보여주는
앱을 통해 커밋이 다음과 같이 잘 구성되었는지 확인해 봅시다:

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: w-conflict-a
---
gitGraph BT:
    commit id: "첫 커밋"
    commit id: "'모두들' 추가" tag: "w-conflict-b"
    commit id: "Flex"
~~~~

앞서와 마찬가지로, 리베이스는 Git의 커밋 역사를 파괴적으로 수정하기 때문에
그냥 `git push`를 하려고 하면 오류가 납니다. `-f`/`--force` 옵션을 주어
강제로 푸시합니다:

~~~~ sh
git push --force origin w-conflict-a
~~~~


커밋 합치기
-----------

마지막으로 실습할 것을 2개 이상의 커밋을 하나로 합치는 것입니다. 다음과 같은 커밋들이 있습니다:

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: fixup
---
gitGraph BT:
    commit id: "첫 커밋"
    commit id: "'모두들' 추가"
    commit id: "파일 → 문서"
    commit id: '"리베이스"에 "rebase" 영문 병기'
    commit id: "'모두들' 오타 수정"
~~~~

일단 우리는 `fixup` 브랜치에 있다고 가정합시다:

~~~~ sh
git switch fixup
~~~~

두 번째 커밋인 `'모두들' 추가`와 마지막 커밋인 `'모두들' 오타 수정`을 다음과 같이 합치는 것이 목표입니다:

~~~~ mermaid
---
config:
  gitGraph:
    mainBranchName: fixup
---
gitGraph BT:
    commit id: "첫 커밋"
    commit id: "'모두들' 추가" tag: "합쳐짐: '모두들' 오타 수정"
    commit id: "파일 → 문서"
    commit id: '"리베이스"에 "rebase" 영문 병기'
~~~~

우선 현재의 커밋 목록을 `git log --graph` 커맨드로 확인합시다 (최신순으로 나옵니다):

~~~~
* commit fd7d20fe822f10ffee308a05c688e2436384dea5
| Author: Hong Minhee <hong@minhee.org>
| Date:   2025-07-19 11:51:50 +0900
| 
|     '모두들' 오타 수정
| 
* commit a8a8830062223c2b5b5444bf5aa44dddade0467b
| Author: Hong Minhee <hong@minhee.org>
| Date:   2025-07-19 11:51:23 +0900
| 
|     "리베이스"에 "rebase" 영문 병기
| 
* commit bbec26eab187183bf2e731c99a6b74dfa7b1914f
| Author: Hong Minhee <hong@minhee.org>
| Date:   2025-07-19 11:50:47 +0900
| 
|     파일 → 문서
| 
* commit 9aae45aba657f3132744402b3cc066bf86542d12
| Author: Hong Minhee <hong@minhee.org>
| Date:   2025-07-19 11:49:52 +0900
| 
|     '모두들' 추가
| 
* commit bc48c3fef401e08fdf23f3cc17a8c4c5a9869554
  Author: Hong Minhee <hong@minhee.org>
  Date:   2025-07-18 10:45:57 +0900
  
      첫 커밋
~~~~

우리는 `fd7d20fe822f10ffee308a05c688e2436384dea5` 커밋(`'모두들' 오타 수정`)을 `9aae45aba657f3132744402b3cc066bf86542d12` 커밋(`'모두들' 추가`)에 합치고자 합니다. 이를 위해서는 `git rebase` 커맨드의 `-i`/`--interactive` 옵션을 써야 합니다.

우리가 수정하려는 가장 오래된 커밋이 `9aae45aba657f3132744402b3cc066bf86542d12` 커밋(`'모두들' 추가`)이므로, 그보다 바로 앞 커밋인 `bc48c3fef401e08fdf23f3cc17a8c4c5a9869554` 커밋(`첫 커밋`)을 대상으로 인터랙티브 리베이스를 실행합니다:

~~~~ sh
git rebase --interactive bc48c3fef401e08fdf23f3cc17a8c4c5a9869554
~~~~

그러면 편집기가 실행되면서 다음과 같은 텍스트를 조작하게 됩니다. 첫 줄이 가장 오래된 커밋이고, 마지막 줄이 가장 최신 커밋입니다:

~~~~
pick 9aae45a # '모두들' 추가
pick bbec26e # 파일 → 문서
pick a8a8830 # "리베이스"에 "rebase" 영문 병기
pick fd7d20f # '모두들' 오타 수정

# Rebase bc48c3f..fd7d20f onto bc48c3f (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
#         create a merge commit using the original merge commit's
#         message (or the oneline, if no original merge commit was
#         specified); use -c <commit> to reword the commit message
# u, update-ref <ref> = track a placeholder for the <ref> to be updated
#                       to this position in the new commits. The <ref> is
#                       updated at the end of the rebase
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
~~~~

그럼 이 텍스트를 다음과 같이 편집합니다. `fd7d20f` 커밋(`'모두들' 오타 수정`)을 `9aae45a` 커밋(`'모두들' 추가`) 바로 아래로 올리고, `pick`이라는 키워드를 `fixup`으로 바꿉니다:

~~~~
pick 9aae45a # '모두들' 추가
fixup fd7d20f # '모두들' 오타 수정 ← 위치가 바뀌고 pick → fixup으로 고침
pick bbec26e # 파일 → 문서
pick a8a8830 # "리베이스"에 "rebase" 영문 병기
~~~~

텍스트를 저장하고 편집기를 종료하면 리베이스가 진행됩니다:

~~~~
Successfully rebased and updated refs/heads/fixup.
~~~~

다시 한 번 `git log --graph` 커맨드를 실행해 보면, 아래와 같이 커밋이 4개로 하나 준 것을 확인할 수 있습니다(각각의 커밋 ID는 아래와 다를 수 있습니다):

~~~~
* commit 180989f281575ffa2991905e0ad0f59d265ec55c
| Author: Hong Minhee <hong@minhee.org>
| Date:   2025-07-19 11:51:23 +0900
| 
|     "리베이스"에 "rebase" 영문 병기
| 
* commit 7f40f552e91ddd5ea2f42e602149c99fbd0b3f1e
| Author: Hong Minhee <hong@minhee.org>
| Date:   2025-07-19 11:50:47 +0900
| 
|     파일 → 문서
| 
* commit 4dfcf927045d02cad9713ed1044d6214854488d9
| Author: Hong Minhee <hong@minhee.org>
| Date:   2025-07-19 11:49:52 +0900
| 
|     '모두들' 추가
| 
* commit bc48c3fef401e08fdf23f3cc17a8c4c5a9869554
  Author: Hong Minhee <hong@minhee.org>
  Date:   2025-07-18 10:45:57 +0900
  
      첫 커밋
~~~~

`'모두들' 추가`에 해당하는 커밋(커밋 ID는 여러분의 `git log` 커맨드 결과에서 복사하세요)을 `git show` 커맨드로 확인하면 2개의 수정이 하나로 합쳐진 것을 확인할 수 있습니다:

~~~~ sh
git show 4dfcf927045d02cad9713ed1044d6214854488d9
~~~~

~~~~ diff
commit 4dfcf927045d02cad9713ed1044d6214854488d9
Author: Hong Minhee <hong@minhee.org>
Date:   2025-07-19 11:49:52 +0900

    '모두들' 추가

diff --git a/README.md b/README.md
index 4cff94d..9cda9da 100644
--- a/README.md
+++ b/README.md
@@ -5,4 +5,4 @@ Git 리베이스 실습
 
 이 실습을 통해 Git 리베이스의 기본 개념과 사용법을 익힐 수 있습니다.
 
-화이팅!
+모두들 화이팅!
~~~~


참고 자료
---------

리베이스에 대해 좀 더 자세히 알고 싶은 분들께는 다음 자료들을 추천합니다:

- [기록 다시 쓰기](https://www.atlassian.com/ko/git/tutorials/rewriting-history)
- [Git 도구 - 히스토리 단장하기](https://git-scm.com/book/ko/v2/Git-%eb%8f%84%ea%b5%ac-%ed%9e%88%ec%8a%a4%ed%86%a0%eb%a6%ac-%eb%8b%a8%ec%9e%a5%ed%95%98%ea%b8%b0)
- [`git-rebase`](https://git-scm.com/docs/git-rebase)
