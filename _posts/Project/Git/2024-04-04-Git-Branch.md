---
title: "협업을 위한 브랜치 전략 - Gitflow Workflow"
date: 2024-04-04 12:45:30 +0900
categories: [Project, Git]
tags: [Project, Git]
---

**Gitflow Workflow** 는 프로젝트에서 Git을 사용하여 코드를 관리하는 방식을 설명하는 것으로 가장 일반적으로 사용되는 workflow 이다. 이는 개발자들이 코드를 공유하고 협업하는 방식을 표준화하고 단순화하여 팀의 생산성을 높이는 데 도움이 된다.

Gitflow Workflow 에서는 항상 유지되는 메인 브랜치(master, develop)과 일정 기간만 유지되는 보조 브랜치(feature, release, hotfix)을 포함해 총 5가지의 브랜치를 사용한다.

아래의 그림은 Gitflow Workflow 방법론에서 사용하는 브랜치의 흐름을 보여준다.

![git_branch_1.png](/assets/img/post_img/project/git/git_branch_1.png){: width="600" align="center"}

<br />

## Master Branch

---

제품(운영)으로 출시될 수 있는 브랜치를 의미한다. 배포(Release) 이력을 관리하기 위해 사용. 즉, 배포 가능한 상태만을 관리하는 브랜치이다.

![git_branch_2.png](/assets/img/post_img/project/git/git_branch_2.png){: width="600" align="center"}

<br />

## Develop Branch

---

다음으로 출시될 버전을 개발하는 브랜치로 기능 개발을 위한 브랜치들을 병합하기 위해 사용된다.

즉, 모든 기능이 추가되고 버그가 수정되어 배포 가능한 안정적인 상태라면 **develop** 브랜치를 **master** 브랜치에 병합(merge) 한다.

평소엔 이 브랜치를 기반으로 개발이 진행된다.

![git_branch_3.png](/assets/img/post_img/project/git/git_branch_3.png){: width="600" align="center"}

<br />

## Feature Branch

---

기능을 개발하는 브랜치로서 feature 브랜치는 새로운 기능 개발 및 버그 수정이 필요할 때마다 “develop” 브랜치로부터 분기한다. feature 브랜치에서의 작업은 지본적으로 공유될 필요가 없기 때문에, 자신의 로컬 저장소에서 관리한다. 그리고 개발이 완료되면 **develop** 브랜치로 병합(merge) 하여 팀원과 개발 내용을 공유한다.

- feature branch naming
  - feature/**Jira ticket 번호** (ex. feature/ECSP-4725)
  - feature/**기능요약** (ex. feature/login)

![git_branch_4.png](/assets/img/post_img/project/git/git_branch_4.png){: width="600" align="center"}

<br />

## Release Branch

---

배포를 위한 전용 브랜치를 사용함으로써 한팀이 해당 배포를 준비하는 동안 다른 침은 다음 배포를 위한 기능 개발을 계속할 수 있다. 즉, 딱딱 끊어지는 개발 단계를 정의하기에 적합하다.

1. develop 브랜치에서 배포할 수 있는 수준의 기능이 모이면 or 정해진 배포 일정이 되면 release 브랜치를 분기해서 생성한다.
   - 생성된 release 브랜치에서는 배포를 위한 최종적인 bug fix, 문서추가 등 배포와 직접적으로 관련된 작업을 수행한다.
   - 관련된 작업을 제외하고는 release 브랜치에 새로운 기능을 추가로 병합하지 않는다.
   - release branch naming: 릴리즈 버전 (ex. release-1.0.67)
2. release 브랜치가 배포 가능한 상태가 되면, master 브랜치에 병합한다.
   - 배포가능한 상태란 **“모든 기능이 정상적으로 동작하는지 테스트가 완료된 상태”** 이다
   - release 브랜치의 변경사항이 최신화 될 수 있도록 develop 에도 병합해준다.

![git_branch_5.png](/assets/img/post_img/project/git/git_branch_5.png){: width="600" align="center"}

<br />

## Hotfix Branch

---

배포한 버전에 긴급하게 수정을 해야 할 필요가 있을 경우, master 브랜치에서 분기하는 브랜치이다.

develop 브랜치에서 문제가 되는 부분을 수정하여 배포 가능한 버전을 만들기에는 시간이 다소 소요되고 안정성을 보장하기도 어려워 배포가능한 master 브랜치에서 분기해 브랜치를 만들어 필요한 부분만을 수정한 후 다시 master 브랜치에 병합해 이를 배포한다.

1. 배포한 버전에 긴급하게 수정을 해야 할 필요가 있을 경우, hotfix 브랜치만 master 에서 바로 딸 수 있다.
   - hotfix branch naming: hotfix + 버그내용/버전 (ex. hotfix-1.2.1)
2. 작업된 hotfix 브랜치는 master 브랜치와 develop 브랜치에 각각 병합 한다.

![git_branch_6.png](/assets/img/post_img/project/git/git_branch_6.png){: width="600" align="center"}
