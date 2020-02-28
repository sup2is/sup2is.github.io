---
layout: default
title: Git Untracked file들 지우기
parent: Git
date: 2020-02-28
comments: true

---



# Git Untracked file들 지우기

- 지워질 목록 확인하기 (디렉토리 포함은 -d )

```
git clean -n or git clean -n -d
```

<br>

- 실제로 지우기 (디렉토리 포함은 -d )

```
git clean -f or git clean -f -d
```

<br>

- ignore 파일까지 싹 다 지우기

```
git clean -f -x
```





**References**

-  https://koukia.ca/how-to-remove-local-untracked-files-from-the-current-git-branch-571c6ce9b6b1 