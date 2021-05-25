# git merge와 rebase의 차이, 적절하게 사용하기

## 1. merge

master branch 에 commit 0 ~ 2 가 있는 상태에서, dev branch 를 생성하여 commit 3, 4 를 진행했을 때의 구조는 아래와 같다.

```
[MASTER] ---> [commit2] ---> [commit1] ---> [commit0]
              |
              |
              [commit3]
              |
              [commit4]
              |
              [-b DEV]
```

이 상태에서 `git checkout master` -> `git merge dev` 로 병합하면,
[commit2] 를 가리키고 있던 master branch 가 fast-forward 되어 dev와 같은 commit 을 가리키게 되며, 구조는 아래와 같이 변경된다.

```
[MASTER] --> [commit4]---> [commit3] ---> [commit2] ---> [commit1] ---> [commit0]
              |
              |
              [-b DEV]

```

동일한 시나리오에서, 병합하기 전에 master branch 에서도 추가 commit 이 발생했을 경우,
merge 하게 되면 fast-forward 하는 대신 새로운 merge commit [commit6] 을 생성한다.

```
[MASTER] --> [commit5] ---> [commit2] ---> [commit1] ---> [commit0]
                            |
                            |
                            [commit3]
                            |
                            [commit4]
                            |
                            [-b DEV]
```

```
[MASTER] --> [commit6] --> [commit5] ---> [commit2] ---> [commit1] ---> [commit0]
              |                            |
              |                            |
              |                           [commit3]
              |                            |
              |                           [commit4]
              |<----------------------------
              [-b DEV]
```

위와 같이 merge commit 이 생성되는 것은 즉 merge graph 에 가독성을 해치는 commit log 가 찍히는 것이기 때문에, 불필요한 commit object 의 생성을 방지하기 위해 rebase 를 사용할 수 있다.

**cf) merge - fast-forward 와 non-fast-forward**
1-1. fast-forward :

![img_01](../z.images/git_ff.JPG)

1-2. merge non-fast-forward:

![img_01](../z.images/git_nff.JPG)

## 2. rebase:

![img_01](../z.images/git_rebase.JPG)
rebase 는 말 그대로, base 를 다시 설정한다는 의미이다.

merge commit 이 일어났던 동일 시나리오에서 rebase 를 진행할 시, [commit5] 가 base commit 으로 변환된다. (`git checkout dev` -> `git rebase master`)

```
[MASTER] --> [commit5] ---> [commit2] ---> [commit1] ---> [commit0]
              |
              |
              [commit3]
              |
              [commit4]
              |
              [-b DEV]
```

이 상태에서 `git checkout master` -> `git merge dev` 할 시 merge commit 이 생기지 않고 fast-forward 가 발생할 수 있다.

브랜치에서 작업한 내용을 바로 merge 시킬 때에는 git merge 명령어를 이용하지만,
일반적으로, 다른 팀원들의 review & confirm 이 필요할 경우 push -> pull request 를 요청한다. (`git rebase master -> git push dev`)

### References

1. https://cyberx.tistory.com/96
