---
title: "Ⅸ. Travis CI 배포 자동화"
excerpt: "코드가 푸시되면 자동으로 배포"

categories:
    - SpringBoot & AWS
tags:
    - SpringBoot
    - Amazon Web Services
    - RDS
    - IntelliJ IDEA 
    - Gradle
    - JPA
    - Git
use_math: true;
last_modified_at: 2020-04-20

# published : false
--- 
> 개발 언어 : __Java 8(JDK 1.8)__  
> 개발 환경 : __Gradle 4.8 ~ Gradle 4.10.2__  
> 참조 :  
> [기억보단 기록을](https://jojoldu.tistory.com/)  
> [스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)](https://jojoldu.tistory.com/463)  
> [예제 코드 내려받기](http://bit.ly/fr-springboot)
  
8장에서 프로젝트를 EC2에 배포했는데 스크립트를 개발자가 직접 실행함으로써 수동으로 Test & Build 했다. 이번 장 부터는 VCS 시스템(Git, SVN 등)에 PUSH 를 하면 자동으로 Test & Build & Deploy가 진행되어 안정적인 배포 파일을 만드는 과정 CI(Continuous Integration - 지속적인 통합)할 것이며, 이 빌드 결과를 자동으로 운영 서버에 무중단 배포까지 진행되는 과정 CD(Continuous Deployment - 지속적인 배포) 할 것이다.  
  
## Travis CI 웹 서비스 설정  
[https://travis-ci.org/](https://travis-ci.org/)에서 깃허브 계정으로 로그인을 한 뒤, 왼쪽 위에 [계정명 -> Setting]를 클릭한다.  
[![그림 1-01](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img01.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img01.png)  
  
설정 페이지 아래쪽을 보면 깃허브 저장소 검색창이 있다. 여기서 저장소 이름을 입력해서 찾은 다음, 오른쪽의 상태바를 활성화 시킨다.  
[![그림 1-02](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img02.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img02.png)  
  
활성화한 저장소를 클릭하면 저장소 빌드 히스토리 페이지로 이동한다.  
[![그림 1-03](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img03.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img03.png)  
  
## 프로젝트 설정  
__Travis CI 설정__  
프로젝트의 `build.gradle`과 같은 위치에서 `.travis.yml`을 생성한 후 코드 추가  
[![그림 1-04](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img04.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img04.png)  
  
```yml  
language: java
jdk:
  - openjdk8

branches: //1.
  only:
    - master

# Travis CI 서버의 Home
cache: //2.
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

//3.
script: "./gradlew clean build"

# C 실행 완료시 메일로 알람
notifications: //4.
  email:
    recipients:
      - 본인 메일 주소
```  
1. branches  
+ Travis CI를 어느 브랜치가 푸시될 때 수행할지 지정  
+ 현재 옵션은 오직 master 브랜치에 push 될 때만 수행  
2. cache  
+ 그레이들을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐시하여, 같은 의존성은 다음 배포 때부터 다시 받지 않도록 설정  
3. script  
+ master 브랜치에 푸시되었을 때 수행하는 명령어  
+ 여기서는 프로젝트 내부에 둔 gradlew 을 통해 clean & build 를 수행  
4. notifications  
+ Travis CI 실행 완료 시 자동으로 알람이 가도록 설정  
  
master 브랜치에 커밋과 푸시를 하고, Travis CI 저장소 페이지를 확인  
[![그림 1-05](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img05.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img05.png)  
  
자동 배포화를 하는 도중  
__Travis /home/travis/.travis/functions: line 351: ./gradlew: Permission denied__  
[![그림 1-06](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img06.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img06.png)  
에러가 발생하며 빌드를 할 수 없다는 메세지가 출력이 되었다면  
local 터미널에서 프로젝트 폴더로 이동한 뒤  
[![그림 1-07](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img07.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img07.png)  
```  
git update-index --chmod=+x gradlew  
```  
를 입력한 후에 커밋과 푸쉬를 해준다.  
  
## Travis CI와 AWS S3 연동하기  
S3란 AWS에서 제공하는 __일종의 파일 서버__ 이다. 이미지 파일을 비롯한 정적 파일들을 관리하거나 배포 파일들을 관리하는 등의 기능을 지원한다.  
[![그림 1-08](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img08.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img08.png)  
  
Travis CI는 CI 툴이다. Test & Build 까지는 자동화 시켜주었지만, 빌드된 파일을 원하는 서버로 전달까지 해주지는 않는다. Travis CI로 Build된 파일을 EC2 인스턴스로 전달하기 위해 AWS CodeDeploy를 사용한다. 그전에 S3 연동이 먼저 필요한 이유는 Jar 파일을 전달하기 위해서이다.  
CodeDeploy는 저장 기능이 없다. 그래서 Travis CI가 빌드한 결과물을 받아서 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간이 필요하다. 보통은 이럴 때 AWS S3를 이용한다.  
  
__AWS Key 발급__  
AWS 서비스에 외부 서비스가 접근할 수 없다. 접근 가능한 권한을 가진 Key를 생성해서 사용하여야 한다.  
AWS에서는 이러한 인증과 관련된 기능을 제공하는 서비스로 IAM(Identity and Access Management)이 있다.
IAM은 AWS에서 제공하는 접근 방식과 권한을 관리한다.
AWS 웹 콘솔에서 IAM을 검색하여 이동한다.  
IAM 페이지 왼쪽 사이드바에서 [사용자 -> 사용자 추가]버튼 클릭  
[![그림 1-09](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img09.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img09.png)  
[![그림 1-10](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img10.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img10.png)  
  
생성할 사용자의 이름과 엑세스 유형을 선택  
엑세스 유형은 프로그래밍 방식 엑세스이다.  
[![그림 1-11](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img11.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img11.png)  
  
권한 설정 방식은 [기존 정책 직접 연결]을 선택  
정책 검색 화면에서 s3full로 검색하여 체크하고 다음 권한으로 codedeployfull을 검색하여 체크  
[![그림 1-12](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img12.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img12.png)  
[![그림 1-13](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img13.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img13.png)  
  
태그 등록  
[![그림 1-14](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img14.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img14.png)  
  
권한 최종 확인  
[![그림 1-15](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img15.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img15.png)  
  
최종 생성 완료되면 엑세스 키와 비밀 엑세스 키가 생성된다. 이 값들이 Travis CI에서 사용될 키이다.  
창을 닫으면 시크릿 엑세스 키를 다시 확인하지 못하니 좌측의 .csv 다운로드 버튼을 클릭해 csv로 키를 저장하거나 본인이 다른 방법으로 복사나 저장해두어야 한다.
[![그림 1-16](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img16.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img16.png)  
  
## Travis CI에 키 등록  
Travis CI의 설정 화면으로 이동  
[![그림 1-17](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img17.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img17.png)  
  
설정 화면의 아래쪽에 Environment Variables 항목에 AWS_ACCESS_KEY, AWS_SECRET_KEY를 변수로 해서 IAM 사용자에서 발급받은 키 값들을 등록한다.  
[![그림 1-18](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img18.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img18.png)  
  
여기에 등록된 값들은 `.travis.yml` 에서 `$AWS_ACCESS_KEY`, `$AWS_SECRET_KEY`란 이름으로 사용할 수 있다.  
  
__S3 버킷 생성__  
AWS의 S3(Simple Storage Service)서비스는 일종의 파일 서버이다. 파일들을 저장하고 접근 권한을 관리, 검색 등을 지원하는 파일 서버의 역할을 한다.  
Travis CI에서 생성된 Build 파일을 저장하도록 구성하고 S3에 저장된 Build 파일은 이후 AWS의 CodeDeploy에서 배포할 파일로 가져가도록 구성한다.  
AWS서비스에서 S3를 검색하여 이동하고 버킷을 생성한다.  
  
[![그림 1-19](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img19.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img19.png)  
[![그림 1-20](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img20.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img20.png)  
[![그림 1-21](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img21.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img21.png)  
  
S3가 생성되었으니 S3로 배포 파일을 전달해 보자  
  
__.travis.yml 추가__  
Travis CI에서 빌드하여 만든 Jar 파일을 S3에 올릴 수 있도록 `.travis.yml`에 코드 추가  
```yml  
language: java
jdk:
  - openjdk8

branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build"

before_deploy: #//1.
  - zip -r freehyun-springboot-webservice * #//2.
  - mkdir -p deploy #//3.
  - mv freehyun-springboot-webservice.zip deploy/freehyun-springboot-webservice.zip #//4.
deploy: #//5.
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings 에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings 에 설정된 값
    bucket: freehyun2-springboot-build # S3 버킷
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private 으로
    local_dir: deploy #//6.
    wait-until-deployed: true

# C 실행 완료시 메일로 알람
notifications:
  email:
    recipients:
      - shounghyun@gmail.com
```  
1. before_deploy  
+ deploy 명령어가 실행되기 전에 수행  
+ CodeDeploy 는 Jar 파일을 인식하지 못하므로 Jar+ 기타 설정 파일들을 모아 압축(zip)  
2. zip -r freehyun-springboot-webservice  
+ 현재 위치의 모든 파일을 freehyun-springboot-webservice 이름으로 압축(zip)한다.  
+ 명령어의 마지막 위치는 본인의 프로젝트 이름이어야 한다.  
3. mkdir -p deploy  
+ deploy라는 디렉토리를 Travis CI가 실행 중인 위치에서 생성  
4. mv freehyun-springboot-webservice.zip deploy/freehyun-springboot-webservice.zip  
+ freehyun-springboot-webservice.zip 파일을 deploy/freehyun-springboot-webservice.zip 으로 이동  
5. deploy  
+ S3로 파일 업로드 혹은 CodeDeploy로 배포 등 외부 서비스와 연동될 행위들을 선언  
6. local_dir: deploy  
+ before_deploy 에서 생성한 디렉토리  
+ 해당 위치의 파일들만 S3로 전송  
  
설정이 다 되었다면 깃허브로 푸시한다.  
Travis CI에서 자동으로 빌드가 진행되는 것을 확인하고, 모든 빌드가 성공하는지 확인한다.  
다음 로그가 나온다면 Travis CI의 빌드가 성공한 것이다.  
[![그림 1-22](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img22.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img22.png)  
  
S3 버킷으로 가보면 업로드가 성공한 것을 확인할 수 있다.  
[![그림 1-23](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img23.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img23.png)  
  
Travis CI를 통해 자동으로 파일이 올려진 것을 확인할 수 있다.  
Travis CI와 S3 연동이 완료되었다. 이제 CodeDeploy로 배포까지 완료해 보자  
  
## Travis CI와 AWS S3, CodeDeploy 연동하기  
AWS의 배포 시스템인 CodeDeploy를 이용하기 전에 배포 대상인 EC2가 CodeDeploy를 연동 받을 수 있게 IAM 역할을 하나 생성하자  
__EC2에 IAM 역할 추가__
S3와 마찬가지로 IAM을 검색하고, 이번에는 [역할] 탭을 클릭해서 이동한다. [역할 -> 역할 만들기] 버튼을 차례로 클릭한다.  
[![그림 1-24](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img24.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img24.png)  
  
+ 역할  
    + AWS 서비스에만 할당할 수 있는 권한  
    + EC2, CodeDeploy, SQS 등  
+ 사용자  
    + AWS 서비스 외에 사용할 수 있는 권한  
    + 로컬PC, IDC 서버 등  
  
지금 만들 권한은 EC2에서 사용할 것이기 때문에 사용자가 아닌 역할로 처리.  
서비스 선택에서는 [AWS 서비스 -> EC2]를 차례로 선택  
  
[![그림 1-25](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img25.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img25.png)  
  
정책에서는 EC2RoleForA를 검색하여 AmazonEC2RoleforAWSCodeDeploy를 선택  
[![그림 1-26](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img26.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img26.png)  
  
태그 등록  
[![그림 1-27](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img27.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img27.png)  
  
이름 등록 및 최종 확인  
[![그림 1-28](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img28.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img28.png)  
  
역할을 EC2 서비스에 등록  
EC2 인스턴스 목록으로 이동한 뒤, 본인의 인스턴스를 선택한 후 작업 버튼을 눌러 [인스턴스 설정 -> IAM 역할 연결/바꾸기]를 차례로 선택  
[![그림 1-29](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img29.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img29.png)  
  
생성한 역할을 선택한다.  
[![그림 1-30](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img30.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img30.png)  
  
역할 선택이 완료되면 해당 EC2 인스턴스를 재부팅 한다.  
[![그림 1-31](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img31.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img31.png)  
  
재부팅이 완료되었으면 CodeDeploy의 요청을 받을 수 있게 에이전트를 설치한다.  
__CodeDeploy 에이전트 설치__  
EC2 접속 CodeDeploy 에이전트 설치  
[![그림 1-32](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img32.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img32.png)  
  
install 파일에 실행 권한이 없으니 실행 권한을 추가  
```  
chmod +x ./install  
```  
install 파일로 설치를 진행  
```  
sudo ./install auto  
```  
설치가 끝났다면 Agent가 정상적으로 실행되고 있는지 상태검사  
```  
sudo service codedeploy-agent status  
```
running 메시지가 출력되면 정상  
```  
The AWS CodeDeploy agent is running as PID xxxx   
```  
만약 설치 중에 `/usr/bin/env:ruby:No such file or directory` 에러가 발생한다면 루비라는 언어가 설치 안 된 상태라서 그렇다. 이럴 경우 yum install 로 루비를 설치하면 된다.  
```  
sudo yum install ruby  
```  
  
## CodeDeploy를 위한 권한 생성  
CodeDeploy에서 EC2에 접근하려면 권한이 필요하다. AWS의 서비스이니 IAM 역할을 생성한다. 서비스는 [AWS 서비스 -> CodeDeploy]를 차례로 선택한다.  
[![그림 1-24](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img24.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img24.png)  
[![그림 1-33](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img33.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img33.png)  
  
CodeDeploy는 권한이 하나뿐이라서 선택 없이 바로 다음으로 넘어간다.  
[![그림 1-34](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img34.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img34.png)  
  
CodeDeploy 태그 등록  
[![그림 1-35](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img35.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img35.png)  
  
CodeDeploy 생성완료  
[![그림 1-36](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img36.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img36.png)  
  
__CodeDeploy 생성__  
CodeDeploy는 AWS의 배포 삼형제 중 하나이다.  
1. Code Commit  
+ 깃허브와 같은 코드 저장소의 역할을 한다.  
+ 프라이빗 기능을 지원한다는 강점이 있지만, 현재 깃허브에서 무료로 프라이빗 지원을 하고 있어서 거의 사용되지 않는다.  
2. Code Build  
+ Travis CI와 마찬가지로 빌드용 서비스  
+ 멀티 모듈을 배포해야 하는 경우 사용해 볼만하지만, 규모가 있는 서비스에서는 대부분 젠킨스/팀시티 등을 이용하니 이것 역시 사용할 일이 거의 없다.  
3. CodeDeploy  
+ AWS 배포 서비스  
+ 앞에서 언급한 다른 서비스는 대체재가 있고, CodeDeploy는 대체재가 없다.
+ 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포 등 많은 기능을 지원한다.
  
이 중에서 현재 진행 중인 프로젝트에서는 Code Commit의 역할은 깃허브가, Code Build의 역할은 Travis CI가 하고 있다. 그래서 우리가 추가로 사용할 서비스는 CodeDeploy이다.  
CodeDeploy 서비스로 이동해서 화면 중앙에 있는 [애플리케이션 생성] 버튼을 클릭한다.  
[![그림 1-37](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img37.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img37.png)  
  
생성할 CodeDeploy의 이름과 컴퓨팅 플랫폼을 선택한다. 컴퓨팅 플랫폼에선 [EC2/온프레미스]를 선택하면 된다.  
[![그림 1-38](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img38.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img38.png)  
  
생성이 완료되면 배포 그룹을 생성하라는 메시지를 볼 수 있다. 화면 중앙의 [배포 그룹 생성] 버튼을 클릭한다.  
[![그림 1-39](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img39.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img39.png)  
  
배포 그룹 이름과 서비스 역할을 등록한다. 서비스 역할은 좀 전에 생성한 CodeDeploy용 IAM 역할을 선택하면 된다.  
[![그림 1-40](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img40.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img40.png)  
  
배포 유형에서는 현재 위치를 선택한다. 만약 본인이 배포할 서비스가 2대 이상이라면 블루/그린을 선택하면 된다.  
[![그림 1-41](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img41.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img41.png)  
  
CodeDeploy 환경 선택  
[![그림 1-42](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img42.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img42.png)  
  
배포 구성을 선택하고 로드밸런싱은 체크 해제한다.  
[![그림 1-43](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img43.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img43.png)  
  
CodeDeployDefault.AllAtOnce는 한 번에 다 배포하는 것을 의미한다. 설정이 끝났으므로 배포 그룹 생성한다.  
  
__Travis CI, S3, CodeDeploy 연동__  
먼저 S3에서 넘겨줄 zip 파일을 저장할 디렉토리를 하나 생성한다. EC2 서버에 접속해서 디렉토리를 생성한다.  
```  
mkdir ~/app/step2 && mkdir ~/app/step2/zip  
```  
Travis CI의 Build가 끝나면 S3에 zip 파일이 전송되고, 이 zip 파일은 `/home/ec2-user/app/step2/zip`로 복사되어 압축을 풀 예정이다.  
Travis CI의 설정은 `.travis.yml`로 진행했다.  
AWS CodeDeploy의 설정은 `appspec.yml`로 진행한다.  
[![그림 1-44](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img44.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img44.png)  
  
`appspec.yml` 코드 생성  
```yml  
version: 0.0 #//1.
os: linux
files:
  - source: / #//2.
    destination: /home/ec2-user/app/step2/zip/ #//3.
    overwrite: yes #//4.
```  
1. version: 0.0  
+ CodeDeploy 버전  
+ 프로젝트 버전이 아니므로 0.0 외에 다른 버전을 사용하면 오류가 발생  
2. source  
+ CodeDeploy 에서 전달해 준 파일 중 destination 으로 이동시킬 대상을 지정  
+ 루트 경로(/)를 지정하면 전체 파일을 이야기함  
3. destination  
+ source 에서 지정된 파일을 받을 위치  
+ 이후 Jar 를 실행하는 등은 destination 에서 옮긴 파일들로 진행  
4. overwrite  
+ 기존에 파일들이 있으면 덮어쓸지를 결정  
+ 현재 yes 라고 했으니 파일들을 덮어쓰게 됨  
  
.travis.yml에도 CodeDeploy 내용을 추가  
deploy 항목에 다음 코드를 추가  
```yml  
deploy:
    ...

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY # Travis repo settings 에 설정된 값
    secret_access_key: $AWS_SECRET_KEY # Travis repo settings 에 설정된 값
    bucket: freehyun2-springboot-build # S3 버킷
    key: freehyun-springboot-webservice.zip # 빌드 파일을 압축해서 전달
    bundle_type: zip # 압축 확장자
    application: freehyun-springboot-webservice # 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
    deployment_group: freehyun-springboot-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포 그룹
    region: ap-northeast-2
    wait-until-deployed: true
```  
  
s3 옵션과 유사하다. 다름 부분은 CodeDeploy의 애플리케이션 이름과 배포 그룹명을 지정하는 것이다.  
전체 코드는 다음과 같다.  
```yml  
language: java
jdk:
  - openjdk8

branches:
  only:
    - master

# Travis CI 서버의 Home
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build"

before_deploy:
  - zip -r freehyun-springboot-webservice *
  - mkdir -p deploy
  - mv freehyun-springboot-webservice.zip deploy/freehyun-springboot-webservice.zip
deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY
    secret_access_key: $AWS_SECRET_KEY
    bucket: freehyun2-springboot-build
    region: ap-northeast-2
    skip_cleanup: true
    acl: private
    local_dir: deploy
    wait-until-deployed: true

  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY
    secret_access_key: $AWS_SECRET_KEY
    bucket: freehyun2-springboot-build
    key: freehyun-springboot-webservice.zip
    bundle_type: zip
    application: freehyun-springboot-webservice
    deployment_group: freehyun-springboot-webservice-group
    region: ap-northeast-2
    wait-until-deployed: true

# C 실행 완료시 메일로 알람
notifications:
  email:
    recipients:
      - 본인 메일 주소
```  
  
모든 내용을 작성했다면 프로젝트를 커밋하고 푸시한다. 깃허브로 푸시가 되면 Travis CI가 자동으로 시작된다.  
Travis CI가 끝나면 CodeDeploy 화면 아래에서 배포가 수행되는 것을 확인할 수 있다.  
[![그림 1-45](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img45.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img45.png)  
[![그림 1-46](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img46.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img46.png)  
  
배포가 끝났다면 파일들이 잘 도착했는지 확인한다.  
[![그림 1-47](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img47.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img47.png)  
  
## 배포 자동화 구성  
앞의 과정으로 __Travis CI, S3, CodeDeploy 연동__ 까지 구현되었다. 이제 이것을 기반으로 실제로 __jar를 배포하여 실행__ 까지 해보자  
  
__deploy.sh 파일 추가__  
step2 환경에서 실행될 `deploy.sh`를 생성  
`scripts` 디렉토리를 생성해서 스크립트를 코드 추가  
[![그림 1-48](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img48.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img48.png)  
  
```sh  
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=freehyun-springboot-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl freehyun-springboot-webservice | grep jar | awk '{print $1}') //1.

echo "> 현재 구동중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME //2.

echo "> $JAR_NAME 실행"

nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=real \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &    //3.
```  
1. CURRENT_PID  
+ 현재 수행 중인 스프링 부트 애플리케이션의 프로세스 ID를 찾는다.  
+ 실행 중이면 종료하기 위해서이다.  
+ 스프링 부트 애플리케이션 이름(freehyun-springboot-webservice)으로 된 다른 프로그램들이 있을 수 있어 freehyun-springboot-webservice로 된 jar(pgrep -fl freehyun-springboot-webservice | grep jar)프로세스를 찾은 뒤 ID를 찾는다(| awk '{print $1}').
2. chmod +x $JAR_NAME  
+ Jar 파일은 실행 권한이 없는 상태이다.  
+ nohup으로 실행할 수 있게 실행 권한을 부여한다.  
3. $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &  
+ nohup 실행 시 CodeDeploy는 무한 대기 한다.  
+ 이 이슈를 해결하기 위해 nohup.out 파일을 표준 입출력용으로 별도로 사용한다.  
+ 이렇게 하지 않으면 nohup.out 파일이 생기지 않고, CodeDeploy 로그에 표준 입출력이 출력된다.  
+ nohup이 끝나기 전에 CodeDeploy도 끝나지 않으니 꼭 이렇게 해야만 한다.  
  
step1에서 작성된 `deploy.sh`와 크게 다르지 않다. 우선 git pull을 통해 직접 빌드했던 부분을 제거했다. 그리고 Jar를 실행하는 단계에서 몇가지 코드가 추가되었다.  
  
__.travis.yml 수정__  
현재는 프로젝트의 모든 파일을 zip 파일로 만드는데, 실제로 필요한 파일들은 jar, appspec.yml, 배포를 위한 스크립트 들이다. 이 외 나머지는 배포에 필요하지 않으니 포함하지 않는다.  
`.travis.yml` 파일의 before_deploy 수정  
(`.travis.yml` 파일은 Travis CI에서만 필요하지 CodeDeploy에서 필요하지는 않는다.)  
```yml  
before_deploy:
  - mkdir -p before-deploy # zip 에 포함시킬 파일들을 담을 디렉토리 생성
  - cp scripts/*.sh before-deploy/
  - cp appspec.yml before-deploy/
  - cp build/libs/*.jar before-deploy/
  - cd before-deploy && zip -r before-deploy * # before-deploy로 이동후 전체 압축
  - cd ../ && mkdir -p deploy # 상위 디렉토리로 이동후 deploy 디렉토리 생성
  - mv before-deploy/before-deploy.zip deploy/freehyun-springboot-webservice.zip # deploy로 zip파일 이동
```  
  
__appspec.yml 파일 수정__  
CodeDeploy의 명령을 담당할 `appspec.yml` 파일을 수정  
appspec.yml 파일에 코드 추가  
location, timeout, runas의 들여쓰기를 주의해야 한다. 들여쓰기가 잘못될 경우 배포가 실패한다.  
```yml  
permissions: #//1.
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks: #//2.
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ec2-user
```  
1. permissions  
+ CodeDeploy 에서 EC2 서버로 넘겨준 파일들을 모두 ec2-user 권한을 갖도록 한다.  
2. hooks  
+ CodeDeploy 배포 단계에서 실행할 명령어를 지정  
+ ApplicationStart 라는 단계에서 `deploy.sh` 를 ec2-user 권한으로 실행  
+ timeout:60 으로 스크립트 실행 60초 이상 수행되면 실패가 된다.(무한정 기다릴 수 없으니 시간 제한을 둬야만 한다.)  
  
모든 설정이 완료되었으니 깃허브로 커밋과 푸시를 한다. Travis CI에서 다음과 같이 성공 메시지를 확인하고 CodeDeploy에서도 배포가 성공한 것을 확인한다.  
[![그림 1-49](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img49.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img49.png)  
[![그림 1-50](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img50.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img50.png)  
  
웹 브라우저에서 EC2 도메인을 입력하여 확인  
[![그림 1-51](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img51.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img51.png)  
  
__실제 배포 과정 체험__  
`build.gradle`에서 프로젝트 버전을 변경한다.  
```  
version '1.0.1-SNAPSHOT'  
```  
변경된 내용을 알 수 있게 `src/main/resources/templates/index.mustache` 내용에 다음과 같이 Ver.2 텍스트를 추가한다.  
```  
    ...
    <h1>스프링 부트로 시작하는 웹서비스 Ver.2</h1>
    ...
```  
  
텍스트를 추가하였다면 깃허브로 커밋과 푸시를 한다. 그럼 다음과 같이 변경된 코드가 배포된 것을 확인할 수 있다.  
[![그림 1-53](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img53.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img53.png)  
신규 버전이 정상적으로 잘 배포되었다.  
  
## CodeDeploy 로그 확인  
CodeDeploy와 같이 AWS가 지원하는 서비스에서는 오류가 발생했을때 로그 찾는 방법을 모르면 오류를 해결하기가 어렵다. 그래서 배포가 실패하면 어느 로그를 봐야 하지 간단하게 보자  
CodeDeploy에 관한 대부분 내용은 `/opt/codedeploy-agent/deployment-root`에 있다. 해당 디렉토리로 이동(`/opt/codedeploy-agent/deployment-root`)한 뒤 목록을 ()확인해 보면 (ll) 다음과 같은 내용을 확인 할 수 있다.  
[![그림 1-52](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img52.png)](/assets/Web/AWS/2020-04-24-SpringBoot-AWS-09-img52.png)  
  
1. 최상단의 영문과 대시( - )가 있는 디렉토리명은 CodeDeploy ID이다.  
+ 사용자마다 고유한 ID가 생성되어 각자 다른 ID가 발급되니 본인의 서버에는 다른 코드로 되어있다.  
+ 해당 디렉토리로 들어가 보면 배포한 단위별로 배포 파일들이 있다.  
+ 본인의 배포 파일이 정상적으로 왔는지 확인해 볼 수 있다.  
2. /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log  
+ CodeDeploy 로그파일이다.  
+ CodeDeploy로 이루어지는 배포 내용 중 표준 입/출력 내용은 모두 여기에 담겨 있다.  
+ 작성한 echo 내용도 모두 표기된다.  
  
테스트, 빌드, 배포까지 전부 자동화되었다. 작업이 끝난 내용을 __Master 브랜치에 푸시만 하면 자동으로 EC2에 배포__ 가 된다.  
한가지 문제점이 있는데 배포하는동안 스프링 부트 프로젝트는 종료 상태가 되어 서비스를 이용할 수 없다는 점이다. 그래서 서비스 중단 없는 배포방법을 다음장에서 진행해 볼것이다.  
  