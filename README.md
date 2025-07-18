OSSCA 2025 Git 리베이스 실습
============================

이 저장소는 OSSCA 2025 멘티들을 위한 Git 리베이스 실습 교보재입니다.


충돌 없는 경우
--------------

가장 먼저 실습할 리베이스는 충돌이 없는 경우입니다. 우선, 다음과 같이 공통된
과거 커밋을 가진 두 개의 브랜치가 있습니다:

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
git rebase no-conflict-b
~~~~
