---
title: "git rebase 란? - 브랜치 히스토리 이쁘게 만들기"
date: 2024-04-12 19:04:30 +0900
categories: [Project, Git]
tags: [Project, Git, Rebase]
---

## Git rebase 란?

---

`git rebase` 는 공통 Base를 가진 두 브랜치에서(main 브랜치 & feature 브랜치) feature 브랜치의 Base 를 main 브랜치의 최신 커밋으로 Base를 옮기는 작업이다.
쉽게말해서, 커밋의 베이스를 재설정 하는 것이라고 생각하면 된다.

### 1. feature 브랜치 생성후, main 브랜치 커밋 반영

![git_rebase_1.png](/assets/img/post_img/project/git/git_rebase_1.png){: width="500" align="center"}

일반적으로 기준이 되는 Main 브랜치에 작업 커밋내용이 반영이 되면 작업중인 feature 브랜치에 Main 브랜치의 새 커밋내용을 가져오기 위해 병합(merge)을 하게 된다.

하지만 단순 병합을 하게 되면 위 그림처럼 Main 브랜치를 병합하는 과정에서 새로운 커밋이 발생하게 된다. 커밋 이력이 순서대로 정렬이 안되고 꼬이게 되고 이러헌 과정이 반복이 된다면, 프로젝트를 관리하는 측면에서 썩 달가운 상황이 아닐 것이다.

<br />

### 2. main 브랜치 최신 HEAD 를 기준으로 feature 브랜치 병합

![git_rebase_2.png](/assets/img/post_img/project/git/git_rebase_2.png){: width="500" align="center"}

feature 브랜치는 흰색 커밋을 기준으로 생성되었던 브랜치이지만, rebase 를 통해 병합(rebase)하게 되면 위 그림과 같이 마치 하늘색 커밋(Main 브랜치의 최신 커밋)을 기준으로 feature 브랜치를 생성한 것처럼 base 지점이 바뀌게 된다.

<br />

## Git Command

```bash
git checkout [feature branch]

git rebase [main branch]

git push origin [feature branch] --force
```

<br />

## git rebase 주의할 점

---

### 1. commit conflict 해소

rebase 를 하는 작업 브랜치의 각각 commit 마다 conflict 를 해결해야 한다.
merge 는 충돌이 발생하면 병합기준 한번만 충돌을 해소하면 되지만, rebase 는 base 기준으로 부터 생성된 commit 을 기준으로 각각 충돌을 해결하고 continue 해줘야 하는 번거러움이 있다.

<br />

### 2. git 강제 푸쉬

다른 곳에서도 말하는 것처럼 강제 푸쉬는 위험하다. origin 과 달라진 local 브랜치의 형상을강제로 remote 에 올려버리는 것이기 때문에 만약 다른 사람들과 같이 작업하고 있는 브랜치에 강제 푸쉬를 하게 되면 하게 되면 리모트에서도 전부 커밋의 해시값이 바뀌게 되면서 다른 사람들의 작업물에도 영향을 주게 된다. 그렇기 때문에 함부로 강제 푸쉬를 하게되면 엄청나게 꼬여버릴 수 있게 되는 것이다.
