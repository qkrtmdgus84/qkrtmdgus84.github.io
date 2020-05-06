---
title: "Ⅹ. 무중단 배포"
excerpt: "서비스 중단 없는 배포"

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
  
## NGINX 무중단 배포  
무중단 배포 방식에는 몇 가지가 있다.  
+ AWS에서 블루 그린(Blue-Green) 무중단 배포  
+ 도커를 이용한 웹서비스 무중단 배포  
  
우리가 진행할 방법은 엔진엑스(Nginx)를 이용한 무중단 배포 이다.  
__Nginx의 개요__  
엔진엑스는 lgor Sysoev라는 러시아 개발자 동시접속 처리에 특화된 웹 서버 프로그램이다. Apache보다 동작이 단순하고, 전달자 역할만 하기 때문에 동시 접속 처리에 특화되어 있다.  
  
__Nginx(웹서버)의 역할__  
1. 정적 파일을 처리하는 HTTP 서버로서의 역할  
+ 웹서버의 역할 HTML, CSS, Javascript, 이미지와 같은 정보를 웹 브라우저에 전송하는 역할(HTTP 프로토콜을 준수)  
2. 응용프로그램 서버에 요청을 보내는 리버스 프록시로서의 역할  
+ 클라이언트는 서버에 요청하면, 프록시 서버가 배후 서버(reverse sever)로부터 데이터를 가져오는 역할  
+ 프록시 서버가 Nginx, 리버스 서버가 응용프로그램 서버  
+ 여러대의 응용프로그램 서버에 배분 하는 역할을 한다.  
3. 비동기 Event-Drive 기반 구조  
  
__엔진엑스 무중단 배포 구성__  
하나의 EC2 혹은 리눅스 서버에 엔진엑스 1대와 스프링 부트 Jar를 2대를 사용  
+ 엔진엑스는 80(http), 443(https) 포트를 할당  
+ 스프링 부트1은 8081포트 실행  
+ 스프링 부트2은 8082포트 실행  
  
엔진엑스 무중단 배포 1  
[![그림 1-01](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img01.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img01.png)  
  
1. 사용자는 서비스 주소로 접속한다(80 혹은 443 포트)  
2. 엔진엑스는 사용자의 요청을 받아 현재 연결된 스프링 부트로 요청을 전달한다.(스프링 부트1 즉, 8081 포트로 요청을 전달한다고 가정)  
3. 스프링 부트2는 엔진엑스와 연결된 상태가 아니니 요청받지 못한다.  
  
엔진엑스 무중단 배포 2  
[![그림 1-02](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img02.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img02.png)  
  
1. 1.1 버전으로 신규 배포가 필요하면, 엔진엑스와 연결되지 않은 스프링 부트2(8082 포트)로 배포  
2. 배포하는 동안에도 서비스는 중단되지 않는다.  
  + Nginx는 스프링부트1을 바라보기 때문이다.  
3. 배포가 끝나고 정상적으로 스프링부트2가 구동중인지 확인한다.  
4. 스프링부트2가 정상 구동중이면 nginx reload를 통해 8081 대신에 8082를 바라보도록 한다.  
5. Nginx Reload는 1초 이내에 실행완료가 된다.  
  
엔진엑스 무중단 배포 3  
[![그림 1-03](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img03.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img03.png)  
  
1. 또다시 신규버전인 1.2 버전의 배포가 필요하면 이번엔 스프링부트1로 배포한다.  
  + 현재는 스프링부트2가 Nginx와 연결되있기 때문이다.  
2. 스프링부트1의 배포가 끝났다면 Nginx가 스프링부트1을 바라보도록 변경하고 nginx reload를 실행한다.  
3. 1.2 버전을 사용중인 스프링부트1로 Nginx가 요청을 전달한다.  
  
무중단 배포 전체 구조  
[![그림 1-04](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img04.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img04.png)  
  
## 엔진엑스 설치와 스프링 부트 연동하기  
__엔진엑스 설치__  
EC2에 접속해서 엔진엑스를 설치  
```  
sudo yum install nginx  
```  
설치가 완료되었다면 엔진엑스를 실행  
```  
sudo service nginx start  
```  
Nginx 잘 실행되었는지 확인  
```  
ps -ef | grep nginx  
```  
[![그림 1-05](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img05.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img05.png)  
  
__보안 그룹 추가__  
엔진엑스의 포트번호를 보안 그룹에 추가  
엔진엑스의 포트번호는 기본적으로 80이다. 해당 포트 번호가 보안 그룹에 없으니 [EC2 -> 보안 그룹 -> EC2 보안 그룹 선택 -> 인바운드 편집]으로 차례로 이동해서 변경  
[![그림 1-06](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img06.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img06.png)  
  
__리다이렉션 주소 추가__  
8080이 아닌 80포트로 주소가 변경되니 구글과 네이버 로그인에도 변경된 주소를 등록해야만 한다. 
구글 웹 콘솔[https://console.cloud.google.com/home/dashboard](https://console.cloud.google.com/home/dashboard)로 접속하여 본인의 프로젝트로 이동한 다음 [API 및 서비스 -> 사용자 인증 정보]로 이동한다.  
[![그림 1-23](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img23.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img23.png)  
  
본인이 등록한 서비스의 이름을 클릭  
[![그림 1-24](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img24.png)](/assets/Web/AWS/2020-04-23-SpringBoot-AWS-08-img24.png)  
  
기존에 등록된 리디렉션 주소에서 8080부분을 제거하여 추가 등록  
[![그림 1-07](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img07.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img07.png)  
  
__8080 포트를 제거__ 하고 접근  
포트번호 없이 도메인만 입력해서 브라우저에 접속  
[![그림 1-08](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img08.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img08.png)  
  
__엔진엑스와 스프링 부트 연동__  
엔진엑스가 현재 실행 중인 스프링 부트 프로젝트를 바라볼 수 있도록 프록시 설정을 하겠다.  
엔진엑스 설정 파일  
```  
sudo vim /etc/nginx/nginx.conf  
```  
설정 내용 중 server 아래의 location / 부분을 찾아서 엔진엑스 설정 추가  
[![그림 1-09](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img09.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img09.png)  
```  
proxy_pass http://localhost:8080; //1.
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; //2.
proxy_set_header Host $http_host;
```  
1. proxy_pass  
+ 엔진엑스로 요청이 오면 http://localhost:8080로 전달  
2. proxy_set_header XXX  
+ 실제 요청 데이터를 header의 각 항목에 할당  
+ proxy_set_header X-Real-IP $remote_addr 요청자의 ip를 저장  
  
수정이 끝났다면 :wq 명령어로 저장하고 종료해서, 엔진엑스를 재시작한다.  
```  
sudo service nginx restart  
```  
다시 브라우저로 접속해서 엔진엑스 시작 페이지가 보이면 화면을 새로고침 한다.  
[![그림 1-10](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img10.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img10.png)  
  
## 무중단 배포 스크립트 만들기  
무중단 배포 스크립트 작업 전에 API를 하나 추가하겠다. 이 API는 이후 배포 시에 8081을 쓸지, 8082를 쓸지 판단하는 기준이 된다.  
__profile API 추가__  
[![그림 1-11](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img11.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img11.png)  
`src/main/resources/com/freehyun/book/springboot/web/ProfileController`  
```java  
import lombok.RequiredArgsConstructor;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;


@RequiredArgsConstructor
@RestController
public class ProfileController {
    private final Environment env;

    @GetMapping("/profile")
    public String profile() {
        List<String> profiles = Arrays.asList(env.getActiveProfiles()); //1.
        List<String> realProfiles = Arrays.asList("real", "real1", "real2");
        String defaultProfile = profiles.isEmpty()? "default" : profiles.get(0);

        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);
    }
}
```  
1. env.getActiveProfiles()  
+ 현재 실행 중인 ActiveProfile을 모두 가져온다.  
+ 즉, real, oauth,real-db등이 활성화되어 있다면(active) 3개가 모두 담겨 있다.  
+ 여기서 real, real1, real2는 모두 배포에 사용될 profile이라 이 중 하나라도 있으면 그 값을 반환하도록 한다.  
  
이 코드가 작동하는지 테스트 코드를 작성해 보자. 해당 컨트롤러는 특별히 스프링 환경이 필요하지는 않는다. 그래서 @SpringBootTest없이 테스트 코드를 작성한다.  
[![그림 1-12](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img12.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img12.png)  
`src/test/resources/com/freehyun/book/springboot/web/ProfileControllerUnitTest`  
```java  
import org.junit.Test;
import org.springframework.mock.env.MockEnvironment;

import static org.assertj.core.api.Assertions.assertThat;

public class ProfileControllerUnitTest {

    @Test
    public void real_profile이_조회된다() {
        //given
        String expectedProfile = "real";
        MockEnvironment env = new MockEnvironment();
        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("oauth");
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void real_profile이_없으면_첫번째가_조회된다() {
        //given
        String expectedProfile = "oauth";
        MockEnvironment env = new MockEnvironment();

        env.addActiveProfile(expectedProfile);
        env.addActiveProfile("real-db");

        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }

    @Test
    public void active_profile이_없으면_default가_조회된다() {
        //given
        String expectedProfile = "default";
        MockEnvironment env = new MockEnvironment();
        ProfileController controller = new ProfileController(env);

        //when
        String profile = controller.profile();

        //then
        assertThat(profile).isEqualTo(expectedProfile);
    }
}
```  
  
Environment는 인터페이스라 가짜 구현체인 MockEnvironment(스프링에서 제공)를 사용해서 테스트하면 된다.  
[![그림 1-13](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img13.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img13.png)  
  
그리고 이 /profile이 인증 없이도 호출될 수 있게 SecurityConfig 클래스에 제외 코드를 추가한다.  
[![그림 1-14](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img14.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img14.png)  
  
```java  
.antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**", "/profile").permitAll()
```  
  
그리고 SecurityConfig 설정이 잘 되었는지도 테스트 코드로 검증한다. 이검증은 스프링 시큐리티 설정을 불러와야 하니 @SpringBootTest를 사용하는 테스트 클래스를 하나 더 추가한다.  
[![그림 1-15](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img15.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img15.png)  
`src/test/resources/com/freehyun/book/springboot/web/ProfileControllerTest`  
```java  
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ProfileControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void profile은_인증없이_호출된다() throws Exception {
        String expected = "default";

        ResponseEntity<String> response = restTemplate.getForEntity("/profile", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isEqualTo(expected);
    }
}
```  
  
[![그림 1-16](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img16.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img16.png)  
  
모든 테스트가 성공했다면 깃허브로 푸시하여 배포 한다. 배포가 끝나면 브라우저에서 /profile로 접속해서 profile이 잘 나오는지 확인한다.  
[![그림 1-17](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img17.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img17.png)  
[![그림 1-18](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img18.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img18.png)  
  
__real1, real2 profile 생성__  
현재 EC2 환경에서 실행되는 profile은 real밖에 없다. 해당 profile은 Travis CI 배포 자동화를 위한 profile이니 무중단 배포를 위한 profile 2개 (real1, real2)를 추가한다.  
[![그림 1-19](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img19.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img19.png)  
  
`application-real1.properties`  
```   
server.port=8081
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```  
  
`application-real2.properties`  
```  
server.port=8082
spring.profiles.include=oauth,real-db
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.session.store-type=jdbc
```  
  
server.port가 8080이 아닌 8081/8082로 되어 있다. 생성한 뒤 깃허브로 푸시  
  
__엔진엑스 설정 수정__  
무중단 배포의 핵심은 엔진엑스 설정이다. 배포 때마다 엔진엑스의 프록시 설정(스프링 부트로 요청을 흘려보내는)이 순식간에 교체된다. 여기서 프록시 설정이 교체될 수 있도록 설정을 추가한다.  
엔진엑스 설정이 모여있는 `/etc/nginx/conf.d/` 에 `service-url.inc` 라는 파일을 하나 생성한다.  
```  
sudo vim /etc/nginx/conf.d/service-url.inc  
```  
그리고 다음 코드를 입력한다.  
```  
set $service_url http://127.0.0.1:8080;  
```  
저장하고 종료한 뒤(:wq) 해당 파일은 엔진엑스가 사용할 수 있게 설정한다. `nginx.conf` 파일을 연다.  
```  
sudo vim /etc/nginx/nginx.conf  
```  
location / 부분을 찾아 다음과 같이 변경한다.  
```java  
include /etc/nginx/conf.d/service-url.inc;

location / {
        proxy_pass $service_url;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
}
```  
[![그림 1-20](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img20.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img20.png)  
  
저장하고 종료한 뒤(:wq) __재시작__ 한다.  
```  
sudo service nginx restart  
```  
다시 브라우저에서 정상적으로 호출되는지 확인한다. 확인되었다면 엔진엑스 설정까지 잘 된 것이다.  
  
__배포 스크립트 작성__  
먼저 step2와 중복되지 않기 위해 EC2에 step3 디렉토리를 생성한다.  
```  
mkdir ~/app/step3 && mkdir ~/app/step3/zip  
```  
무중단 배포는 앞으로 step3를 사용한다. `appspec.yml` 역시 step3로 배포되도록 수정한다.  
```  
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/step3/zip/
    overwrite: yes
```  
무중단 배포를 진행할 스크립트들은 총 5개이다.  
+ stop.sh: 기존 엔진엑스에 연결되어 있진 않지만, 실행 중이던 스프링 부트 종료  
+ start.sh: 배포할 신규 버전 스프링 부트 프로젝트를 `stop.sh`로 종료한 'profile'로 실행  
+ health.sh: `start.sh` 로 실행시킨 프로젝트가 정상적으로 실행됬는지 체크  
+ switch.sh: 엔진엑스가 바라보는 스프링 부트를 최신 버전으로 변경  
+ profile.sh: 앞선 4개 스크립트 파일에서 공용으로 사용할 'profile'과 포트 체트 로직  
  
`appspec.yml`에 앞선 스크립트를 사용하도록 설정  
```yml  
hooks:
  AfterInstall:
    - location: stop.sh # 엔진엑스와 연결되어 있지 않은 스프링 부트를 종료
      timeout: 60
      runas: ec2-user
  ApplicationStart:
    - location: start.sh # 엔진엑스와 연결되어 있지 않은 Port로 새 버전의 스프링 부트를 시작
      timeout: 60
      runas: ec2-user
  ValidateService:
    - location: health.sh # 새 스프링 부트가 정상적으로 실행됬는지 확인
      timeout: 60
      runas: ec2-user
```  
Jar파일이 복사된 이후부터 차례로 앞선 스크립트들이 실행된다고 보면 된다. 각 스크립트들 역시 scripts 디렉토리에 추가한다.  
[![그림 1-21](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img21.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img21.png)  
  
`profile.sh`  
```sh  
#!/usr/bin/env bash

# bash는 return value가 안되니 *제일 마지막줄에 echo로 해서 결과 출력*후, 클라이언트에서 값을 사용한다

# 쉬고 있는 profile 찾기: real1이 사용 중이면 real2가 쉬고 있고, 반대면 real1이 쉬고 있음

function find_idle_profile()
{
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile) #1. 

    if [ ${RESPONSE_CODE} -ge 400 ] # 400 보다 크면 (즉, 40x/50x 에러 모두 포함)
    then
        CURRENT_PROFILE=real2
    else
        CURRENT_PROFILE=$(curl -s http://localhost/profile)
    fi

    if [ ${CURRENT_PROFILE} == real1 ]
    then
        IDLE_PROFILE=real2 #2. 
    else
        IDLE_PROFILE=real1
    fi

    echo "${IDLE_PROFILE}" #3. 
}

# 쉬고 있는 profile의 port 찾기
function find_idle_port()
{
  IDLE_PROFILE=$(find_idle_profile)

  if [ ${IDLE_PROFILE} == real1 ]
  then
    echo "8081"
  else
    echo "8082"
  fi
}
```  
1. $(curl -s -o /dev/null -w "%{http_code}" http://localhost/profile)  
+ 현재 엔진엑스가 바로보고 있는 스프링 부트가 정상적으로 수행 중인지 확인  
+ 응답값을 HttpStatus로 받는다.  
+ 정상이면 200, 오류가 발생한다면 400~503 사이로 발생하니 400 이상은 모두 예외로 보고 real2를 현재 profile로 사용  
2. IDLE_PROFILE  
+ 엔진엑스와 연결되지 않은 profile이다.  
+ 스프링 부트 프로젝트를 이 profile로 연결하기 위해 반환한다.  
3. echo "${IDLE_PROFILE}"  
+ bash라는 스크립트는 값을 반환하는 기능이 없다.  
+ 그래서 제일 마지막 중에 echo로 결과를 출력 후, 클라이언트에서 그 값을 잡아서 ($(find_idle_profile)) 사용한다.  
+ 중간에 echo를 사용해선 안된다.  
  
`stop.sh`  
```sh  
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH) #1.
source ${ABSDIR}/profile.sh #2.

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 ${IDLE_PID}
  sleep 5
fi
```  
1. `ABSDIR=$(dirname $ABSPATH)`  
+ 현재 `stop.sh`가 속해 있는 경로를 찾는다.  
+ 하단의 코드와 같이 `profile.sh`의 경로를 찾기 위해 사용  
2. source ${ABSDIR}/profile.sh  
+ 자바로 보면 일종의 import 구문이다.  
+ 해당 코드로 인해 `stop.sh` 에서도 `profile.sh`의 여러 function 을 사용할 수 있게 된다.  
  
`start.sh`  
```sh  
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app/step3
PROJECT_NAME=freehyun-springboot-webservice

echo "> Build 파일 복사"
echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 새 어플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

IDLE_PROFILE=$(find_idle_profile)

echo "  > $JAR_NAME 를 profile=$IDLE_PROFILE 로실행합니다."

nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=$IDLE_PROFILE \
    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```  
1. 기본적인 스크립트는 step2의 `deploy.sh`와 유사  
2. 다른 점이라면 IDLE_PROFILE을 통해 properties 파일을 가져오고(`application-$IDLE_PROFILE.properties`), active profile을 지정하는 것(`-Dspring.profiles.active=$IDLE_PROFILE`)뿐이다.  
3. 여기서도 IDLE_PROFILE을 사용하니 `profile.sh`을 가져와야 한다.  
  
`health.sh`  
```sh  
#!/usr/bin/env bash

# 엔진엑스와 연결되지 않은 포트로 스프링 부트가 잘 수행되었는지 체크한다.
# 잘 떴는지 확인되어야 엔진엑스 프록시 설정을 변경(switch_proxy)한다.
# 엔진엑스 프록시 설정 변경은 switch.sh에서 수행
ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://localhost:$IDLE_PORT/profile "
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ]
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
      echo "> Health check 성공"
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 status가 UP이 아닙니다."
      echo "> Health check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```  
1. 엔진엑스와 연결되지 않은 포트로 스프링 부트가 잘 수행되었는지 체크한다.  
2. 잘 떴는지 확인되어야 엔진엑스 프록시 설정을 변경(switch_proxy)한다.  
3. 엔진엑스 프록시 설정 변경은 `switch.sh`에서 수행  
  
`switch.sh`  
```sh  
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 Port: $IDLE_PORT"
    echo "> Port 전환"
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc
    #1.
    #2.
    CURRENT_PROFILE=$(curl -s http://localhost/profile)
    echo "> 엔진엑스 Current Profile : $CURRENT_PROFILE"
    
    echo "> 엔진엑스 Reload"
    sudo service nginx reload #3.
}

```  
1. `echo "set \$service_url http://127.0.0.1:${IDLE_PORT};"`  
+ 하나의 문장을 만들어 파이프라이(|)으로 넘겨주기 위해 echo를 사용  
+ 엔진엑스가 변경할 프록시 주소를 생성  
+ 쌍따옴표 (")를 사용해야 한다.  
+ 사용하지 않으면 $service_url을 그대로 인식하지 못하고 변수를 찾게 된다.  
2. | sudo tee /etc/nginx/conf.d/service-url.inc  
+ 앞에서 넘겨준 문장을 service-url.inc에 덮어쓴다.  
3. sudo service nginx reload  
+ 엔진엑스 설정을 다시 불러온다.  
+ restart와는 다르다.  
+ restart는 잠시 끊기는 현상이 있지만, reload는 끊김 없이 다시 불러온다.  
+ 다만, 중요한 설정들은 반영되지 않으므로 restart를 사용해야 한다.  
+ 여기선 외부의 설정 파일인 service-url을 다시 불러오는 거라 reload로 가능하다.  
  
## 무중단 배포 테스트  
배포 테스트를 하기 전, 한 가지 추가 작업을 진행하도록 한다. 잦은 배포로 Jar 파일명이 겹칠수 있다. 매번 버전을 올리는 것이 귀찮으므로 자동으로 버전값이 변경될 수 있도록 하자  
`build.gradle`  
```  
version '1.0.1-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")  
```  
1. build.gradle은 Groovy 기반의 빌드툴이다.  
2. 당연히 Groovy 언어의 여러 문법을 사용할 수 있는데, 여기서는 new Date()로 빌드할 때마다 그 시간이 버전에 추가되도록 구성하였다.  
  
여기까지 구성한 뒤 최종 코드를 깃허브로 푸시한다. 배포가 자동으로 진행되면 CodeDeploy 로그로 잘 진행되는지 확인해 보자  
```  
tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log  
```  
다음과 같은 메시지가 차례로 출력된다.  
[![그림 1-22](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img22.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img22.png)  
스프링 부트 로그도 확인한다.  
```  
vim ~/app/step3/nohup.out  
```  
한 번 더 배포하면 그때는 real2로 배포된다. 이 과정에서 브라우저 새로고침을 해보면 전혀 중단 없는 것을 확인할 수 있다. 2번 배포를 진행한 뒤에 다음과 같이 자바 애플리케이션 실행 여부를 확인한다.  
```  
ps -ef | grep java  
```  
다음과 같이 2개의 애플리케이션이 실행되고 있음을 알 수 있다.  
[![그림 1-23](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img23.png)](/assets/Web/AWS/2020-04-28-SpringBoot-AWS-10-img23.png)  
  
이제 이 시스템은 마스터 브랜치에 푸시가 발생하면 자동으로 서버 배포가 진행되고, 서버 중단 역시 전혀 없는 시스템이 되었다.  
