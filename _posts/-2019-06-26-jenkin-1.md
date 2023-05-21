---
layout: post
title: "Jenkins & Docker로 CI/CD 학습하기 #1"
tags: [CI, Jenkins , Jenkins in Docker]
comments: true
nav_order: 5
date: 2019-06-26
parent: Jenkins
---



이번시간에는 CI 툴인 Jenkins에 대해 알아보고 간단한 예제를 통해 직접 파이프라인을 구성하여 배포 자동화를 해보는 시간을 가져보겠습니다.



<br>

# CI(Continuous Integration) / CD (Continuous  Delivery)

Jenkins는 대표적인 CI/CD 툴이다. 여기서 말하는 CI는 Continuous Integration(지속적인 통합)을 의미하고 CD는 Continuous  Delivery(지속적인 배포) 로 해석이된다.  [링크](https://www.redhat.com/ko/topics/devops/what-is-ci-cd) 에서 아주 잘 설명해주는 것 같다.



> 약자 CI/CD는 몇 가지의 다른 의미를 가지고 있습니다. CI/CD의 “CI”는 개발자를 위한 자동화 프로세스인 지속적인 통합(Continuous Integration)을 의미합니다. CI를 성공적으로 구현할 경우 애플리케이션에 대한 새로운 코드 변경 사항이 정기적으로 빌드 및 테스트되어 공유 리포지토리에 병합되므로 여러명의 개발자가 동시에 애플리케이션 개발과 관련된 코드 작업을 할 경우 서로 충돌할 수 있는 문제를 해결할 수 있습니다.
>
> CI/CD의 “CD”는 지속적인 서비스 제공 (Continuous Delivery) 및/또는 지속적인 배포 (Continuous Deployment)를 의미하며 이 두 용어는 상호 교환적으로 사용됩니다. 두 가지 의미 모두 파이프라인의 추가 단계에 대한 자동화를 뜻하지만 때로는 얼마나 많은 자동화가 이루어지고 있는지를 설명하기 위해 별도로 사용되기도 합니다.
>
> 지속적인 *제공*은 개발자들이 애플리케이션에 적용한 변경 사항이 버그 테스트를 거쳐 리포지토리(예: [GitHub](https://redhatofficial.github.io/#!/main) 또는 컨테이너 레지스트리)에 자동으로 업로드되는 것을 뜻하며, 운영팀이 이 리포지토리에서 애플리케이션을 실시간 프로덕션 환경으로 배포할 수 있습니다. 이는 개발팀과 비즈니스팀 간의 가시성과 커뮤니케이션 부족 문제를 해결해 줍니다. 지속적인 제공은 최소한의 노력으로 새로운 코드를 배포하는 것을 목표로 합니다.
>
> 지속적인 *배포*(또 다른 의미의 “CD”: Continuous Deployment)는 개발자의 변경 사항을 리포지토리에서 고객이 사용 가능한 프로덕션 환경까지 자동으로 릴리스하는 것을 의미합니다. 이는 애플리케이션 제공 속도를 저해하는 수동 프로세스로 인한 운영팀의 프로세스 과부하 문제를 해결합니다. 지속적인 배포는 파이프라인의 다음 단계를 자동화함으로써 지속적인 제공이 가진 장점을 활용합니다.
>
> CI/CD가 지속적 통합 및 지속적 제공의 구축 사례만을 지칭하는 것일 수도 있고, 지속적 통합, 지속적 제공, 지속적 배포라는 3 가지 구축 사례 모두를 의미하는 것일 수도 있습니다. 좀 더 복잡하게 설명하면 "지속적 제공"은 때로 지속적 배포의 과정까지 포함하는 방식으로 사용되기도 합니다.
>
> 결과적으로 CI/CD는 파이프라인으로 표현되는 실제 프로세스를 의미하고, 애플리케이션 개발에 지속적인 자동화 및 지속적인 모니터링을 추가하는 것을 의미합니다. 이 용어는 사례별로 CI/CD 파이프라인에 구현된 자동화 수준 정도에 따라 그 의미가 달라집니다. 대부분의 기업들은 CI를 먼저 추가한 다음 [클라우드 네이티브 애플리케이션](https://www.redhat.com/ko/topics/cloud-native-apps)의 일부로서 배포 및 개발 자동화를 구현해 나갑니다.



<br>

# Jenkins 따라하기

jenkins 공홈에서 documents -> tutorials 탭의 [Build a Java app with Maven](https://jenkins.io/doc/tutorials/build-a-java-app-with-maven/)을 기반으로 하나하나씩 차례대로 jenkins를 활용하는 방법을 따라해보겠다. 필자는 window로 진행했다 ~~후회중~~

<br>

먼저 간단한 준비물이다.

> - A macOS, Linux or Windows machine with:
>   - 256 MB of RAM, although more than 512MB is recommended.
>   - 10 GB of drive space for Jenkins and your Docker images and containers.
> - The following software installed:
>   - [Docker](https://www.docker.com/) - Read more about installing Docker in the [Installing Docker](https://jenkins.io/doc/book/installing/#installing-docker) section of the [Installing Jenkins](https://jenkins.io/doc/book/installing/) page.
>     **Note:** If you use Linux, this tutorial assumes that you are not running Docker commands as the root user, but instead with a single user account that also has access to the other tools used throughout this tutorial.
>   - [Git](https://git-scm.com/downloads) and optionally [GitHub Desktop](https://desktop.github.com/).

<br>

참고로 이 글을 쓰는 시점(20190625)의 나는 docker 대해서 지식이 전무한 상태이다 ... jenkins를 위해서 docker를 방금 설치했다. 그러므로 docker에 대한 자세한 설명은 생략한다.. ~~구글에 많음~~

<br>

docker가 설치되었다는 가정하에 진행한다. 이 튜토리얼은 [`jenkinsci/blueocean`](https://hub.docker.com/r/jenkinsci/blueocean/)  라는 docker container에서 jenkins를 올려서 진행한다. docker에대한 내용은 [Docker](https://jenkins.io/doc/book/installing#docker) ,  [Downloading and running Jenkins in Docker](https://jenkins.io/doc/book/installing#downloading-and-running-jenkins-in-docker) 에서 확인할 수 있다.

<br>

간략하게 예제에 필요한 **jenkinsci/blueocean** image는 아래의 명령어로 docker hub를 통해 설치할 수 있다.

```
docker pull jenkinsci/blueocean
```

이제 jenkins를 실행시켜보자.

<br>

> #### On Windows
>
> 1. Open up a command prompt window.
>
> 2. Run the `jenkinsci/blueocean` image as a container in Docker using the following [`docker run`](https://docs.docker.com/engine/reference/commandline/run/)command (bearing in mind that this command automatically downloads the image if this hasn’t been done):
>
>    ```
>    docker run ^
>      --rm ^
>      -u root ^
>      -p 8080:8080 ^
>      -v jenkins-data:/var/jenkins_home ^
>      -v /var/run/docker.sock:/var/run/docker.sock ^
>      -v "%HOMEPATH%":/home ^
>      jenkinsci/blueocean
>    ```

<br>

위 명령어를 통해 docker container에서 jenkins를 실행한다.

는 fail...

자꾸 이상한 에러가 나서 원래 git bash에서 작업하다가 powershell로 작업하니까 또다른 에러 ... 조금 찾아보니까 위 명령어에서 **%HOMEPATH%** 는 여러분들의 local git repository 경로를 올려주면된다. 이렇게 올리면 docker container에서 D:/github 경로를 /home으로 액세스할 수 있다.

```
docker run --rm -u root -p 8080:8080 -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v D:/github:/home jenkinsci/blueocean
```



<br>

이제 jenkins를 실행시켜보자

<br>

![9](https://user-images.githubusercontent.com/30790184/60075024-50839a00-975f-11e9-94b6-74ef73c4aa73.png)



최초 실행시 빨간 박스안에 password를 통해 최초인증을 받는다. 복사!!

![1](https://user-images.githubusercontent.com/30790184/60075084-76a93a00-975f-11e9-8446-790fd4f90c5c.png)

<br>

![2](https://user-images.githubusercontent.com/30790184/60075104-84f75600-975f-11e9-9a3c-70a20b79494d.png)

<br>

플러그인 설치인데 잘 모르니까 왼쪽꺼로 설치했다.

![3](https://user-images.githubusercontent.com/30790184/60075168-ab1cf600-975f-11e9-9724-190f46458b6f.png)

<br>

![4](https://user-images.githubusercontent.com/30790184/60075169-abb58c80-975f-11e9-98f3-65f451b2335b.png)

<br>

알아서 계정입력해준다.

![5](https://user-images.githubusercontent.com/30790184/60075170-abb58c80-975f-11e9-9c23-95f62ec4e3ce.png)

<br>

실제 access할 url을 지정하는거같다. 나는 로컬에서만 실행할꺼니까 127.0.0.1로 지정해준다.

![6](https://user-images.githubusercontent.com/30790184/60075166-ab1cf600-975f-11e9-8250-0f477b1e1853.png)

<br> 설치 끝!!!



![7](https://user-images.githubusercontent.com/30790184/60075270-e0294880-975f-11e9-837a-60c47c147e9b.png)

설치가 완료되면  다음과 같은 화면이 나온다.

<br>

이제 jenkins 공홈에서 제공해주는 [`simple-java-maven-app`](https://github.com/jenkins-docs/simple-java-maven-app)을 통하여 간단한 실습을 진행한다. 이 github repository를 여러분들의 github에 fork받고 local에 clone한다. local에 cloen할때 위에서 언급한 **%HOMEPATH%** 에 맞게 clone받아야한다. 나같은경우는 **D:\github\simple-java-maven-app** 가 이 repository의 경로이다. docker container에서는 **/home/simple-java-maven-app**  으로 접근 가능하다.

<br>

위 github repository를 이용하여 실제로 jenkins pipeline을 생성해보자



- 화면 왼쪽에 'New Item' 또는 '새로운 Item'을 클릭하여 job을 생성한다.

- item의 이름을 지정해주고 Pipeline을 클릭 후 OK버튼을 누른다

![8](https://user-images.githubusercontent.com/30790184/60076519-b7568280-9762-11e9-8928-a2d3594cdeac.png)

- 하단에 Pipeline 탭에서 **Definition을 Pipeline script from SCM**을 지정하고 SCM을 **GIT**으로 지정한다

- 이제 아까 설정해놓은  **/home/simple-java-maven-app** 를 repository url에 입력하자

이제  **'Jenkinsfile'** 이라는 pipeline에 대한 script를 만들껀데 사실 우리가 clone한 repository의 **/jenkins** 디렉토리에 이미 작성이 되어있다. 따라하다가 모르면 참고해도 좋을것 같다.

![10](https://user-images.githubusercontent.com/30790184/60077153-5039cd80-9764-11e9-93ef-0d5f5ff77bb6.png)





- 이제  **'Jenkinsfile'** 이라는 pipeline에 대한 script를 만들껀데 사실 우리가 clone한 repository의 **/jenkins** 디렉토리에 이미 작성이 되어있다. 따라하다가 모르면 참고해도 좋을것 같다.

<br>

- local repository의 root 디렉토리에서 **'Jenkinsfile'** 이라는 이름의 파일을 생성하고 다음의 스크립트를 복붙한다.

> ```groovy
> pipeline {
>     agent {
>         docker {
>             image 'maven:3-alpine' 
>             args '-v /root/.m2:/root/.m2' 
>         }
>     }
>     stages {
>         stage('Build') { 
>             steps {
>                 sh 'mvn -B -DskipTests clean package' 
>             }
>         }
>     }
> }
> ```



- 이후에 다음 명령어를 입력하여 **Jenkinsfile** 을 git stage 상태로 올린다.

```
git add .
git commit -m "Add initial Jenkinsfile"
```



- jenkins 기본페이지에서 왼쪽메뉴에보면 **'Open Blue Ocean'**을 클릭한다.

- **'This job has not been run'** 박스의 **'Run'** 버튼을 클릭한다



- 그럼 이제 놀라운일들이 벌어지는데 jenkins는 **Jenkinsfile** 파일을 통해서 docker에 maven:3-alpine가 있는지 판단하고 없으면 **docker pull maven:3-alpine** 명령어를 통해서 docker hub에서 image를 받는다. (조금 걸림)



![12](https://user-images.githubusercontent.com/30790184/60160831-2c908900-9831-11e9-9dad-dc747c36d7d2.png)



- 이후 **stage**에 정의된 **'Build'**  부분을 실행한다. 이때도 역시 **maven**은 build에 필요한 artifacts들을 받는데 위치는 jenkins의 local maven repository에 받는다. (in the Docker host’s filesystem)

- **Blue Ocean interface**가 초록색으로 변했으면 jenkins가 빌드에 성공했음을 의미한다.

- 다시 **Jenkinsfile** 파일을 열고 **'Build Stage'**  밑에 아래의 스크립트를 추가한다


```groovy
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
```

- 이후 **Jenkinsfile** 을 아래의 명령어로 stage 상태에 올린다.


```
git stage .
git commit -m "Add 'Test' stage"
```



- 다시 **Blue Ocean interface** 탭에서 **Run** 을 동작시킨다. 이때 docker hub에서 maven image나 build에 필요한 artifact들은 다시 받지 않아도 된다. 이전에 **Build Stage**를 통해서 다운받은 상태이기 때문이다.



- 화면에 **'Test'** 라는 하나의 Stage가 추가된 걸 확인할 수 있다.




![13](https://user-images.githubusercontent.com/30790184/60160832-2c908900-9831-11e9-8d03-4c9f663f339b.png)



- 마지막으로 **Jenkinsfile** 을 열고 다음 스크립트를 추가한다.

```groovy
        stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
```

- 이후 **Jenkinsfile** 을 아래의 명령어로 stage 상태에 올린다.

```
git stage .
git commit -m "Add 'Deliver' stage"
```

- 똑같이 **Blue Ocean interface** 탭에서 **Run** 을 동작시킨다.  그럼 화면에 **'Deliver'** 라는 Stage가 추가된 걸 확인할 수 있다.

<br>



![14](https://user-images.githubusercontent.com/30790184/60160834-2d291f80-9831-11e9-8488-f58d0f864cbc.png)



<br>

이정도까지가 Jenkins 공홈에있는 Jenkins & docker 를 이용한 예제가 되겠다.

<br>



사실 공홈에서 는 이 예제가 20~40 분정도 소요된다고 했는데 나는 하루를 날렸다 ^^ docker에 대해서 잘 모른다가 최종 변명! Jenkins pipeline script에 대한 상세설명은 생략했다. 나중에 다시 정리해보는걸로 ...

<br>

포스팅은 여기까지 하겠습니다. 



<br>

퍼가실때는 링크와 출처를 반드시 명시해주세요. 감사합니다.

<br>



출처 : https://jenkins.io/

