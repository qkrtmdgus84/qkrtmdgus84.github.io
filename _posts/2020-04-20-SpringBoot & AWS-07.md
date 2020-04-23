---
title: "Ⅶ. AWS RDS"
excerpt: "AWS에 데이터베이스 환경을 만들어보자"

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
  
## RDS 인스턴스 생성하기  
[![그림 1-1](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img01.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img01.png)  
  
[![그림 1-2](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img02.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img02.png)  
  
DB 엔진 선택 에서 가격과 Amazon Aurora 교체 용이성 때문에 MariaDB를 선택하도록 하겠다.  
[![그림 1-3](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img03.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img03.png)  
  
[프리티어] 선택  
[![그림 1-4](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img04.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img04.png)  
  
스토리지 설정  
[![그림 1-5](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img05.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img05.png)  
  
DB 인스턴스와 마스터 사용자 정보를 등록  
[![그림 1-6](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img06.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img06.png)  
  
네트워크에서는 퍼블릭 액세스를 [예]로 변경  
보안 그룹에서 지정된 IP만 접근하도록 막을 예정이다.  
[![그림 1-7](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img07.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img07.png)  
  
데이터베이스 옵션에서는 기본값으로 넘어간다.  
[![그림 1-8](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img08.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img08.png)  
  
모든 설정이 끝났다면 [데이터베이스 생성]클릭  
## RDS 운영환경에 맞는 파라미터 설정하기  
필수 설정 3가지  
+ 타임존  
+ Character Set  
+ Max Connection  
  
왼쪽 카테고리에서 [파라미터 그룹] 탭을 클릭  
[![그림 1-9](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img09.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img09.png)  
  
[파라미터 그룹 생성] 버튼 클릭  
[![그림 1-10](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img10.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img10.png)  
  
생성한 MariaDB와 같은 버전을 맞춰야 한다. 앞에서 10.2.21 버전으로 생성했기 때문에 10.2를 선택한다.  
[![그림 1-11](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img11.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img11.png)  
  
생성이 완료되면 파라미터 그룹 목록 창에 새로 생성된 그룹을 볼 수 있다. 해당 파라미터 그룹을 클릭한다.  
[![그림 1-12](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img12.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img12.png)  
  
[파라미터 편집] 버튼을 클릭해 편집 모드로 전환한다.  
[![그림 1-13](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img13.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img13.png)  
  
설정값들을 변경해보자  
time_zone을 검색하여 [Asia/Seoul]로 변경  
[![그림 1-14](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img14.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img14.png)  
  
character 항목들과 collation 항목들 변경  
[![그림 1-15](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img15.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img15.png)  
[![그림 1-16](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img16.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img16.png)  
[![그림 1-17](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img17.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img17.png)  
[![그림 1-18](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img18.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img18.png)  
  
Max Connection 수정  
[![그림 1-20](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img20.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img20.png)  
  
[![그림 1-19](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img19.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img19.png)  
  
생성된 파라미터 그룹을 데이터베이스에 연결  
[![그림 1-21](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img21.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img21.png)  
  
옵션 항목에서 DB 파라미터 그룹은 default로 되어있다. DB 파라미터 그룹을 방금 생성한 신규 파라미터 그룹으로 변경한다.  
[![그림 1-22](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img22.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img22.png)  
  
[계속]을 누르면 수정 사항이 요약된 것을 볼 수 있다. 반영 시점은 [즉시 적용]으로 한다.  
[![그림 1-23](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img23.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img23.png)  
  
재부팅 진행  
[![그림 1-24](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img24.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img24.png)  
  
## 내 PC에서 RDS에서 접속해 보기  
로컬 PC에서 RDS로 접근하기 위해서 RDS의 보안 그룹에 본인 PC의 IP를 추가  
해당 DB 식별자를 클릭  
[![그림 1-25](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img25.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img25.png)  
  
RDS의 세부정보 페이지에서 보안 그룹의 값으로 할당된 링크를 클릭해서 보안 그룹 페이지로 이동  
[![그림 1-26](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img26.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img26.png)  
  
여기서 검색을 통해 이전에 만들어놓은 __EC2 인스턴스의 보안 그룹을 찾아서 그룹 ID를 복사__ 한다. 그리고 __보안 그룹 생성__ 버튼을 클릭한다.  
[![그림 1-27](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img27.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img27.png)  
  
복사된 보안 그룹 ID와 본인의 IP를 __RDS 보안 그룹의 인바운드__ 로 추가한다.  
[![그림 1-28](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img28.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img28.png)  
  
해당 DB 식별자 선택한 후 수정 클릭  
[![그림 1-21](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img21.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img21.png)  
  
수정 페이지에서 보안 그룹을 좀전에 생성한 보안 그룹 이름을 선택한다.  
[![그림 1-29](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img29.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img29.png)  
  
수정이 완료되면 보안그룹이 반영되었음을 확인할 수 있다.  
[![그림 1-30](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img30.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img30.png)  
  
RDS 정보 페이지에서 엔드 포인트를 확인한다.  
이 주소가 외부에서 RDS로 붙을수 있는 Host 주소이다.  
주소를 복사한다.  
[![그림 1-31](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img31.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img31.png)  
  
IntelliJ로 와서 복사한 Host주소와 RDS 생성시 등록했던 사용자 이름, 비밀번호, 데이터베이스 이름 등을 등록한 뒤 [Test connection]을 클릭해서 연결 테스트를 해보자  
[![그림 1-32](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img32.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img32.png)  
  
[![그림 1-33](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img33.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img33.png)  
  
생성된 콘솔창에서 SQL을 실행하자  
쿼리가 수행될 database를 선택한다.  
[![그림 1-34](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img34.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img34.png)  
  
데이터베이스가 선택된 상태에서 현재의 character_set, collation 설정을 확인한다.  
```  
show variables like 'c%';
```  
  
character_set_database, collation_connection 항목이 latin으로 되어있다.  
이 항목은 MariaDB에서만 RDS 파라미터 그룹으로는 변경이 안된다.  
직접 변경한다. 쿼리 실행  
```  
ALTER DATABASE 데이터베이스명
    CHARACTER SET = 'utf8mb4'
    COLLATE = 'utf8mb4_general_ci';
```  
  
쿼리를 수행하였다면 다시 한번 character set 확인한다.  
[![그림 1-35](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img35.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img35.png)  
  
타임존 확인  
[![그림 1-36](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img36.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img36.png)  
  
한글명이 잘 들어가는지 간단한 테이블 생성과 insert 쿼리를 실행해보자  
[![그림 1-37](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img37.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img37.png)  
  
## EC2에서 RDS에 접근 확인  
RDS가 EC2와 잘 연동되는지 확인한다.  
EC2에 SSH 접속을 진행한다. 터미널로 접속하자. 접속되었다면 MySQL 접근 테스트를 위해 MySQL CLI를 설치한다.  
(실제 EC2의 MySQL을 설치해서 쓰는게 아닌, 커맨드 라인만 쓰기 위한 설치이다.)
[![그림 1-38](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img38.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img38.png)  
  
설치가 다 되었다면 로컬에서 접근하듯이 계정, 비밀번호, 호스트 주소를 사용해 RDS에 접속한다.  
```  
mysql -u 계정 -p -h Host주소
```  
[![그림 1-39](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img39.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img39.png)  
  
RDS에 접속되었으면 실제로 생성한 RDS가 맞는지 간단한 쿼리를 실행 해보자  
```  
show databases;
```  
[![그림 1-40](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img40.png)](/assets/Web/AWS/2020-04-20-SpringBoot-AWS-07-img40.png)  
  
