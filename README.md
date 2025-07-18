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
