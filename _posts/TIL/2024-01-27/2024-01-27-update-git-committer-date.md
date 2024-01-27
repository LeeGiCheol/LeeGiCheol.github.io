---
title : "Git, Github 커밋 날짜 바꾸기"
categories : 
    - git 
tags :
    - git 
    - github
toc : true
---

### 원인
업무 중 개발한 내용을 원격 dev 브랜치에 merge를 하고 보니 누락된 것이 있었다.

새로 커밋을 할까, rebase squash를 할까 고민하다 squash 후 force push를 하기로 결정했다.

`이때까지는 이 방법이 좋지 않은 결과를 가지고 올 줄 몰랐다.`

이후 팀원이 pull을 하는 과정에서 conflict가 발생했고, 정리 후 다시 push를 하니

커밋이 recursive가 되어 커밋이 배로 늘어나 버렸다.

복구하기위해 다시 한번 중복되는 커밋을 squash 해주고, force push를 해줬다.

recursive는 해결됐지만, 원격 저장소 히스토리에 노출되는 커밋 날짜와 작성자가 실제 내용과 달라진 것이다.


### 삽질

구글에서 찾은 아래 방법대로 날짜와 작성자를 수정해주었다.

```bash
# rebase할 커밋 edit mode 진입
git rebase -i {COMMIT_HASH}

# Author 변경
git commit --amend --author="LeeGiCheol<leegicheolgc@gmail.com>"

# Committer Date 변경
git commit --amend --date "Sat Jan 20 23:15:06 2024 +0900"

# rebase 완료
git rebase --continue

# 원격 브랜치 push
git push -f origin dev
```

터미널에서 커밋 로그가 모두 정상인 것을 확인하고 force push를 했는데..

여전히 원격 커밋 히스토리는 날짜는 현재 시각 , 커밋 작성자는 두 명이 나왔다.

`나 authored and 최초커밋자 committerd now`

나중에 찾아보니 rebase는 커밋을 수정한다기보다는, 변경된 커밋을 새로 만들어준다는데 이게 문제인 것 같다.

---

### 해결방법 - 커밋 날짜 변경

구글검색을 계속해봤지만 내가 원하는 답변은 나오지 않았다.

그 와중에 Git Plugin중 redate 라는 것을 알게 되었는데, 혹시나 하는 마음으로 설치하고 사용해봤다.

[https://github.com/PotatoLabs/git-redate](https://github.com/PotatoLabs/git-redate)

사용법은 간단했는데, 설치 후 redate 명령어를 입력하면 커밋 히스토리가 조회되고, 

변경하면 force push를 사용하여 원격 저장소에 올릴 수 있다.

```bash
# -c: Commit 
# 3: 최근 commit 부터 3개 조회
git redate -c 3
```

![Untitled](/assets/images/study/TTL/2024-01-27/git-redate.png)


git log에서도 실제 커밋 시간이 기준이었는데 redate를 통해 날짜를 조회 해보니 
원격 저장소에 push한 시간이 조회되었다.

아래와 같이 신규 브랜치를 만들어서 날짜를 수정하고 
force push를 하니 원격 저장소에도 변경이 된 것을 확인했다.

```bash
# dev 브랜치 진입
git switch dev

# 실제 Commit 시간 수집
git show {COMMIT_HASH} --pretty=fuller

# 신규 브랜치 생성 후 이동
git switch -c hotfix/restore-dev

# redate 진입 후 수집한 시간으로 변경
git redate -c 3

# 원격 저장소 push
git push -f origin hotfix/restore-dev
```

---

### 해결방법 - 커밋 날짜, 작성자 변경

날짜는 정상적으로 변경되었는데, 커밋 작성자가 여전히 두 명이 나온다..

이건 도저히 방법을 찾을 수가 없어서 

git local config에 최초 커밋 작성자 정보를 기입하고 redate를 하는 야매를 쓰기로 했다.

```bash
# dev 브랜치 진입
git switch dev

# 실제 Commit 시간 수집
git show {COMMIT_HASH} --pretty=fuller

# 신규 브랜치 생성 후 이동
git switch -c hotfix/restore-dev

# rebase 진입 후 커밋 작성자가 꼬인 커밋 edit 모드로 변경
git rebase -i {COMMIT_HASH}

# git local config 수정
git config --local user.name 커밋 작성자 이름
git config --local user.email 커밋 작성자 이메일

# commit author 수정
git commit --amend --author="커밋 작성자 이름<커밋 작성자 이메일>"

# rebase 완료
git rebase --continue

# redate 진입 후 수집한 시간으로 변경
git redate -c 3

# 원격 저장소 push
git push -f origin hotfix/restore-dev
```

원격 저장소 push 후 다행히도 히스토리가 조회되었다.

기존 dev 브랜치를 hotfix/restore-dev 브랜치로 대체 후 모든 상황이 종료되었다.

작업중인 범위 내에서 혼자 사용하는 브랜치를 통해 rebase 하는 건 문제 없지만

공용으로 사용하는 브랜치에서 과거 작업 내용을 변경하는 건 큰 고민이 필요하다는 걸 알게 되었다.