# Merging vs. Rebasing

_본 글은 atlassian 의 Git Tutorials - Merging vs. Rebasing 을 번역한 글임을 밝힙니다._

## 소개

`git rebase` 명령어는 초보자가 멀리해야할 마법의 Git hocus pocus 라는 명성이 있지만, 주의하여 잘 사용한다면 개발 팀의 삶을 훨씬 더 쉽게 만들 수 있다.

이 글에서는, git rebase 와 git merge 명령을 비교하고, rebase 를 전형적인 Git workflow 에서 통합할 수 있는 모든 잠재적인 기회를 알아보도록 하겠다.

## 개념

가장 먼저 이해해야 하는 것은, `git rebase` 와 `git merge` 는 같은 문제를 해결한다는 것이다. 이 두 명령어는, 하나의 브랜치에서 다른 브랜치로부터의 변경 내용을 통합하기 위해 설계되었다. 하지만, 이 둘은 매우 다른 방식을 취한다.

브랜치에서 새로운 feature 에 대한 내용을 작업하기 시작한 후, 다른 팀원이 메인 브랜치를 새로운 커밋으로 업데이트 하였을 때 어떤 일이 발생하는 지 고려해보자,
이렇게 되면 브랜치 히스토리가 분기(forked history)되는데, 이는 협업 툴로 Git을 사용해본 사람이라면 매우 친숙할 것이다.

![](https://wac-cdn.atlassian.com/dam/jcr:1523084b-d05a-4f5a-bd1a-01866ec09ca3/01%20A%20forked%20commit%20history.svg?cdnVersion=1044)

자, 이제 메인에 있는 새로운 커밋이 내가 작업 중인 feature 와 관련이 있다고 가정해보자. 새로운 커밋과 나의 feature branch 를 통합하기 위하여, 우리는 두 가지 옵션이 있다: 하나는 merging 이고, 다른 하나는 rebasing 이다.

**_1. Merging_**

가장 쉬운 방법은, `main` 브랜치를 `feature` 브랜치에 병합하는 것이다.

```
git checkout feature
git merge main
```

혹은, 아래와 같이 한 줄로도 가능하다.

```
git merge feature main
```

이렇게 하면, 새로운 "merge commit" 이 feature branch 에 생성되며, 이 머지 커밋은 두 브랜치의 이력을 하나로 묶어준다. 구조를 그려보자면 아래와 같다.

![](https://wac-cdn.atlassian.com/dam/jcr:4639eeb8-e417-434a-a3f8-a972277fc66a/02%20Merging%20main%20into%20the%20feature%20branh.svg?cdnVersion=1044)

Merging 은 비파괴적인(non-destructive) 한 방식이기에 좋다. 기존의 브랜치는 어떤 방식으로도 변경되지 않는다. 이는 rebasing 에서 발생하는 모든 잠재적인 함정들를 방지한다. (rebase 의 함정에 대하여는 후술한다.)

반면에, 이는 내가 upstream changes 를 통합할 때마다 feature branch 에 불필요한 merge commit 이 생성됨을 의미한다. `main` 브랜치가 변경이 잦다면, 나의 브랜치의 이력이 merge commit 으로 인해 제법 오염될 수 있다. 향상된 `git log` 옵션을 이용해 이러한 이슈를 완화할 수 있지만, 이는 다른 개발자들이 프로젝트의 이력을 이해하는 데 어렵게 만든다.

**_2. Rebasing_**

merging 대신, 우리는 feature 브랜치를 main 브랜치로 rebase 할 수 있다.

```
git checkout feature
git rebase main
```

rebase 는 모든 `feature` 브랜치를, `main` 브랜치의 끝 지점에서부터 시작하도록 옮기는 것으로, 모든 새로운 커밋이 main 브랜치로 통합되도록 한다. 그러나, merge commit 을 사용하는 대신에, rebase 는 원본 브랜치의 각각의 커밋에 대해 새로운 커밋으로 생성함으로써, 프로젝트의 이력을 다시 쓴다(re-writes).

![](https://wac-cdn.atlassian.com/dam/jcr:3bafddf5-fd55-4320-9310-3d28f4fca3af/03%20Rebasing%20the%20feature%20branch%20into%20main.svg?cdnVersion=1044)

rebasing 의 주요 이점은, 프로젝트 이력을 훨씬 깔끔하게 관리할 수 있다는 점이다. 첫째로, `git merge` 를 사용할 때 요구되었던 불필요한 merge commit 을 제거 할 수 있다. 둘째, 위의 그림에서 보듯이, rebasing 은 완벽한 선형 프로젝트 이력을 만들어낸다. 이는 어떠한 분기(fork) 없이, 프로젝트의 시작과 feature 브랜치의 끝을 따라갈 수 있다. 즉, `git log`, `git bisect`, `gitk` 등의 명령어를 이용해 우리의 프로젝트를 탐색하기 쉬워진다는 뜻이다.

그러나, 이 깔끔한 커밋 이력에는 2개의 trade-off 가 따른다. 안전성(safety)과 추적성(traceability)이다.
만약 당신이 [Golden Rule of Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing)을 따르지 않았다면, 프로젝트 이력을 다시 쓰면(re-writing) 당신의 협업 workflow에 잠재적인 대재앙이 될 수 있다. 또한, (덜 중요하기는 하지만), rebasing 을 하면 merge commit 이 제공하는 컨텍스트를 잃게 된다. 즉, upstream changes 가 언제 feature에 통합되었는지 알 수 없게 된다.

**_3. Interactive Rebasing_**

Interactive Rebasing 은 커밋이 새로운 브랜치를 옮겨질 때에, 커밋을 변경할 기회를 준다. 이는 브랜치의 커밋 이력을 완전히 제어할 수 있기 때문에, automated rebase 보다 훨씬 더 강력하다. 전형적으로, 이는 feature 브랜치를 main 으로 병합하기 전에, 지저분한 이력을 지우는 데에 사용된다.

Interactive Rebasing 은 `-i` 옵션을 사용한다.

```
git checkout feature
git rebase -i main
```

아래와 같이 text editor 에서 옮겨질 모든 커밋이 나열된다.

```
pick 33d5b7a Message for commit #1
pick 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

이 목록은, rebase 가 수행된 후 브랜치가 어떻게 보여질 지 정의한다. `pick` 명령을 변경하거나, 항목을 재정렬하여 브랜치 이력을 원하는대로 만들 수 있다.
두 번째 커밋이 첫 번째 커밋의 작은 문제를 수정하는 경우, `fixup` 커맨드를 이용해 하나의 커밋으로 압축할 수 있다.

```
pick 33d5b7a Message for commit #1
fixup 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

editor 파일을 save/close 하면, Git 은 당신의 지시에 따라 rebase 를 수행한다. 그 결과, 프로젝트 이력은 아래 그림과 같을 것이다.

![](https://wac-cdn.atlassian.com/dam/jcr:a712e238-6cb9-4c8c-8ef7-1975dca49be3/04%20Squashing%20a%20commit%20with%20an%20interactive%20rebase.svg?cdnVersion=1044)

중요하지 않은 커밋을 제거하여, 우리의 feature 브랜치의 이력을 더욱 이해하기 쉽게 만들었다. 이는 `git merge` 로는 할 수 없는 일이다.

## Rebasing 의 황금법칙 (Golden Rule of Rebasing)

일단 당신이 rebasing 이 무엇인지 이해하면, 배워야 할 가장 중요한 것은 **_언제 쓰면 안되는지_** 이다. `git rebase`의 황금 법칙은 **public branch 에서는 사용하지 말 것** 이다.

예를 들어, main 브랜치를 feature 브랜치로 rebase 하였다고 가정하자. (rebase main onto your feature branch)

![](https://wac-cdn.atlassian.com/dam/jcr:2908e0e6-f74b-4425-b5d2-f5eca8cfcd99/05%20Rebasing%20the%20main%20branch.svg?cdnVersion=1044)

rebase 는 메인 브랜치의 모든 커밋을 feature 브랜치의 끝으로 옮긴다. 문제는, 이것이 당신의 레파지토리에서만 일어나는 일이라는 점이다. 다른 개발자들은 여전히 origin main 에서 작업을 하고 있다. rebasing 은 새로운 커밋을 만드므로, Git 은 당신의 main 브랜치의 이력은 다른 모든 브랜치의 이력과 다른 것으로 간주할 것이다.

두 메인 브랜치를 동기화하는 유일한 방법은, 그들을 다시 병합하는 것이다: 그렇게 하면 부가적인 merge commit 과, 동일한 변경 사항이 포함된 2개 세트의 커밋이 생성된다. (origin 에 있는 내역 하나, rebase 된 브랜치에 있는 내역 하나). 말할 필요도 없이, 매우 혼란스러운 상황이 벌어진다.

그러므로, `git rebase` 를 실행하기 전에는, 반드시 **"누군가가 이 브랜치를 보고 있는지?"** 확인하여야 한다. 만약 그렇다면, 키보드에서 손을 떼고, 비파괴적으로 변경할 수 있는 방법을 고민해보아야 한다. (ex) `git revert`) 그렇지 않으면, 당신이 원하는 만큼 기록을 다시 써도(re-write) 안전하다.

**_Force-Pushing_**

만약 당신이 rebase 된 `main` 브랜치를 다시 원격 레파지토리에 push 하고자 한다면, 원격 `main` 브랜치와 충돌이 일어나기 때문에, Git 은 이를 막을 것이다. 그러나, 당신은 `--force` 옵션을 이용하여 push 를 강제로 실행할 수 있다.

```
# Be very careful with this command! git push --force
```

이렇게 하면 원격 `main` 브랜치를 레파지토리의 rebased 브랜치와 일치하도록 덮어쓴다. 그리고 이것은 남은 팀원들을 매우 혼란스럽게 만든다!
그러므로, `--force` 옵션을 사용할 때에는 당신이 무엇을 하고 있는지 명확히 알고 있을 때에만, 매우 주의하여 사용하여야 한다.

강제로 push 하여야 하는 유일한 경우는, private feature 브랜치를 원격 레파지토리에 푸시한 후, 로컬을 정리하는(cleanup) 경우이다. (ex. 백업 목적)

이는 "죄송합니다. 원래 버전의 feature 브랜치를 푸시하고 싶지 않았습니다. 대신 현재 버전을 사용하세요" 라고 말하는 것과 같다. 다시 한 번 말하지만, 아무도 원래 버전의 feature 브랜치에서 커밋하지 않는 것이 중요하다.

## References.

- https://www.atlassian.com/git/tutorials/merging-vs-rebasing
