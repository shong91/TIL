# Git bash 명령어 정리 


## 기본 명령어 

* 커맨드 창 화면 초기화 
```
Ctrl + L
```

* 명령어 맨 앞 / 맨 뒤로 이동 
```
Ctrl + A
Ctrl + E
``` 

* 디렉토리 이동 
```
$ cd ${directory_name}
```

* 디렉토리 생성
```
$ mkdir ${directory_name}
```

* 디렉토리 삭제
```
$ git rm -r ${directory_name}
```

* 디렉토리 목록 조회
```
$ dir
$ ls
```

* 파일 내용 조회 
```
$ cat ${file_name}
```


## github 계정 정보 세팅 - github new repository 생성 후


* 1. 리모트 저장소에 Github 사용자 정보 세팅
```
$ git config [--global] user.name "Full name"
$ git config [--global] user.email "email@email.com"
```

## 2. 리모트 저장소 연결 

* 리모트 저장소에 Github 원격 저장소 연결 정보 추가 
```
$ git remote add origin ${자신의 github 원격 저장소 주소}
```

* 3. 리모트 저장소 초기화 
```
$ git init
```

* 4. 리모트 저장소 연결 정보 조회 
```
$ git remote show ${리모트 저장소 이름}
```

* 아이디/비밀번호 저장하기 

원격 저장소 서버에 접속할 때 마다 입력하기 귀찮으니까 
```
$ git config credential.helper store
```

* 리모트 저장소에서 데이터 가져오기 

fetch 명령어는 리모트 저장소의 데이터를 모두 로컬로 가져오지만, 자동으로 merge 하지 않는다. 
pull 명령어는 데이터를 가져오고 자동으로 로컬 브랜치와 merge 시킬 수 있다. 

```
$ git fetch origin
$ git pull origin main
```

* 파일의 상태 확인하기
```
$ git status
```

* 파일을 새로 추적하기

Untracked 상태의 파일을 Tracked & Staged 상태로 변경하여 해당 파일을 commit 할 수 있도록 한다. 
```
$ git add [파일명]
```

* 변경 사항 커밋하기

Staging area 에 올라온 파일들을 커밋한다. 
```
$ git commit -m "add comments "
$ git commit -am "add comments "

[-m] 메시지를 적어주지 않으면 vim 이 실행되니 유의!
[-am] Tracked 상태의 파일을 자동으로 Staging area에 넣어주는 옵션. (a 파일 수정 -> add -> 추가수정 -> 다시 add 하는 수고를 덜 수 있다 )

```

* 파일 삭제하기 
Staging area에서 삭제 & 워킹 디렉토리에서도 실제 삭제된다. 
```
$ git rm [파일명]
```

Staging area에서만 제거하고, 워킹 디렉토리에는 남겨두고 싶은 경우 
```
$ git rm -- cached [파일명]
```

* 변경된 내용 발행하기 

push 하여 리모트 저장소에 변경사항을 반영한다. 
```
$ git push -u origin main (master -> main 으로 변경됨)
git push [-u: 원격 저장소로부터 업데이트 받은 후 push] [리모트 저장소] [브랜치 이름]
```

충돌 발생 시, 원격 저장소와 로컬의 충돌 내용을 확인, 수정 후 해당 파일을 add - commit - push 한다. 

## 브랜치

* 브랜치 확인
```
$ git branch [-r]
[-r] : 서버 브랜치 확인 옵션
```

* 브랜치 생성 & 전환
```
$ git branch [브랜치명]
$ git checkout [브랜치명]

$ git checkout -b [브랜치명]
브랜치 생성 & 체크아웃 한번에 가능
``

* 로컬 저장소에 생성한 브랜치를 원격 저장소로 push
```
$ git push --set-upstream origin [브랜치명]
```

* 브랜치 이름 변경 
```
$ git branch -m <new_name_branch>
```


## 자주 사용되는 git 명령어 정리 
```
$ git branch -> 로컬 branch 확인 
$ git branch -r 서버 branch 확인 
$ git checkout -b 브랜치명 브랜치를 만들고 바로 이동 
$ git branch -d(D) test 브랜치 삭제 
$ git status 현재상태(머지나 추가사항) 확인 
$ git add 경로 에러를 해결하고 추가하여 에러해결 
$ git stash 임시저장 
$ git stash pop 임시저장한파일 불러오기 
$ git remote prune origin 깃랩에서 삭제한거 서버와 동기화 
$ git push origin :브랜치네임 서버에서 삭제하기 
$ git remote 
$ git push origin dev 
$ git config http.postBuffer 104857600 git오류시 해결 
$ git merge --squash dev $ git merge --no-ff feature- : 새로운 가지 따서 merge(관리상 용이) 
$ git clone 주소 
$ git remote set-url origin 주소 : gitlap 저장소 변경시 설정 
$ git remote -v : gitlap 저장소 주소 확인 // 고아 브랜치 만드는 방법 $ git checkout master 
$ git checkout --orphan c_YYMMDD_CAMPAIGNNAME $ git rm -rf . 
$ git push origin c_YYMMDD_CAMPAIGNNAME

```


### References
1. https://webclub.tistory.com/317 [Web Club]
