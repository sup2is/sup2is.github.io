---
layout: post
title: "Github 1일 1커밋 자동화 스크립트 만들기"
tags: [GitHub, Bash Shell]
date: 2020-08-07
comments: true
---



<br>

# OverView

현재 나는 **1일 1커밋**을 실천하고 있다. 본격적으로 시작한건 2019년 4월 10일인데 이때 목표로 잡았던게 **365일동안 매일 하루 한개의 알고리즘 문제를 푸는것**이었다. 문제의 난이도는 중요하지 않았다. 그냥 매일매일 문제를 풀고 코드를 작성하는 습관을 들이는 것이였는데 이것이 현재 나에게 매우 큰 원동력이 되었고 앞으로도 개발자 인생을 마치는 순간까지 쭉 이어갈 예정이다.

물론 **매일같이 컴퓨터에 앉아서 한 줄의 코드를 작성하리라는 보장을 할 수 없는게 사람 일이다.**여행을 1주일 이상 가거나 본의아니게 깜빡했다거나 라는 이유 등등으로 **커밋을 할 수 없는 경우가 있다.** 나같은 경우에는 급하게 노트북이나 태블릿, 심지어 휴대폰으로 알고리즘 문제를 풀고 커밋하는 날도 꽤 있었다.(매우 간단한 문제만 .. )

![20200807_114145](https://user-images.githubusercontent.com/30790184/89610209-ebccc580-d8b4-11ea-9eee-ea1889292d6b.png)

> ※중간중간 깜빡하거나 놀러가서 커밋 못한날이 보임

**"이렇게 까지 해야돼?"** 라고 생각할 수도 있지만 1일 1커밋을 실천해본 사람으로써 꽤 집착할만한 주제다. 급하게 노트북이나 태블릿, 휴대폰으로 알고리즘 문제를 꾸역꾸역 풀지 않으려면 **미리미리 문제를 풀어두는 방법 밖에는 없었다.** 하지만 문제를 풀어놔도 어쨌든 매일 24시간 안에는 어떤 기기를 사용하던 간에 커밋을 해야했기 때문에 이것 역시 효율성이 그렇게 좋지는 않았다.

이때부터 생각했던게 **'아... 빨리 자동화 스크립트를 만들어야지'** 였는데 이걸 이제야 만든다. shell 프로그래밍에 익숙하지 않아서 문법이나 관례가 어색할 수 있지만 정리하는 목적으로 포스팅을 작성한다!

# 시작하기 전에

일단 시나리오는 다음과같다.

**1. 알고리즘 문제를 푼다. 풀고 난 뒤에 파일의 첫번째 줄에 커밋 메시지를 작성한다.**

```java
//find-numbers-with-even-number-of-digits: https://leetcode.com/problems/find-numbers-with-even-number-of-digits/ Complete
package find_numbers_with_even_number_of_digits;

import java.util.Arrays;

public class Solution {
    public int findNumbers(int[] nums) {
        return (int) Arrays.stream(nums)
                .mapToObj(i -> String.valueOf(i))
                .filter(s -> s.length() % 2 == 0)
                .count();
    }
}
```

> 나는 알고리즘 문제의 커밋 메시지를 **{문제 제목}: {문제 원본 url} {상태값}** 으로 작성한다. 예를 들어 백준같은 경우는 '**회의실배정: https://www.acmicpc.net/problem/1931 Complete**'식으로 작성한다

**2. 물리적인 개인 서버가 없기때문에 GCP f1-micro(vCPU 1개, 0.6GB 메모리) 서버를 사용한다 이 서버는 평생무료로 사용할 수 있다.**

**3. 풀어둔 문제를 GCP f1-micro 서버 내부 git이 관리하는 디렉토리에 넣어둔다.**

**4. git status 명령어로 가장 첫번째 파일을 가져온 뒤 파일의 첫번째 행을 읽어서 커밋메시지를 가져온다.**

**5. 가져온 커밋메시지를 기반으로 github 서버에 푸시한다.**

위에 적어놓은 시나리오대로 스크립트를 작성해보자.

# 디렉토리 분석하기

이제 스크립트를 작성해보도록 하자. 현재 나의 **algorithm** 디렉토리 구조는 다음과 같다.

```
[root@github-auto-push algorithm]# ll
total 24
drwxr-xr-x. 308 root root 12288 Aug  7 00:15 beakjoon
drwxr-xr-x.  21 root root  4096 Aug  7 00:10 leetcode
drwxr-xr-x.  93 root root  4096 Aug  6 23:48 programmers
[root@github-auto-push algorithm]# pwd
/root/algorithm
```

**alogirithm** 디렉토리 내부에는 각각 **문제를 제공하는 사이트별로** 폴더를 나눠놨다.

```
[root@github-auto-push beakjoon]# pwd
/root/algorithm/beakjoon
[root@github-auto-push beakjoon]# ll
total 0
drwxr-xr-x. 2 root root 23 Aug  6 23:48 beakjoon_1001
drwxr-xr-x. 2 root root 23 Aug  6 23:48 beakjoon_10026
drwxr-xr-x. 2 root root 23 Aug  6 23:48 beakjoon_1003
drwxr-xr-x. 2 root root 23 Aug  6 23:48 beakjoon_10039
drwxr-xr-x. 2 root root 23 Aug  6 23:48 beakjoon_1008

...

[root@github-auto-push leetcode]# pwd
/root/algorithm/leetcode
[root@github-auto-push leetcode]# ll
total 0
drwxr-xr-x. 2 root root 27 Aug  6 23:48 add-two-numbers
drwxr-xr-x. 2 root root 27 Aug  6 23:48 binary_tree_level_order_traversal
drwxr-xr-x. 2 root root 27 Aug  6 23:48 binary_tree_level_order_traversal_ii
drwxr-xr-x. 2 root root 27 Aug  6 23:48 can_make_arithmetic_progression_from_sequ

```

문제를 제공하는 사이트별로 **각각 문제별 고유 id 를 기반**으로 폴더를 나눠놨다.

```
[root@github-auto-push leetcode]# cd add-two-numbers/
[root@github-auto-push add-two-numbers]# ll
total 4
-rw-r--r--. 1 root root 2045 Aug  6 23:48 Solution.java
```

**폴더 내부에는 알고리즘을 풀이한 .java 파일이 각각 폴더별로 한개씩만 존재**한다. 이 파일이 우리가 github에 푸시할 파일이다.

# 자동화 스크립트 만들기

위에서 디렉토리 분석이 끝났다면 아주 간단한 자동화 스크립트를 작성해보자.

```shell
      1 #!/bin/bash
      2 
      3 echo '##### auto push start #####'
      4 
      5 baseDir=/root/algorithm
      6 cd ${baseDir}
      7 filePath=`git status -s | head -n 1`
      8 filePath=${filePath:3}
      9 
     10 if [ -z "$filePath" ]; then
     11   echo '##### file not found #####'
     12   exit
     13 fi
     14 
     15 cd $filePath
     16 filename=`ls -1 | head -1`
     17 commitMsg=`cat $filename | head -1`
     18 
     19 git add ./
     20 git status
     21 git commit -m "${commitMsg:2}"
     22 git push
     23 
     24 echo '##### auto push end #####'
```

매우 간단한 스크립트이지만 나같은 사람을 위해 정리해보자면.

- 5 ~ 8 : git이 관리하는 /root/algorithm 디렉토리에서 `git status -s | head -n 1` 를 사용해서 untracked file을 한개 가져온다. 보통의 `git status` 명령어는 `git status --long` 명령어로 동작한다. `-s` 또는 `--short` 옵션을 준다면 untracked file목록만 출력한다.
- 10 ~ 13 : ` -z "$filePath" `를 사용해서 가져온 filePath가 공백인지 검사한다. 공백이라면 아무것도 커밋할게 없으니 그대로 종료시킨다.
- 15 ~ 17 : filePath로 이동해서 `ls -1 | head -1` 명령어로 커밋할 파일명을 가져온 뒤 `cat $filename | head -1` 명령어를 사용해서 파일 최상단에 커밋 메시지를 읽어온다.
- 19 ~ 22 : 현재 디렉토리를 `git add` 명령어로 staging 시키고 `git commit + git push` 를 사용해서 푸시한다.
- 이렇게 작성한 뒤 `git config --global user.name`, `git config --global user.email` 를 지정하고 초기 username과 password만 잘 입력해준다.
- `crontab`같은 스케쥴러에 해당 쉘을 등록시킨다. 나같은경우는 매일 밤 11시에 쉘파일을 실행하도록 설정했다. (GCP 리전을 US로 잡은 경우 미국시간이니 주의할것 !)
- `${filePath:3}` 또는 `${commitMsg:2}`를 사용해서 필요 없는 문자를 잘라냈다.
- 별다른 설정이 없다면 초기 커밋시에는 username, password를 요구할 수 있으니 주의하자. git credential 에 관련된 정보는 [[https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Credential-%EC%A0%80%EC%9E%A5%EC%86%8C](https://git-scm.com/book/ko/v2/Git-도구-Credential-저장소)](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Credential-%EC%A0%80%EC%9E%A5%EC%86%8C) 에서 확인할 수 있다.

1일 1커밋을 실천하고 있지만 약 1주일동안 컴퓨터의 사용이 어려울것이라고 예상된다면 미리 1주일치 알고리즘을 풀어놓자. 풀어놓은 뒤에 개인서버에 올려두기만 한다면 자동화된 스크립트가 알아서 github에 푸시해줄 것이다.

# 마무리

만약 github 전체 repo에서 가장 최근 커밋을 가져오는 api가 있다면 날짜를 비교해서 밤 11시까지 커밋이 없을때 쉘 파일을 실행시키도록 하고 싶었으나 아쉽게도 [https://developer.github.com/](https://developer.github.com/) 에서 지원해주지 않는것 같다.. 비슷한 방법을 찾아낸다면 업데이트해야겠다.



<hr>
포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!
<br>
- https://www.baeldung.com/orika-mapping)

