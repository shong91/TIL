# git tutorial 
프로젝트 진행하는데 git 을 제대로 쓸 줄 모르다 보니 충돌 났을 때 벌벌 떨면서 삐질삐질 땀 흘리는 것도 지겹다.. (이젠 어느정도 할 줄 알긴 하지만 내가 아는 방법으로 해결 안되면 또다시 벌벌대기 시작함)

막상 해보면 그리 어렵지 않은데 자꾸 새로운 케이스가 등장해서 나를 떨게 만든다. 이참에 정리해보는 git 주요 사용법. 

push, pull, clone, commit 은 너무 간단하니 branch 에서부터. 

# branch 
- 독립적으로 작업을 진행하기 위한 개념. 각각의 브랜치는 다른 브랜치에 영향을 받지 않으며, 각각의 브랜치는 다른 브랜치와 병합하여 새로운 하나의 브랜치로 모을 수 있다. 
- default: master 브랜치


브랜치 전환하기 
```
git checkout <branchname>
```

HEAD: 현재 사용중인 브랜치의 선두 부분을 나타내는 이름. 
~(틸드), ^(캐럿) 을 사용하여 현재 커밋으로부터 특정 커밋의 위치를 가리킬 수 있다. 
ex) HEAD~~: HEAD 보다 2세대 앞의 커밋
HEAD^: 브랜치 병합에서 1번째 원본 

stash: 파일의 변경 내용을 일시적으로 기록해두는 영역. 
아직 커밋하지 않은 변경을 일시적으로 저장해둘 수 있으며, stash에 저장된 내용은 나중에 다시 불러와 브랜치에 커밋할 수 있다. 

merge:
- fast-forward : 
    A -> B[master] -> X[issue1] -> Y[master=issue1]
- non-fast-forward: 
    A -> B[master] -> C -> D -> E [master]
        X[issue1] -> Y ------->

rebase:
A -> B -> C -> D -> X' -> Y'
    X -> Y