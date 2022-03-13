---
layout: post
title: [Linux] 자주 쓰이는 Linux 명령어 모음
tags: [Linux]
author: shong91
excerpt_separator: 
---

# 자주 쓰이는 Linux 명령어 모음

## ls: 자신이 속해있는 폴더 내에서의 파일 및 폴더들을 표시 
    - ls -l: 파일 상세 정보 표시
    - ls -a: 숨어있는 파일 표시
    - ls -t: 생성 시간별로 (desc) 표시
    - ls -rt: 생성 시간별로 (asc) 표시
    - ls -F: 유형을 나타내는 파일명을 끝에 표시 (/: 디렉토리, *: 실행파일, @: 링크)

## pwd: 현재 자신이 위치하는 디렉토리 표시

## cd: 디렉토리 이동(change directory)

## touch: 빈 파일을 생성 
    - touch [filename]: 파일 생성
    - touch -c [filename]: 파일의 시간을 현재 시간으로 변경
    - touch -t YYYYMMDDhhmm [filename]: 파일의 시간을 입력 시간으로 변경 
    - touch -r [filename1] [filename2]: filename2 의 날짜정보를 filename1 과 동일하게 변경 
    
## mkdir: 디렉토리 생성 (make directory)
    - mkdir [dirname]
    - mkdir -p [dirname/subname]: 하위 디렉토리까지 생성
    - mk -m 644 [dirname]: 특정 퍼미션을 갖는 디렉토리 생성 

## rmdir: 디렉토리 삭제 (remove directory)

## mv: 파일 이동 (move)
    - mv [fname] [mfname]: fname 의 파일을 mfname 으로 이동/변경
    - mv -b [fname] [mfname]: mfname 의 파일이 존재하면 mfname 을 백업한 뒤 이동
    - mv -f [fanme] [mfname]: mfname 의 파일이 존재하면 백업 없이 덮어쓴다. 

## rm: 파일/디렉토리 삭제 (remove)
    - rm [fname]
    - rm -f [fname]: 묻지 않고 삭제
    - rm -r [dir]: 디렉토리 삭제. (디렉토리 삭제 시에는 -r 옵션 필수)

## cp: 파일 복사
    - cp [file] [cfile]: file 을 cfile 이라는 이름으로 복사
    - cp -f [file] [cfile]: 복사할 때 복사 대상이 있을 시 지우고 강제로 복사
    - cp -R [dir] [cdir]: 디렉토리 복사. 하위 경로 및 파일까지 모두 복사한다. 

## cat: 파일 내용을 화면에 출력 
    - cat [fname]
        - head [number] [fname]
        - tail [number] [fname]
    - cat [fname1] [fname2] [fname3]
    - cat [fname1] [fname2] | more: 페이지 별 출력
    - cat [fname1] [fname2] | head: 처음부터 10번째까지만 출력
    - cat [fname1] [fname2] | tail: 끝에서 10번째까지만 출력

## find: 특정 파일/디렉토리를 검색
- find [검색경로] [-option] [파일명]
    ex) find ./ -name 'App.vue'
        find ./ -name '*.jpg' -exec rm {} \;: 결과값 삭제
        find ./ -name '*.txt' -exec sed -i 's/hi/hello/g' {} \;: 결과값의 hi -> hello 로 변경 
        find ./ -type d : [type: dir] 만 검색
        find ./ -type f | wc -l: 결과값의 개수 출력 

    
## grep: 특정 패턴을 이용해서 파일을 찾는 명령어 
- grep [-option] [pattern] [file]    
    ex) grep -r "conflict" App.vue

## redirection: 리눅스 스트림의 방향을 조정 ('>', '>>')
- 명령 > 파일: 명령의 결과를 파일로 저장
    * cat [fname1] [fname2] > [fname3]: fname1, fname2 를 출력하고 fname3 에 저장
- 명령 >> 파일: 명령의 결과를 파일에 추가
    * cat [fname4] >> [fname3]: fname3 에 fname4의 내용을 추가
- 명령 < 파일: 파일의 데이터를 명령에 입력
    * cat < [fname1]: fname1 의 내용을 출력 

## alias
- alias [newcommand] = '[command]': command 를 실행하는 새 명령어 newcommand 를 생성
    * alias ls = 'ls -l': ls 를 실행하면 ls -l 이 실행된다. 
- unalias [newcommand]: alias 해제 


* rpm: rpm 패키지를 설치하고 삭제/관리하는 명령어
* yum: 인터넷을 통하여 rpm 패키지가 저장된 서버에 접속하여 설치하고자 하는 rpm 패키지를 설치. 다른 rpm 필요 패키지까지 다 알아서 다운받아주는 유용한 명령어! 

<br>

* kill: 특정 프로세스에 특정 시그널을 포내는 명령어 
* killall: 특정 프로세스를 모두 종료
* killall5: 모든 프로세스 종료 (사용하지 말것!!))

<br>

* chmod: 특정 파일/디렉토리의 퍼미션 수정
* chown: 파일/디렉토리의 소유자, 소유 그룹 수정 (소유자, 그룹 모두 수정 가능)
* chgrp: 파일/디렉토리의 소유 그룹 수정 (그룹만 수정 가능)

- - - 

<br><br>
* vi: 에디터. 명령모드와 편집모드로 나뉨. 
    * 커서 이동
        * h : ← 이동
        * j : ↓ 이동
        * k : ↑ 이동
        * l : → 이동

    * backspace : 커서가 있는 행에서 커서를 왼쪽으로 옮김
    * space : 커서가 있는 행에서 커서를 오른쪽으로 옮김
        * `+ : 다음 행 으로 커서 이동
        * `-  : 이전 행 으로 커서 이동
        * 0 :  현재 행의 처음으로 커서 이동
        * $ : 현재 행의 끝으로 커서 이동
        * ^ : 현재 행의 첫 문자로 커서 이동(공백 무시)
        * w: 단어 단위로 커서 이동 (왼쪽 -> 오른쪽 , 위 -> 아래)
        * b : 단어 단위로 커서 이동 (오른쪽 -> 왼쪽 , 아래 -> 위)
    

    * 파일 저장
        * :w : vi 파일을 저장
        * :w 파일이름 : 파일이름으로 저장
        * :wq : 저장 후 강제 종료
        * ZZ : 저장 후 종료
        
<br>

### References:
https://vaert.tistory.com/105?category=801166
https://iamfreeman.tistory.com/entry/vi-vim-%ED%8E%B8%EC%A7%91%EA%B8%B0-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%A0%95%EB%A6%AC-%EB%8B%A8%EC%B6%95%ED%82%A4-%EB%AA%A8%EC%9D%8C-%EB%AA%A9%EB%A1%9D