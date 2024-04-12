---
title: "주로 사용하는 git 명령어 모음"
date: 2024-04-12 20:23:30 +0900
categories: [Project, Git]
tags: [Project, Git]
---

## 1. git 저장소 복제

```bash
git clone [Github Repository URL] [**Directory**]
```

**Github Repository URL** 에는 클론해올 저장소의 주소를 지정해준다. Directory 에는 저장소를 복제할 파일 경로를 의미한다. 기능적으로 DIR 은 생략 가능하며, 보통 특별한 이유가 없는한 생략한다.

<br />

## 2. git 로컬 저장소에 원격 저장소 등록하기

```bash
git init

git remote add origin [Repository URL]

git push origin [Branch Name]
```

<br />

## 3. git 커밋 되돌리기

```bash
git reset --hard HEAD~1

git reset --hard [커밋 hash_code]
```

<br />

## 4. git 병합 (fast-forward / no fast-forward)

```bash
git merge -ff [병합하고자 하는 브랜치명]

git merge --no-ff [병합하고자 하는 브랜치명]
```

<br />

## 5. git 커밋합치기

```bash
git checkout [rebase 하고자 하는 브랜치명]

git rebase -i HEAD~[커밋개수] // 합치고자 하는 커밋의 개수를 의미한다.

# pick 을 s 로 변경해준뒤 저장후 종료

git push origin [rebase한 브랜치명] --force
```

<br />

## 6. git 병합

```bash
git merge [병합하고자 하는 브랜치]

git merge --abort # 병합취소

git merge --continue # 병합 계속 진행
```

<br />

## 7. git 임시저장

```bash
git stash

git stash apply
```

<br />

## 8. 기타 명령어

```bash
# 현재 상태 확인
git status 

# 전체 로그 확인 
git log 

# git 저장소 생성하기 
git init 

# 저장소에 코드 추가
git add * 

# 커밋에 파일의 변경 사항을 한번에 모두 포함  
git add -A 

# 커밋 생성
git commit -m "message" 

# 변경 사항 원격 서버 업로드 (push)
git push origin master 

# 원격 저장소의 변경 내용을 현재 디렉토리로 가져오기 (pull)
git pull 

# 변경 내용을 merge 하기 전에 바뀐 내용 비교
git diff [브랜치 이름] [다른 브랜치 이름]
```
