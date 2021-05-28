## git stash

아직 마무리하지 않은 작업을 스택에 잠시 저장할 수 있도록 하는 명령어.

이를 통해 아직 완료하지 않은 일을 commit 하지 않고 나중에 다시 꺼내와 마무리할 수 있다.
(잠시 브랜치를 변경하여 필요한 작업을 하는 일 등을 가능하게 한다. )

stash 란 아래에 해당하는 파일들을 보관해두는 장소이다.

1. modified 이면서 tracked 상태인 파일
2. staging area에 있는 파일

## git stash 명령어 모음

- 하던 작업 임시 저장하기

스택에 새로운 stash 를 생성하며, 이를 통해 working directory 는 깨끗해진다.

```
$ git stash
$ git stash save
```

- stash 목록 확인

```
$ git stash list
```

- stash 적용 (했던 작업 다시 가져오기)

stash 명을 적지 않으면 가장 최근의 stash 를 가져온다.

기본적으로 staged 상태까지는 가져오지 않으며, --index 옵션을 추가하여 staged 상태까지 복원할 수 있다.

```
$ git stash apply [stash 이름]
$ git stash apply --index
```

- stash 제거

```
$ git stash drop [stash 이름]
$ git stash pop // pop = apply + drop
```

- stash 되돌리기

잘못 적용한 stash 를 되돌린다.

```
// 가장 최근의 stash 를 사용하여 패치를 만들고, 이를 거꾸로 적용한다.
$ git stash show -p | git apply -R
$ git stash show -p [stash 이름] | git apply -R
```

#### References:

1. https://gmlwjd9405.github.io/2018/05/18/git-stash.html
