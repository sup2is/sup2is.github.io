---
layout: post
title: "Thread dump와 Heap dump"
tags: [Thread Dump, Heap Dump]
date: 2020-10-29
comments: true
---



<br>

# OverView

이번시간에는 Thread dump와 Heap dump에 대해서 알아보고 실제로 덤프를 떠서 분석해보도록 하겠다.

# Dump?

dump라는 단어의 정의는 아래와 같다.

<br>

> - **타동사**
>   - **1.** [내버리다](https://ko.wiktionary.org/wiki/내버리다), 뒤집어서 [비우다](https://ko.wiktionary.org/wiki/비우다), 짐을 [부리다](https://ko.wiktionary.org/wiki/부리다).
>   - **2. (컴퓨터) 시스템에 저장된 정보를 복사하다.**
>
> - **명사**
>   - **1.** [더미](https://ko.wiktionary.org/wiki/더미).
>   - **2.** [쓰레기장](https://ko.wiktionary.org/w/index.php?title=쓰레기장&action=edit&redlink=1), [집적소](https://ko.wiktionary.org/w/index.php?title=집적소&action=edit&redlink=1).
>   - **3. (컴퓨터) 시스템에 저장된 정보를 복사하여 뽑은 것. 또는 그와 같은 일.**
>   - 파생어: [dump car](https://ko.wiktionary.org/w/index.php?title=dump_car&action=edit&redlink=1), [dumping](https://ko.wiktionary.org/wiki/dumping), [dump truck](https://ko.wiktionary.org/w/index.php?title=dump_truck&action=edit&redlink=1)
>
> [https://ko.wiktionary.org/wiki/dump](https://ko.wiktionary.org/wiki/dump)

<br>

여기에서 사용되는 뜻은 **시스템에 저장된 정보를 복사하여 뽑은 것. 또는 그와 같은 일**이다. 일종의 snapshot 개념인데  주로 분석 최적화에 많이 쓰인다. DB dump, Thread dump 등이 있다.

# Thread Dump

## Thread Dump가 무엇이고 필요한 이유가 무엇일까?

- 프로세스에 속한 모든 thread들의 상태를 기록한것.
- 여기서 말하는 thread들 중 일부는 JVM 내부 thread 이고 일부는 실행중인 애플리케이션의 thread임
- **Thread dump는 발생된 문제들을 진단, 분석하고 jvm 성능 최적화하는데 필요한 정보를 보여줌**
- 예를들어서 Thread dump는 자동으로 데드락을 표시함

<br>

> - **데드락의 사전적 의미**
>   - [운영체제](https://namu.wiki/w/운영체제) 혹은 [소프트웨어](https://namu.wiki/w/소프트웨어)의 잘못된 [자원](https://namu.wiki/w/자원) 관리로 인하여 둘 이상의 프로세스(심하면 **운영체제 자체**도 포함해서)가 함께 멈추어 버리는 현상을 말한다. 
>   - [https://namu.wiki/w/Deadlock](https://namu.wiki/w/Deadlock)
>
> - 우리나라말로 교착상태라고 함
> - 데드락은 두 개 이상의 Thread에서 작업을 완료하기 위해 상대의 작업이끝나기를 기다림

<br>

Thread dump는 위에서 설명한 것처럼 주로 데드락이나 성능문제가 있을때 사용한다. 아래는 데드락을 발생시키는 간단한 java 코드이다.

```java
package me.sup2is.exam;

public class TestDeadlockExample1 {

    public static void main(String[] args) {
        final String resource1 = "ratan jaiswal";
        final String resource2 = "vimal jaiswal";
        // t1 tries to lock resource1 then resource2
        Thread t1 = new Thread(() -> {
            synchronized (resource1) {
                System.out.println("Thread 1: locked resource 1");

                try {
                    Thread.sleep(100);
                } catch (Exception e) {
                }

                synchronized (resource2) {
                    System.out.println("Thread 1: locked resource 2");
                }
            }
        });

        // t2 tries to lock resource2 then resource1
        Thread t2 = new Thread(() -> {
            synchronized (resource2) {
                System.out.println("Thread 2: locked resource 2");

                try {
                    Thread.sleep(100);
                } catch (Exception e) {
                }

                synchronized (resource1) {
                    System.out.println("Thread 2: locked resource 1");
                }
            }
        });


        t1.start();
        t2.start();
    }
}
```





## Thread Dump 떠보기

**jstack**이라는 도구를 사용해서 thread dump를 시도해보자. 일단 별도로 분석할만한게 없다면 위에서 언급한 데드락을 발생시키는 코드를 직접 실행해서 데드락을 발생시켜보자.

이후에 jstack이라는 도구를 사용할껀데 이 도구의 위치는 **%JAVA_HOME%/bin** 디렉토리에 위치하고있다.

jstack을 사용하기 위해서는 프로세스의 pid를 알아야하는데 pid를 간단하게 알고싶다면 **%JAVA_HOME%/bin**에서 다음 명령어를 입력해보자.

```
jps -mv
```

![20201029_085447](https://user-images.githubusercontent.com/30790184/97509834-abb64400-19c6-11eb-973b-1b28e9c48508.png)

> 사진이 생각보다 작아서 잘 안보이는데 맨앞에 **2968**이 pid다.

이런식으로 pid를 알아냈다면 다음 명령어로 thread dump를 분석해보자.

```shell
jstack -l <PID>


# 파일로 출력하기
jstack -l <PID> > dump.txt
```

![20201029_090241](https://user-images.githubusercontent.com/30790184/97509832-ab1dad80-19c6-11eb-9b56-29004d50fc30.png)

로그 하단부분에 이런식으로 데드락발생 부분을 표시해주는것을 확인할 수 있다.

## Thread Dump분석 툴

Thread Dump를 단순히 로그로만 분석하기에는 불편할 수 있다. 따라서 별도의 분석 툴을 이용할 수 있는데 여기에서는 간단하게 [https://fastthread.io/](https://fastthread.io/)를 사용한 분석을 할 예정이다. 조금 더 다양한 분석 툴에 대한 정보는 [https://blog.leocat.kr/notes/2016/03/07/java-dump-analyze-tool](https://blog.leocat.kr/notes/2016/03/07/java-dump-analyze-tool) 여기에서 확인할 수 있다.

![20201029_090832](https://user-images.githubusercontent.com/30790184/97509831-aa851700-19c6-11eb-8277-695e48c7571e.png)



![20201029_090852](https://user-images.githubusercontent.com/30790184/97509836-ac4eda80-19c6-11eb-9ae0-31f40868db73.png)



![20201029_090901](https://user-images.githubusercontent.com/30790184/97509826-a953ea00-19c6-11eb-8759-dd4628c5fb3b.png)



![20201029_090925](https://user-images.githubusercontent.com/30790184/97509835-abb64400-19c6-11eb-90f0-d74a98afc590.png)


dump.txt를 알아서 분석해주어 thread group, dead lock 정보 등등 다양한 방법으로 분석을 도와주고 있다.

# Heap Dump

- 특정 시점에 JVM heap 영역에 있는 모든 개체의 스냅샷
- 일반적으로 알고 있는 JVM 구조에서 heap영역을 생각하면 됨
- **GC가 힙에서 불필요한 객체를 제거하지 못하는경우 Java VisualVM을 사용하여 해당 객체에 대한 정보를 얻어낼 수 있음**

![20201029_092922](https://user-images.githubusercontent.com/30790184/97510983-52034900-19c9-11eb-83dd-d2133a7dcaee.png)



## Heap Dump 떠보기

heap dump는 **jmap**이라는 도구를 사용할 예정이다. 이 도구 역시 pid를 알아야하기 때문에 위에서 언급한 jps로 간단하게 확인할 수 있다.

jps는 **%JAVA_HOME%/bin**에서 찾을 수 있다.

```
jps -mv
```

![20201029_094506](https://user-images.githubusercontent.com/30790184/97515502-7f092900-19d4-11eb-9736-eafe92abcf42.png)

이제 jmap으로 heap dump를 얻어보자.

```shell
# 실시간 분석하기
jmap -heap <PID>

# 파일로 출력하기
jmap -dump:format=b,file=<FILE_OUTPUT> <PID>
```

> 참고로 heap dump의 기본확장자는 **.hprof** 이다.

![20201029_094450](https://user-images.githubusercontent.com/30790184/97515503-7fa1bf80-19d4-11eb-9e61-e88bd1c7424f.png)

나는 heap dump 분석을 위해 아래와 같이 100만개의 UUID를 갖는 LinkedList를 생성했다.

```java
package me.sup2is.exam;

import java.util.LinkedList;
import java.util.List;
import java.util.UUID;

public class TestHeapDump {
    public static void main(String[] args) throws InterruptedException {

        List<String> list = new LinkedList<>();

        for (int i = 0; i < 1000000; i++) {
            list.add(UUID.randomUUID().toString());
        }

        System.out.println("initial");

        Thread.sleep(30000);

    }
}

```



## Java Visual VM

Java VisualVM은 heapdump 분석, 프로세스와 메모리 모니터링, Thread 모니터링 등등 여러 기능을 지원한다.

관련된 정보와 다운로드는 [https://visualvm.github.io/index.html](https://visualvm.github.io/index.html)에서 확인할 수 있다.

VisualVM을 실행시킨뒤 heap dump 파일을 분석해보자.

![20201029_104126](https://user-images.githubusercontent.com/30790184/97515104-a27fa400-19d3-11eb-89a4-ae2c2b4e38a3.png)

1.File > Load에서 이전에 생성한 .hprof 파일을 불러들인다.

![20201029_104150](https://user-images.githubusercontent.com/30790184/97515101-a14e7700-19d3-11eb-9ca6-4b007502d236.png)

2.여러개의 정보가 나오는데 **Classes by Number of Instances**에서 **view all**을 클릭하면 heap 내부 전체 Object들을 확인할 수 있다.

![20201029_104242](https://user-images.githubusercontent.com/30790184/97515103-a27fa400-19d3-11eb-821c-68d4973db4ef.png)

3.heap 내부에서 위에서 생성한 100만개의 UUID를 갖는 LinkedList를 확인할 수 있다.

# 마무리

Thread Dump와 Heap Dump는 데드락, 성능이슈 및 분석에 굉장히 탁월한 도구이다. 이 포스팅에서는 매우 간단하게 알아봤지만 VisualVM 과 같은 도구들이 실제 실무 애플리케이션에 성능개선에 굉장히 많은 도움이 될 것 같다.



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

- [https://d2.naver.com/helloworld/10963](https://d2.naver.com/helloworld/10963)\
- [https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/using_threaddumps.html](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/using_threaddumps.html)
- [https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=18&dbnum=183741&mode=detail&type=techreport](https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=18&dbnum=183741&mode=detail&type=techreport)
- [https://www.javatpoint.com/deadlock-in-java](https://www.javatpoint.com/deadlock-in-java)
- [https://intellij-support.jetbrains.com/hc/en-us/articles/206544899-Getting-a-thread-dump-when-IDE-hangs-and-doesn-t-respond](https://intellij-support.jetbrains.com/hc/en-us/articles/206544899-Getting-a-thread-dump-when-IDE-hangs-and-doesn-t-respond)
- [https://docs.oracle.com/javase/8/docs/technotes/guides/visualvm/heapdump.html](https://docs.oracle.com/javase/8/docs/technotes/guides/visualvm/heapdump.html)
- [https://ktdsoss.tistory.com/439](https://ktdsoss.tistory.com/439)
