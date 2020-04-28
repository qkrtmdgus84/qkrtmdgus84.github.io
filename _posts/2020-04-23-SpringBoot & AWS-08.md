---
title: "Ⅷ. EC2 서버에 배포"
excerpt: "EC2서버에 프로젝트 배포하기"

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
  
## EC2에 프로젝트 Clone 받기  
깃허브에서 코드를 받아올 수 있게 EC2에 깃을 설치한다.  
[![그림 1-01](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img01.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img01.png)  
  
설치가 완료되면 설치 상태를 확인  
```  
git --version
```  
git clone으로 프로젝트를 저장할 디렉토리를 생성  
```  
mkdir ~/app && mkdir ~/app/step1
```  
생성된 디렉토리 이동  
```  
cd ~/app/step1
```  
[![그림 1-02](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img02.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img02.png)  
  
본인의 깃허브 주소를 복사한다.  
[![그림 1-03](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img03.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img03.png)  
  
복사한 주소를 통해 git clone을 진행한다.  
```  
git clone 복사한 주소
```  
[![그림 1-04](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img04.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img04.png)  
  
clone된 프로젝트로 이동해서 파일들이 잘 복사되었는지 확인한다.  
```  
cd 프로젝트명
ll
```  
[![그림 1-05](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img05.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img05.png)  
  
코드들이 잘 수행되는지 테스트로 검증한다.  
```  
./gradlew test
```  
[![그림 1-06](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img06.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img06.png)  
  
[![그림 1-07](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img07.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img07.png)  
  
__-bash: ./gralew: Permission denied__ gradlew 실행 권한이 없다면  
  
```  
chmod +x ./gradlew
```  
[![그림 1-08](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img08.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img08.png)  
  
테스트가 실패해서 수정하고 깃허브에 푸시를 했다면 프로젝트 폴더 안에서 __git pull__ 한다.  
  
## 배포 스크립트 만들기  
`~/app/step1/` 에 `deploy.sh` 파일 생성  
```  
vim ~/app/step1/deploy.sh
```  
`deploy.sh` 코드 추가  
```sh  
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1   1.
PROJECT_NAME=freehyun-springboot-webservice

cd $REPOSITORY/$PROJECT_NAME/  2.

echo "> Git Pull"  3.

git pull

echo "> 프로젝트Build 시작"

./gradlew build  4.

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/  5.

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)  6.

echo "  현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then  7.
	echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
	echo "> kill -15 $CURRENT_PID"
	kill -15 $CURRENT_PID
	sleep 5
fi
echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)  8.

echo "> JAR Name: $JAR_NAME"

nohup java -jar  $REPOSITORY/$JAR_NAME 2>&1 &  9.

```  
1. REPOSITORY=/home/ec2-user/app/step1  
+ 프로젝트 디렉토리 주소는 스크립트 내에서 자주 사용하는 값이기 때문에 이를 변수로 저장  
+ PROJECT_NAME=freehyun-springboot-webservice도 동일하게 변수로 저장  
+ 쉘에서는 타입 없이 선언하여 저장  
+ 쉘에서는 $변수명으로 변수를 사용  
2. cd $REPOSITORY/$PROJECT_NAME/  
+ git clone 받았던 디렉토리로 이동  
+ /home/ec2-user/app/step1/freehyun-springboot-webservice 주소로 이동  
3. git pull  
+ 디렉토리 이동 후, master 브랜치의 최신 내용을 받는다.  
4. ./gradlew build  
+ 프로젝트 내부의 gradlew로 build를 수행  
5. cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/  
+ build의 결과물인 jar 파일을 복사해 jar 파일을 모아둔 위치로 복사  
6. CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)  
+ 기존에 수행 중이던 스프링 부트 애플리케이션을 종료  
+ pgrep은 process id만 추출하는 명령어  
+ -f 옵션은 프로세스 이름으로 찾는다.  
7. if ~ else ~ fi  
+ 현재 구동 중인 프로세스가 있는지 없는지를 판단해서 기능을 수행  
+ process id 값을 보고 프로세스가 있으면 해당 프로세스를 종료  
8. JAR_NAME=$(ls -tr $REPOSITORY/ | grep *.jar | tail -n 1)  
+ 새로 실행할 jar 파일명을 찾는다.  
+ 여러 jar 파일이 생기기 때문에 tail -n로 가장 나중의 jar 파일(최신 파일)을 변수에 저장  
9. nohup java -jar  $REPOSITORY/$JAR_NAME 2>&1 &  
+ 찾은 jar 파일명으로 해당 jar 파일을 nohup으로 실행  
+ 스프링 부트의 장점으로 특별히 외장 톰캣을 설치할 필요가 없다.  
+ 내장 톰캣을 사용해서 jar 파일만 있으면 바로 웹 애플리케이션 서버를 실행할 수 있다.  
일반적으로 자바를 실행할 때는 java -jar라는 명령어를 사용하지만, 이렇게 하면 사용자가 터미널 접속을 끊을 때 애플리케이션도 같이 종료된다.  
+ 애플리케이션 실행자가 터미널을 종료해도 애플리케이션은 계속 구동될 수 있도록 nohup 명령어를 사용한다.  
  
생성한 스크립트에 실행 권한을 추가한다.  
```sh  
chmod +x ./deploy.sh
```  
[![그림 1-09](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img09.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img09.png)  
  
스크립트 명령어 실행  
```  
./deploy.sh
```
  
[![그림 1-10](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img10.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img10.png)  
  
nohup.out 파일을 열어 로그를 보면 ClientRegistrationRepository를 찾을 수 없다는 에러가 발생하여 애플리케이션 실행이 실패했다는 것을 알 수 있다.  
(nohup.out은 실행되는 애플리케이션에서 출력되는 모든 내용을 갖고 있다.)  
[![그림 1-11](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img11.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img11.png)  
  
이유는 ClientRegistrationRepository를 생성하려면 clientId와 clientSecret가 필수이다. 로컬 PC에서 실행할 때는 application-oauth.properties가 있어 문제가 없었다.  
하지만 이 파일은 gitignore로 git에서 제외 대상이라 깃허브에는 올라가지 않았다.  
애플리케이션을 실행하기 위해 공개된 저장소에 ClientId와 ClientSecret을 올릴 수는 없으니 __서버에서 직접 이설정들을 가지고 있게__ 하겠다.  
  
## 외부 Security 파일 등록하기  
`step1`이 아닌 `app` 디렉토리에 `properties` 파일을 생성  
```  
vim /home/ec2-user/app/application-oauth.properties
```  
그리고 로컬에 있는 `application-oauth.properties` 파일 내용을 그대로 붙여넣기를 한다. 해당 파일을 저장하고 종료한다.(:wq). 그리고 방금 생성한 `application-oauth.properties`을 쓰도록 `deploy.sh` 파일을 수정한다.  
```sh  
...
nohup java -jar \
 -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
   $REPOSITORY/$JAR_NAME 2>&1 &
```  
1. Dspring.config.location  
+ 스프링 설정 파일 위치를 지정  
+ 기본 옵션들을 담고 있는 application.properties과 OAuth 설정들을 담고 있는 application-oauth.properties의 위치를 지정한다.  
+ classpath가 붙으면 jar 안에 있는 resources 디렉토리를 기준으로 경로가 생성된다.  
+ application-oauth.properties은 절대경로를 사용한다. 외부에 파일이 있기 때문이다.  
  
수정이 다 되었다면 다시 deploy.sh를 실행해 보고 vim nohup.out 파일을 열어 로그 확인 해보자  
[![그림 1-12](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img12.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img12.png)  
  
## 스프링 부트 프로젝트로 RDS 접근하기  
RDS는 MariaDB를 사용중이다. MariaDB에서 스프링부트 프로젝트를 실행하기 위해서는 다음 작업이 필요하다.  
+ 테이블 생성: H2에서 자동생성해주던 테이블들을 직접 쿼리를 이용해서 생성한다. 
+ 프로젝트 설정: 자바 프로젝트가 MariaDB에 접근하려면 데이터베이스 드라이버가 필요하다. MariaDB에서 사용 가능한 드라이버를 프로젝트에 추가  
+ EC2(리눅스 서버) 설정: 데이터베이스의 접속 정보는 중요하게 보호해야 할 정보이다. 공개되면 외부에서 데이터를 모두 가져갈 수 있기 때문이다. 프로젝트 안에 접속 정보를 갖고 있다면 깃허브와 같이 오픈된 공간에선 누구나 해킹할 위험이 있다. EC2 서버 내부에서 접속 정보를 관리하도록 설정한다.  
  
__RDS 테이블 생성__  
JPA가 사용될 엔티티 테이블과 스프링 세션이 사용될 테이블 생성한다.  
JPA가 사용할 테이블은 테스트 코드 수행 시 로그로 생성되는 쿼리를 사용하면 된다. 테스트 코드를 수행하면 로그가 발생하니 create table부터 복사하여 RDS에 반영한다.  
[![그림 1-13](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img13.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img13.png)  
  
[![그림 1-14](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img14.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img14.png)  
  
스프링 세션 테이블은 `schema-mysql.sql` 파일에서 확인할 수 있다.  
[Ctrl+Shift+N]으로 File 검색  
[![그림 1-15](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img15.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img15.png)  
  
세션 테이블을 복사하여 RDS에 반영한다.  
[![그림 1-16](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img16.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img16.png)  
  
[![그림 1-17](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img17.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img17.png)  
  
__프로젝트 설정__
MariaDB 드라이버를 `build.gradle`에 등록  
```  
compile("org.mariadb.jdbc:mariadb-java-client")
```
  
서버에서 구동될 환경을 구성하기 위해서 `src/main/resources/` 에 `application-real.properties` 파일을 추가한다.  
`application-real.properties` 로 파일을 만들면 profile=real 인 환경이 구성된다고 보면 된다. 실제 운영될 환경이기 때문에 보안/로그상 이슈가 될 만한 설정들을 모두 제거하며 RDS 환경 profile 설정이 추가된다.  
```properties  
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```  
  
깃 허브로 푸시한다.  
  
__EC2 설정__
OAuth와 마찬가지로 RDS 접속 정보도 보호해야 할 정보이니 EC2 서버에 직접 설정 파일을 둔다.  
`app` 디렉토리에 `application-real-db.properties` 파일을 생성한다.  
```  
vim ~/app/application-real-db.properties
```
코드 추가  
```properties  
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mariadb://rds주소:포트명(기본은3306)/database이름
spring.datasource.username=db계정
spring.datasource.password=db계정 비밀번호
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```  
1. spring.jpa.hibernate.ddl-auto=none  
+ JPA로 테이블이 자동 생성되는 옵션을 None로 지정  
+ RDS에는 실제 운영으로 사용될 테이블이니 절대 스프링 부트에서 새로 만들지 않도록 해야한다.  
+ 이 옵션을 하지 않으면 자칫 테이블이 모두 새로 생성될 수 있다.  
+ 주의해야 하는 옵션이다.  
  
deploy.sh가 real profile을 쓸 수 있도록 수정한다.  
```sh  
nohup java -jar \
 -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
  -Dspring.profiles.active=real \
   $REPOSITORY/$JAR_NAME 2>&1 &
```  
1. Dspring.profiles.active=real
+ application-real.properties를 활성화시킨다.  
+ application-real.properties의 spring.profiles.include=oauth,real-db 옵션 때문에 real-db 역시 함께 활성화 대상에 포함  
  
deploy.sh를 실행  
nohup.out파일을 열어 다음과 같이 로그가 보인다면 성공적으로 수행된 것이다.  
[![그림 1-18](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img18.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img18.png)  
  
curl 명령어로 html 코드가 정상적으로 보인다면 성공이다.  
```  
curl localhost:8080
```  
[![그림 1-19](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img19.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img19.png)  
  
## EC2에서 소셜 로그인 하기  
__AWS 보안 그룹 변경__  
EC2에 스프링 부트 프로젝트가 8080 포트로 배포되었으니, 8080포트가 보안 그룹에 열려 있는지 확인한다.  
[![그림 1-20](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img20.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img20.png)  
  
8080 열려 있다면 OK, 안 되어있다면 [인바운드 규칙 편집] 버튼을 눌러 추가한다.  
  
__AWS EC2 도메인으로 접속__  
왼쪽 사이드바의 [인스턴스] 메뉴를 클릭  
해당 EC2 인스턴스를 선택하면 상세 정보에서 __퍼블릭 DNS__ 를 확인할 수 있다.  
[![그림 1-21](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img21.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img21.png)  
  
[![그림 1-22](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img22.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img22.png)  
  
현재 상태에서는 해당 서비스에 EC2의 도메인을 등록하지 않았기 때문에 구글과 네이버 로그인이 작동하지 않는다.  
서비스에 등록해보자  
__구글에 EC2 주소 등록__  
구글 웹 콘솔[https://console.cloud.google.com/home/dashboard](https://console.cloud.google.com/home/dashboard)로 접속하여 본인의 프로젝트로 이동한 다음 [API 및 서비스 -> 사용자 인증 정보]로 이동한다.  
[![그림 1-23](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img23.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img23.png)  
  
본인이 등록한 서비스의 이름을 클릭  
[![그림 1-24](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img24.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img24.png)  
  
퍼블릭 DNS 주소에 :8080/login/oauth2/code/google 주소를 추가하여 승인된 리디렉션 URI 에 등록한다.  
[![그림 1-25](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img25.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img25.png)  
  
EC2 DNS 주소로 이동해서 다시 구글 로그인을 시도해 보면 구글 로그인이 정상적으로 수행되는 것을 확인할 수 있다.  
[![그림 1-26](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img26.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img26.png)  
  
__네이버에 EC2 주소 등록__
네이버 개발자 센터 [https://developers.naver.com/apps/#/myapps](https://developers.naver.com/apps/#/myapps)로 접속해서 본인의 프로젝트로 이동한다.  
[![그림 1-27](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img27.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img27.png)  
  
아래로 내려가 보면 __PC 웹__ 항목이 있는데 여기서 __서비스 URL과 Callback URL 2개를 수정한다.  
[![그림 1-28](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img28.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img28.png)  
  
1. 서비스 URL  
+ 로그인을 시도하는 서비스가 네이버에 등록된 서비스인지 판단하기 위한 항목이다.  
+ 8080 포트는 제외하고 실제 도메인 주소만 입력한다.  
+ 네이버에서 아직 지원되지 않아 하나만 등록 가능하다.  
+ 즉, EC2의 주소를 등록하면 localhost가 안 된다.  
+ 개발 단계에서는 등록하지 않는 것을 추천한다.  
+ localhost도 테스트하고 싶으면 네이버 서비스를 하나 더 생성해서 키를 발급받으면 된다.  
2. Callback URL  
+ 전체 주소를 등록한다(EC2 퍼블릭 DNS:8080/login/oauth2/code/naver).  
  
네이버 로그인  
[![그림 1-26](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img26.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img26.png)  
  
