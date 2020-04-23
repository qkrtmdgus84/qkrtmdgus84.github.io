---
title: "Ⅵ. AWS EC2"
excerpt: "AWS 서버 환경을 만들어보자"

categories:
    - SpringBoot & AWS
tags:
    - SpringBoot
    - Amazon Web Services
    - IntelliJ IDEA 
    - Gradle
    - JPA
    - Git
use_math: true;
last_modified_at: 2020-04-09

# published : false
--- 
> 개발 언어 : __Java 8(JDK 1.8)__  
> 개발 환경 : __Gradle 4.8 ~ Gradle 4.10.2__  
> 참조 :  
> [기억보단 기록을](https://jojoldu.tistory.com/)  
> [스프링 부트와 AWS로 혼자 구현하는 웹서비스 (프리렉, 이동욱 지음)](https://jojoldu.tistory.com/463)  
> [예제 코드 내려받기](http://bit.ly/fr-springboot)
  
## AWS 회원가입  
Master 혹은 Visa 카드가 필요하다. 본인의 카드 중 Master 혹은 Visa 카드를 준비한 뒤 진행하자.  
AWS 공식 사이트[https://aws.amazon.com/ko/](https://aws.amazon.com/ko/)로 이동한 뒤 무료계정 만들기를 선택한다.  
  
신규로 생성할 AWS 계정 정보를 등록한다.  
[![그림 1-1](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img01.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img01.png)  
  
[동의하고 계정 만들기]를 클릭한 후 해당 정보를 입력한다. 여기서 계정 유형은 __개인__ 으로 선택한다. 주소와 연락처 정보는 영어 또는 다른 라틴 문자 기반 언어로 입력해야 한다. 국가/리전은 대한민국으로 선택한다.  
[![그림 1-2](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img02.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img02.png)  
  
다음은 결제 정보란이다. 준비한 Master 혹은 Visa 카드의 정보를 등록한다. 이때 해당 카드에는 최소 1달러가 결제 가능해야 한다. 보안 전송을 클릭하면 계좌 확인을 위한 1달러 결제가 진행된다. 문자로 결제 성공 알람이 오면 휴대폰 정보와 보안 문자를 기재한다.  
보안 문자를 정상적으로 입력했다면 등록한 전화번호로 전달된 4자리 코드를 등록한다.  
  
※ 환화로 대략 1200원 정도하는데 유효한 카드인지 확인하는 것이므로 실제 청구되지는 않는다. 혹시 문자로 코드 확인 문자가 안온다면 수신차단이나 메시지 차단을 확인 해보자.  
[![그림 1-3](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img03.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img03.png)  
  
[![그림 1-4](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img04.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img04.png)  
  
[![그림 1-5](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img05.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img05.png)  
  
지원 플랜을 선택하는데 무료로 사용하기 위해서 무료 기본 플랜을 선택  
[![그림 1-6](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img06.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img06.png)  
  
AWS 회원가입이 끝났다면 콘솔 로그인 버튼을 클릭해서 가입한 정보로 로그인을 진행한다. 가입할 때 사용된 이메일 주소와 비밀번호를 차례로 입력  
[![그림 1-7](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img07.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img07.png)  
  
[![그림 1-8](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img08.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img08.png)  
  
## EC2 인스턴스 생성하기  
EC2(Elastic Compute Cloud)는 AWS에서 제공하는 성능, 용량 등을 유동적으로 사용할 수 있는 서버이다. 보통 "AWS에서 리눅스 서버 혹은 윈도우 서버를 사용한다" 라고 하면 이 EC2를 이야기하는 것이다.  
AWS에서 무료로 제공하는 프리티어 플랜에서는 EC2 사용에 제한이 있다.  
1. 사양이 t2.micro만 가능하다.  
+ vCPU(가상CPU) 1Core, 메모리 1GB 사양이다.  
+ 보통 vCPU는 물리 CPU 사양의 절반 정도의 성능을 가진다.  
2. 월 750시간의 제한이 있다. 초과하면 비용이 부과된다.  
+ 24시간 * 31일 = 744시간이다.  
+ 즉, 1대의 t2.micro만 사용한다면 24시간 사용할 수 있다.  
  
제한 사항을 주의하면서 AWS를 1년간 무료로 사용한다.  
EC2를 만들기 전에 본인의 리전을 확인해 보자  
서울로 되어있지 않다면 서울로 변경한다.  
[![그림 1-9](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img09.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img09.png)  
  
서울로 리전을 변경했다면 검색창에서 ec2를 입력하여 EC2 서비스를 클릭한다.  
[![그림 1-10](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img10.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img10.png)  
  
EC2 대시보드가 나오는데, [인스턴스 시작] 버튼을 클릭  
인스턴스란 EC2 서비스에 생성된 가상머신을 말한다.  
[![그림 1-11](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img11.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img11.png)  
  
인스턴스를 생성하는 첫 단계는 AMI(Amazon Machine Image)를 선택하는 것이다. AMI는 EC2 인스턴스를 시작하는 데 필요한 정보를 __이미지로 만들어 둔 것__ 을 말한다. 인스턴스라는 가상머신에 운영체제 등을 설치할 수 있게 구워 넣은 이미지로 생각하면 된다.  
Amazon Linux AMI를 선택  
[![그림 1-12](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img12.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img12.png)  
  
인스턴스 유형에서는 프리티어로 표기된 t2.micro를 선택  
[![그림 1-13](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img13.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img13.png)  
  
다음 단계는 세부정보 구성이다. 기업에서 사용할 경우 화면상에서 표기된 VPC, 서브넷 등을 세세하게 다루지만, 여기서는 혼자서 1대의 서버만 사용하니 별다른 설정없이 넘어간다.  
[![그림 1-14](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img14.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img14.png)  
  
다음 단계는 스토리지 선택이다. 스토리지는 흔히 하드디스크라고 부르는 서버의 디스크(SSD도 포함)를 말하며 서버의 용량을 얼마나 정할지 선택하는 단계이다.  
설정의 기본값은 8GB이다. 하지만 30GB까지 프리티어로 가능하다. 물론 그 이상의 사이즈는 비용이 청구되니 프리티어의 최대치인 30GB로 변경한다.  
[![그림 1-15](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img15.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img15.png)  
  
태그에는 웹 콘솔에서 표기될 태그인 Name 태그를 등록한다. 태그는 해당 인스턴스를 표현하는 여러 이름으로 사용될 수 있다.  
※ 인스타그램, 페이스북 등 SNS의 태그와 동일한 역할을 한다.  
여러 인스턴스가 있을 경우 이를 태그별로 구분하면 검색이나 그룹 짓기 편하므로 여기서 본인 서비스의 인스턴스를 나타낼 수 있는 값으로 등록한다.  
[![그림 1-16](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img16.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img16.png)  
  
다음으로 보안 그룹이다. 유형 항목에서 SSH이면서 포트 항목에서 22인 경우는 AWS EC2에 터미널로 접속할 때를 말한다.  
보안은 높을수록 좋으니 pem 키 관리와 지정된 IP에서만 ssh 접속이 가능 하도록 구성하는 것이 안전하다. 그래서 본인 집의 IP를 기본적으로 추가하고(내 IP를 선택하면 현재 접속한 장소의 IP가 자동 지정됨) 카페와 같이 집 외에 다른 장소에서 접속할 때는 해당 장소의 IP를 다시 SSH규칙에 추가하는 것이 안전하다. 현재 프로젝트의 기본 포트인 8080을 추가하고 [검토 및 시작] 버튼을 클릭한다.  
[![그림 1-17](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img17.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img17.png)  
  
검토 화면에서 보안 그룹 경고를 하는데, 이는 8080이 전체 오픈이 되어서 발생한다. 8080을 열어 놓는 것은 위험한 일이 아니니 바로 [시작하기] 버튼을 클릭한다.  
[![그림 1-18](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img18.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img18.png)  
  
인스턴스로 접근하기 위해서는 pem 키(비밀키)가 필요하다.  
인스턴스는 지정된 pem 키(비밀키)와 매칭되는 공개키를 가지고 있어, 해당 pem 키 외에는 접근을 허용하지 않는다.  
pem 키는 이후 EC2서버로 접속할 때 필수 파일이니 절대 유출되서도 안되고 잃어버려서도 안된다.  
기존에 생성된 pem 키가 있다면 선택하고 없다면 신규로 생성한다.  
[![그림 1-19](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img19.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img19.png)  
  
pem 키 까지  내려받았다면 다음과 같이 인스턴스 생성 시작 페이지로 이동하고 인스턴스 id를 클릭하여 EC2목록으로 이동한다.  
[![그림 1-20](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img20.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img20.png)  
  
인스턴스가 생성 중인 것을 확인할 수 있으며 Name 태그로 인해 Name이 노출되는 것도 확인 가능하다.  
[![그림 1-21](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img21.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img21.png)  
  
생성이 다 되었다면 IP와 도메인이 할당된 것을 확인할 수 있다.  
[![그림 1-22](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img22.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img22.png)  
  
인스턴스 생성 시에 항상 새IP를 할당하는데, 인스턴스를 중지하고 다시 시작할 때 새IP가 할당된다. IP가 매번 변경되지 않고 고정 IP를 갖도록 하자.  
AWS의 고정 IP를 Elastic IP(EIP, 탄력적 IP)라고 한다. EC2 인스턴스 페이지의 왼쪽 카테고리에서 탄력적 IP를 눌러 선택하고 주소가 없으므로 [새 주소 할당] 버튼을 클릭해서 바로[할당] 버튼을 클릭한다.  
[![그림 1-23](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img23.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img23.png)  
[![그림 1-24](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img24.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img24.png)  
[![그림 1-25](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img25.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img25.png)  
  
새 주소 할당이 완료되면 __탄력적 IP__ 가 발급된다.  
[![그림 1-26](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img26.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img26.png)  
  
탄력적 IP와 EC2 주소를 연결한다. 탄력적 IP를 확인하고, 페이지 위에 있는 [Actions] 버튼 -> [탄력적 IP 주소 연결] 클릭한다.  
[![그림 1-27](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img27.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img27.png)  
  
주소 연결을 위해 생성한 EC2 이름과 IP를 선택하고 [연결] 버튼을 클릭한다.  
[![그림 1-28](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img28.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img28.png)  
  
연결이 완료되면 왼쪽 카테고리에 있는 [인스턴스] 탭을 클릭해서 다시 인스턴스 목록 페이지로 이동한다.  
[![그림 1-29](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img29.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img29.png)  
  
해당 인스턴스의 __퍼블릭, 탄력적 IP__ 가 모두 잘 연결되었는지 확인한다.  
[![그림 1-30](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img30.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img30.png)  
  
※ __탄력적 IP를 생성하고 EC2 서버에 연결하지 않으면 비용이 발생__ 한다. 즉, __생성한 탄력적 IP는 무조건 EC2에 바로 연결__ 해야 하며 만약 더는 사용할 인스턴스가 없을 때도 탄력적 IP를 삭제 해야 한다.  
마찬가지로 비용 청구가 되므로 꼭 잊지 않고 삭제해야 한다.  
  
__추가 (탄력적 IP 취소, 삭제)__  
해당 퍼블릭 주소를 클릭한 [작업] 메뉴에서 [탄력적 IP 주소 연결해제] 선택  
[![그림 1-31](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img31.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img31.png)  
  
연결해제 한 다음 다시 해당 퍼블릭 주소로 들어가 [작업] 메뉴에서 [탄력적 주소 릴리스] 선택  
[![그림 1-32](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img32.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img32.png)  
  
## EC2 서버에 접속하기  
윈도우 사용자들은 리눅스 웹서버에 접속하거나 파일을 보내기위해서 PuTTY나 FileZilla와 같은 프로그램을 사용한다. 그리고 IntelliJ를 사용하고 있다면 IntelliJ에서 바로 웹서버 터미널을 사용할 수 있다. 두 가지 방식을 설명 하려고 한다. 본인이 편한걸로 사용하길 바란다.  
첫 번째 putty로 웹서버에 접속해보자  
putty 사이트[https://www.putty.org](https://www.putty.org)에 접속하여 실행 파일을 내려받는다.  
[![그림 1-33](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img33.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img33.png)  
  
실행 파일은 2가지 이다.  
+ putty.exe  
+ puttygen.exe  
  
두 파일을 모두 내려받은 뒤, `puttygen.exe` 파일을 실행한다.  
[![그림 1-34](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img34.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img34.png)  
  
putty는 pem 키로 사용이 안 되며 pem 키를 ppk 파일로 변환을 해야만 한다. puttygen은 이 과정을 진행해 주는 클라이언트이다. puttygen화면에서 상단 [Conversiions -> Import Key]를 선택해서 내려받은 pem 키를 선택한다.  
[![그림 1-35](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img35.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img35.png)  
  
[![그림 1-36](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img36.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img36.png)  
  
자동 변환이 진행된다. [Save private key] 버튼을 클릭하여 ppk 파일을 생성한다. 경고 메시지가 뜨면 [예] 를 클릭하고 넘어간다.  
[![그림 1-37](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img37.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img37.png)  
  
[![그림 1-38](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img38.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img38.png)  
  
ppk 파일이 저장될 위치와 ppk 이름을 등록한다.  
[![그림 1-39](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img39.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img39.png)  
  
ppk 파일이 잘 생성되었으면 `putty.exe` 파일을 실행  
[![그림 1-40](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img40.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img40.png)  
  
+ HostName: username@public_Ip 를 등록한다. 생성한 Amazon Linux는 ec2-user가 username이라서 ec2-user@탄력적 IP 주소를 등록하면 된다.  
+ Port: ssh 접속 포트인 22를 등록한다.  
+ Connection type: SSH를 선택한다.  
  
항목들을 모두 채웠다면 왼쪽 사이드바에 있는 [Connection -> SSH -> Auth]를 차례로 클릭해서 ppk 파일을 로드할 수 있는 화면으로 이동한다.  
[![그림 1-41](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img41.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img41.png)  
  
생성한 ppk 파일을 선택해서 불러온 다음 [Session] 탭으로 이동하여 [Saved Sessions]에 현재 설정들을 저장할 이름을 등록하고 [Save] 버튼을 클릭한다.  
[![그림 1-42](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img42.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img42.png)  
  
[![그림 1-44](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img44.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img44.png)  
  
저장한 뒤 [open] 버튼을 클릭하면 SSH 접속 알림이 등장한다. [예]를 클릭  
[![그림 1-43](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img43.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img43.png)  
  
[![그림 1-45](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img45.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img45.png)  
  
두 번째 IntelliJ에서 SSH연결하기  
[Tools -> Deployment -> Browse Remote Host] 실행  
[![그림 1-46](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img46.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img46.png)  
  
Remote Host가 열리면 접속설정을 추가하기 위해 ...버튼을 누른다.  
[![그림 1-47](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img47.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img47.png)  
  
Name을 지정한후 Type은 SFTP로 선택한다.  
[![그림 1-48](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img48.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img48.png)  
  
호스트에 아이피를 적어준 후 리눅스서버의 계정이름과 EC2인스턴스 생성시 받았던 암호키파일을 선택한 후 호스트아이피 왼쪽에 있는 테스트 버튼을 눌러 연결 테스트를 한다.  
[![그림 1-49](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img49.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img49.png)  
  
[![그림 1-50](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img50.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img50.png)  
  
Remote Host에 웹서버 디렉토리 구조가 반영됬다.  
[![그림 1-51](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img51.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img51.png)  
  
웹서버 터미널을 사용하려면 [Tools -> start SSH session]을 선택한 후 대상 웹서버 이름을 선택  
[![그림 1-52](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img52.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img52.png)  
  
[![그림 1-53](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img53.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img53.png)  
  
[![그림 1-54](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img54.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img54.png)  
  
## 아마존 리눅스 서버 생성시 해야할 설정  
+ Java 8 설치
+ 타임존 변경
+ 호스트 네임 변경
  
__Java 8 설치__  
EC2에서 명령어를 실행  
[![그림 1-55](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img55.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img55.png)  
  
설치가 완료되었다면 인스턴스의 Java버전을 8로 변경  
[![그림 1-56](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img56.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img56.png)  
  
Java8을 선택(2입력)  
[![그림 1-57](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img57.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img57.png)  
  
버전이 변경되었다면 사용하지 않는 Java7을 삭제  
[![그림 1-58](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img58.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img58.png)  
  
현재 버전이 Java8인지 확인  
[![그림 1-59](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img59.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img59.png)  
  
__타임존 변경__  
한국 시간으로 변경  
[![그림 1-60](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img60.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img60.png)  
  
__Hostname 변경__  
각 서버가 어느 서비스인지 표현하기 위해 HOSTNAME을 변경  
[![그림 1-61](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img61.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img61.png)  
  
HOSTNAME을 본인이 원하는 서비스명으로 변경  
[변경 전]  
[![그림 1-62](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img62.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img62.png)  
  
[i] 키를 눌러 입력모드로 변경 후 esc를 누르고 콜론(:)wq로 저장 후 종료  
[변경 후]  
[![그림 1-63](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img63.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img63.png)  
  
서버 재부팅  
```  
sudo reboot
```
재부팅이 끝나고 나서 다시 접속해 보면 HOSTNAME이 잘 변경된 것을 확인할 수 있다.  
[![그림 1-64](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img64.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img64.png)  
  
/etc/hosts에 변경한 hostname을 등록한다.  
/etc/hosts 파일을 열어 본다.  
```  
sudo vim /etc/hosts
```
HOSTNNAME을 등록한다.  
[![그림 1-65](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img65.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img65.png)  
  
:wq로 저장하고 종료한 뒤 정상적으로 등록되었는지 확인해 보자.  
```  
curl 등록한 호스트 이름
```  
잘 등록하였다면 80포트로 접근이 안 된다는 에러가 발생할 것이다.  
아직 80포트로 실행된 서비스가 없음을 의미한다. 즉, curl 호스트 이름으로 실행은 잘 되었음을 의미한다.  
[![그림 1-66](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img66.png)](/assets/Web/AWS/2020-04-16-SpringBoot-AWS-06-img66.png)  
  
