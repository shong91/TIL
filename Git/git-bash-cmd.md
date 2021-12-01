# Git bash 명령어 정리

## 기본 명령어

- 커맨드 창 화면 초기화

```
Ctrl + L
```

- 명령어 맨 앞 / 맨 뒤로 이동

```
Ctrl + A
Ctrl + E
```

- 디렉토리 이동

```
$ cd ${directory_name}
```

- 디렉토리 생성

```
$ mkdir ${directory_name}
```

- 디렉토리 삭제

```
$ git rm -r ${directory_name}
```

- 디렉토리 목록 조회

```
$ dir
$ ls
```

- 파일 내용 조회

```
$ cat ${file_name}
```

## github 계정 정보 세팅 - github new repository 생성 후

- 1. 리모트 저장소에 Github 사용자 정보 세팅

```
$ git config [--global] user.name "Full name"
$ git config [--global] user.email "email@email.com"
```

## 2. 리모트 저장소 연결

- 리모트 저장소에 Github 원격 저장소 연결 정보 추가

```
$ git remote add origin ${자신의 github 원격 저장소 주소}
```

- 리모트 저장소 초기화

```
$ git init
```

- 리모트 저장소 연결 정보 조회

```
$ git remote show ${리모트 저장소 이름}
```

- 아이디/비밀번호 저장하기

원격 저장소 서버에 접속할 때 마다 입력하기 귀찮으니까

```
$ git config credential.helper store
```

- 리모트 저장소에서 데이터 가져오기

fetch 명령어는 리모트 저장소의 데이터를 모두 로컬로 가져오지만, 자동으로 merge 하지 않는다.
pull 명령어는 데이터를 가져오고 자동으로 로컬 브랜치와 merge 시킬 수 있다.

```
$ git fetch origin
$ git pull origin main
```

- 리모트 저장소와 연결 시 Git refusing to merge unrelated histories on rebase

  리모트 레파지토리 initial create 시 README.md 파일 등 초기 생성 파일이 함께 생성되어, 로컬 저장소와 sync 되지 않아 생기는 문제.

  `--allow-unrelated-histories` 옵션을 추가하여 주면 된다.

```
$ git pull origin main --allow-unrelated-histories
```

- 파일의 상태 확인하기

```
$ git status
```

- 파일을 새로 추적하기

Untracked 상태의 파일을 Tracked & Staged 상태로 변경하여 해당 파일을 commit 할 수 있도록 한다.

```
$ git add [파일명]
```

- 변경 사항 커밋하기

Staging area 에 올라온 파일들을 커밋한다.

```
$ git commit -m "add comments "
$ git commit -am "add comments "

[-m] 메시지를 적어주지 않으면 vim 이 실행되니 유의!
[-am] Tracked 상태의 파일을 자동으로 Staging area에 넣어주는 옵션. (a 파일 수정 -> add -> 추가수정 -> 다시 add 하는 수고를 덜 수 있다 )

```

- 파일 삭제하기
  Staging area에서 삭제 & 워킹 디렉토리에서도 실제 삭제된다.

```
$ git rm [파일명]
```

Staging area에서만 제거하고, 워킹 디렉토리에는 남겨두고 싶은 경우

```
$ git rm -- cached [파일명]
```

- git add 취소하기

뒤에 파일명이 없으면 add 한 모든 파일을 취소한다.

```
$ git reset [파일명]
```

- git commit 취소하기

```
$ git reset --soft HEAD^
$ git reset --mixed HEAD^ // 기본 옵션
$ git reset HEAD~2 // 마지막 2개의 commit을 취소
$ git reset --hard HEAD^

```

| 모드명 | HEAD의 위치 | 인덱스 | 작업 디렉토리 |
| ------ | ----------- | ------ | ------------- |
| soft   | 변경        | 변경X  | 변경X         |
| mixed  | 변경        | 변경   | 변경X         |
| hard   | 변경        | 변경   | 변경          |

**reset 옵션**

– soft : index 보존(add한 상태, staged 상태), 워킹 디렉터리의 파일 보존. 즉 모두 보존.

– mixed : index 취소(add하기 전 상태, unstaged 상태), 워킹 디렉터리의 파일 보존 (기본 옵션)

– hard : index 취소(add하기 전 상태, unstaged 상태), 워킹 디렉터리의 파일 삭제. 즉 모두 취소.

- 상태 되돌리기: revert
  결과는 reset --mixed와 동일하나, 이력은 남는다. (reset 은 이력도 지워진다. )

```
$ git revert [되돌릴 커밋]
```

**revert와 reset, 어떨 때 사용하면 좋을까?**

reset 은 이력을 함께 지우기 때문에 이력을 단순하게 만들어준다는 점에서 장점이 있다.

하지만, 예를 들어, 이력 중간에 로그 출력하도록 한 커밋이 있고, 그 커밋만을 취소하려고 한다면 reset 을 사용하여 이후의 이력을 모두 제거하는 것은 이후 이력을 모두 날려버리는 결과를 낳는다.

이 때 revert 를 사용하여 해당 커밋의 내용만들 되돌릴 수 있다.

또한, 이미 원격 리파지토리에 push 한 상태라면, reset 을 사용하면 reset 하기 이전으로 되돌리기 전까지는 push 할 수 없다. (-force 옵션이 있기는 하나, 이미 push 한 코드라면 revert 를 사용하는 것을 권장. )

- commit 메시지 변경하기

```
$ git commit --amend [변경할 커밋 메시지]
```

- 변경된 내용 발행하기

push 하여 리모트 저장소에 변경사항을 반영한다.

```
$ git push -u origin main (master -> main 으로 변경됨)
git push [-u: 원격 저장소로부터 업데이트 받은 후 push] [리모트 저장소] [브랜치 이름]
```

충돌 발생 시, 원격 저장소와 로컬의 충돌 내용을 확인, 수정 후 해당 파일을 add - commit - push 한다.

- 로그 확인하기

```
$ git log
$ git log --graph // 그래프로 확인하기
```

## 브랜치

- 브랜치 확인

```
$ git branch [-r]
[-r] : 서버 브랜치 확인 옵션
```

- 브랜치 생성 & 전환

```
$ git branch [브랜치명]
$ git checkout [브랜치명]

$ git checkout -b [브랜치명]
브랜치 생성 & 체크아웃 한번에 가능
```

- 로컬 저장소에 생성한 브랜치를 원격 저장소로 push

```
$ git push --set-upstream origin [브랜치명]
```

- 원격 저장소의 브랜치를 로컬로 가져오기

```
$ git branch -t [브랜치명]
```

- 브랜치 이름 변경

```
$ git branch -m [new_branchname]
```

- 브랜치 병합

```
$ git checkout [브랜치명]
$ git merge [--squash/--no-ff] [병합할 대상 브랜치]
// --squash: 대상 브랜치의 모든 커밋을 하나의 커밋으로 합쳐서 merge => 간략한 히스토리 관리 시 사용
// --no-ff: fast-forward 관계에서 merge 를 실행 시 merge commit 이 생략되는데(--ff), --no-ff 옵션을 추가 시 merge commit 을 생성한다.
```

### References

1. https://webclub.tistory.com/317 [Web Club]
2. https://gmlwjd9405.github.io/2018/05/25/git-add-cancle.html
3. https://www.devpools.kr/2017/02/05/%EC%B4%88%EB%B3%B4%EC%9A%A9-git-%EB%90%98%EB%8F%8C%EB%A6%AC%EA%B8%B0-reset-revert/
