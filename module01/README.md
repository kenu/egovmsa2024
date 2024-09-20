# GitHub 원격 저장소(Remote)

### 원격 저장소
* GitHub, GitLab, BitBucket 등 서버(remote repository)를 통해서 코드를 공유
* 원격 저장소에 아이디와 이메일을 등록하고, 이메일 인증을 마치면 사용 가능
* 조직(Organization)을 만들어서 팀으로 공동 관리 가능

### 아키텍처
* 원격 저장소에 브랜치를 보낼 수 있음
* 원격 저장소의 최종 변경 사항을 로컬에 적용해야 보낼 수 있음
* 그렇지 않으면 `non-fast-forward` 예외
* `--force` 옵션으로 원격의 히스토리를 이전으로 돌릴 수 있지만, 최종 변경 사항이 사라지기 때문에 사용 주의 필요

### GitHub
* https://github.com 에 계정을 만들고, 이메일 인증
* 로그인하고 새로운 저장소 생성
* 로컬과 원격 저장소 연결
* 코드 Sync

#### GitHub 계정 생성
1. Sign up: https://github.com/signup
2. Email, password, username 입력
3. Email로 온 인증 숫자 입력
4. 가입 완료

#### 새로운 원격 저장소 생성
* GitHub 로그인 후 `Create repository` 버튼 클릭
* https://github.com/new

#### 프로젝트 생성
1. Repository name에 `okchat` 입력
2. `Add a README file` 체크 안함
3. .gitignore 언어별 선택 안함
4. license 선택 안함
5. `Create repository` 클릭으로 생성

#### 원격 저장소 주소 복사
* 우측 복사 버튼 클릭
* HTTPS 주소 복사

#### 원격 저장소 연결
1. 로컬 프로젝트와 원격 저장소 연결
2. `cd ~/git/okchat` 폴더로 이동
3. `git remote add origin https://github.com/kenuheo/okchat.git` 명령으로 연결
   * `origin`은 다른 것으로 변경 가능한 별칭
   * 주소줄 뒤의 .git은 생략 가능
4. `git remote -v` 명령으로 설정 정보 확인

#### 코드 동기화(Sync)
* 원격 저장소의 변경사항 가져와 적용하기: `git pull origin main`
  * `origin`: 원격 저장소 URL 별칭
  * `main`: 브랜치
* 로컬의 브랜치 보내기: `git push origin main`
* 먼저 받고, 정리한 후에 보내기

#### 원격 저장소 코드 가져오기
1. 다른 컴퓨터의 경우라고 가정
2. VS Code를 종료하고, 터미널 창에서 `cd ~/git && mv okchat okchat_` 명령 실행
3. 원격 저장소 가져오기(clone): `git clone https://github.com/kenuheo/okchat`
4. `code ~/git/okchat` 으로 VS Code 편집기 실행

## 브랜치 다루기(Branch)

### 브랜치(branch)
* 분기해서 작업하는 방법
* 브랜치 만들기
* 브랜치 합치기
  * merge
  * rebase

### 브랜치 만들기
1. `develop` 브랜치를 만들어서 커밋한 후에 `main`에 합치는 예제
2. `git checkout -b develop`
3. chat.js 파일에서 `var`를 `const`로 수정
4. `git commit -am "var 2 const"`
   * 수정된 파일은 -am 옵션으로 한 번에 커밋 가능

### 브랜치 합치기
1. main 브랜치에 develop 커밋을 합치려면 기준을 main으로 잡아야 함
2. `git checkout main`
3. README.md 파일에 다음 내용 추가하고 커밋
   ```
   * simple chat server and client
   ```
4. `git commit -m "README"`

#### 브랜치 합치기 merge
* main 브랜치에 develop 브랜치 합치기
* `git merge develop`
* merge comment 창이 나타나면 `:wq` 입력해서 종료

#### 브랜치 합치기 rebase
* merge 커밋이 추가되지 않고, 분기의 기준점을 바꿈
* history가 깔끔하게 유지
* main 브랜치 상태에서 다음 명령 입력
* `git rebase develop`

### 임시 저장 stash
* 미완성 파일 임시 저장하고, 브랜치 동기화할 때 필요
* stash: 한 쪽으로 챙겨 두다
* `git stash`
* `stash list`
* `stash pop`

## VS Code 브랜치 관리 기능

### VS Code 브랜치
* 왼쪽 아래 상태바에 표시
* 클릭 후 브랜치 생성, 전환 가능
* 로컬 저장소 기준
* 원격은 git fetch로 싱크 필요

### 브랜치 전환
* 최초 `git checkout -b develop`
* 브랜치 목록: `git branch -v`
* 브랜치 전환: `git checkout main`
* 원격 브랜치 push: `git push origin develop`
* 원격 브랜치 삭제: `git push origin :develop`

## 충돌 해결하기(Conflict)

### 코드 충돌이 나는 경우 - Conflict
1. 같은 부분을 두 명이 동시에 수정하고 커밋할 때
   - 먼저 커밋한 사람은 이상 없음
   - 나중에 커밋한 사람이 해야할 일이 생김
2. 로컬에서 열심히 작업했는데, 오랜만에 서버의 최신 상태를 가져올 때
3. 대규모 파일 변경이 한꺼번에 원격 저장소에 반영된 것을 가져올 때

### 충돌 사고 현장
```
<<<<<<<
현재 내가 짠 부분

=======
가져올 부분
>>>>>>>
```

### 충돌(Conflict) 해결하기
* Accept Current Change : 위쪽, 내가 코딩한 것을 남기기
* Accept Incoming Change : 아래쪽, 선택하고 내가 코딩한 것은 포기
* Accept Both Changes : 위 아래 모두 남겨두고 `<<<`, `===`, `>>>` 라인만 삭제

### 코드 충돌을 최소화하는 방법
1. 업무 단위가 아닌 함수 단위의 커밋습관
2. 하루 3번 원격 저장소 동기화
   - 출근 직후
   - 점심 식사 후
   - 퇴근하기 1시간 전
3. 원격 저장소와 갭이 커질수록 충돌 가능성과 파일 수 증가
   * 주기적으로 원격 소스를 작업하는 로컬 코드에 반영
   * 인터벌이 길수록 충돌 확률과 범위 상승

# Git Merge와 Rebase 비교

## 개념 설명

### Merge (병합)
- 두 개의 브랜치를 하나로 합치는 방법
- 두 브랜치의 역사가 그대로 보존됨
- 새로운 "병합 커밋"이 생성됨

### Rebase (리베이스)
- 한 브랜치의 시작점을 다른 브랜치의 끝으로 옮기는 방법
- 한 브랜치의 역사가 다시 쓰여짐
- 깔끔한 일직선 형태의 히스토리가 만들어짐

## 비교 테이블

| 특성 | Merge | Rebase |
|------|-------|--------|
| 역사 보존 | 모든 브랜치 역사 유지 | 한 브랜치 역사 재작성 |
| 결과물 모양 | 다이아몬드 형태 | 직선 형태 |
| 새 커밋 생성 | 병합 커밋 생성 | 커밋 재배치 |
| 충돌 해결 | 한 번에 해결 | 커밋마다 해결 가능 |
| 주 사용 상황 | 팀 협업 | 개인 브랜치 정리 |

## 예시

### Merge 예시

```
  A---B---C (master)
       \
        D---E (feature)

  A---B---C---F (master)
       \     /
        D---E (feature)
```

1. `feature` 브랜치에서 작업 완료
2. `master` 브랜치로 이동
3. `git merge feature` 실행
4. 새로운 병합 커밋 F 생성

### Rebase 예시

```
  A---B---C (master)
       \
        D---E (feature)

  A---B---C (master)
           \
            D'---E' (feature)
```

1. `feature` 브랜치에서 작업 완료
2. `feature` 브랜치에서 `git rebase master` 실행
3. D와 E 커밋이 C 뒤로 재배치되어 D'와 E'가 됨



## 참고 자료
* [Git Book(pdf)](https://git-scm.com/book/ko/v2)
* [누구나 쉽게 이해할 수 있는 Git 입문~버전 관리를 완벽하게 이용해보자~](https://nulab.com/ko/learn/software-development/git-tutorial/)
* [생활 코딩; 지옥에서 온 Git](https://opentutorials.org/course/2708)
* [VS Code에서 쉽게 사용하는 Git](https://inf.run/LPpDg)

## Appendix: MacOS에서 VS Code 경로 설정
MacOS에서 VS Code 경로 설정:
```bash
cat << EOF >> ~/.bash_profile
export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
EOF
```
* zsh일 경우 `cat << EOF >> ~/.zshrc` 로 변경 필요

[VS Code에서 쉽게 사용하는 Git](https://inf.run/LPpDg)
