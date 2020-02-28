---
layout: default
title: Git Fork한 저장소를 original 저장소로 동기화 시키기
parent: Git
date: 2020-02-28
comments: true

---



# Git Fork한 저장소를 original 저장소로 동기화 시키기



**※ 선행사항 (되어있으면 상관없음)**

## Fork한 저장소에 original 저장소 세팅하기

1. **git remote -v**로 현재 등록된 저장소 확인

```shell
$ git remote -v
> origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
> origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```

2. 원래 **original** 저장소 추가

```shell
git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```

3. 아래처럼 다시 **git remote -v** 로 저장소 확인

```shell
$ git remote -v
> origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
> origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
> upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
> upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
```

<br>



## Fork한 저장소를 원래 original 저장소로 동기화 시키기



1.  **git fetch upstream** 으로 원래 저장소를 fetch

```shell
$ git fetch upstream
> remote: Counting objects: 75, done.
> remote: Compressing objects: 100% (53/53), done.
> remote: Total 62 (delta 27), reused 44 (delta 9)
> Unpacking objects: 100% (62/62), done.
> From https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY
>  * [new branch]      master     -> upstream/master
```

2. 다시 master로 돌아가기

```shell
$ git checkout master
> Switched to branch 'master'
```

3. **master**에서 **upstream** merge하기

```shell
$ git merge upstream/master
> Updating a422352..5fdff0f
> Fast-forward
>  README                    |    9 -------
>  README.md                 |    7 ++++++
>  2 files changed, 7 insertions(+), 9 deletions(-)
>  delete mode 100644 README
>  create mode 100644 README.md
```





<br>

**References**

-   https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/syncing-a-fork 
-   https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/configuring-a-remote-for-a-fork 