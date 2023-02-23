- 배포는 이번이 2번째 경험이다. 처음에는 "스프링부트와 aws로 혼자 구현하는 웹서비스" 책을 그대로 따라하면서 진행했고, 이번에는 내 프로젝트를 책을 참고해서 직접 배포한다. 
경험이 적어서 그런지 생소하기도 하고 꽤나 어렵다. 그래도 배포까지의 전체적인 루틴과 단어를 정리하려고 한다. 책과 다른 부분도 있고 생소한 AWS 배포와 최대한 친해지기 위해서다.


# 1. 인스턴스 생성(EC2 인스턴스)

### AMI(Amazon Machine Image)
- ec2 인스턴스를 시작하는데 필요한 정보를 이미지로 만들어둔 것
- 나는 Amazon Linux 2를 사용했다.

### tc2.micro(인스턴스 유형)
- tc는 요금타입을 의미한다.
- 다른 서비스와 달리 크레딧이란 일종의 cpu를 사용할 수 있는 포인트 개념이 있다. 인스턴스 크기에 따라 정해진 비율로 cpu 크레딧을 계속 받게 되며, 사용하지 않을 때는 크레딧을 축적하고,
사용할 떄 이 크레딧을 사용한다.
- 정해진 사양보다 더 높은 트래픽이 오면 크레딧을 좀 더 적극적으로 사용하면서 트래픽을 처리한다. 크레딧이 모두 사용되면 더이상 ec2를 사용할 수 없다.

### 보안그룹(=방화벽)
- 1. SSH, 22번 포트, 내IP : AWS EC2에 터미널로 접속할 때 필요한 정보다.
  - 모든 IP를 오픈하는건 지양하자. 
- 특정 IP를 지정할떄는 당연히 외부 IP를 등록해야한다. 멍청하게 cmd키고 ipconfig 치고 있지 말자.
- public IP로 세팅하고 putty.gen으로 접속해도 안될때가 있는데 그땐 보안그룹에서 AnyWhere로 설정 후 접속 성공한다음 다시 public IP로 설정하면 되는것 같다.

### pem 키(비밀키)
- ec2 인스턴스에 접근하기 위해서 pem키가 필요하다.
- 인스턴스는 pem키(비밀키)와 매칭되는 공개키를 가지고 있기때문에 해당 pem키 외에는 접근 불가능
- pem키는 일종의 마스터키이기때문에 절대 유출하면 안되고, 꼭 따로 보관하고 있어야한다.

### EIP(탄력적 IP)
- 인스턴스도 결국 하나의 서버이기때문에 IP가 존재한다. 인스턴스가 생성될때 IP가 생성되는데, 인스턴스를 중지하고
다시 시작하면 새로운 IP가 할당된다. 이러면 매번 접속해야하는 IP가 변경되기때문에 불편하다.
- 그렇기때문에 인스턴스의 IP가 매번 변경되지 않게 고정 IP를 가지게 해야하는데, 이 고정 IP를 탄력적 IP라고 한다.
- 탄력적 IP는 EC2 서버에 연결하지않으면 비용이 발생한다.

### putty
- puttygen.exe로 pem키를 ppk 파일로 변환한다.
- putty.exe에 ppk를 로드해서 ssh로 ec2에 접속한다.

# 2. EC2 내 설정 
### java 11 설치
- JAVA 11 파일 다운로드 (-o 옵션을 줌으로써 jdk11.rpm 이라는 파일명으로 다운받아진다)
```
sudo curl -L https://corretto.aws/downloads/latest/amazon-corretto-11-x64-linux-jdk.rpm -o jdk11.rpm
```
- JAVA 11 설치
```
sudo yum localinstall jdk11.rpm
```

- JAVA 버전 변경 (원하는 버전의 번호를 입력)
```
sudo /usr/sbin/alternatives --config java
```

- JAVA 버전 확인
```
java --version
javac --version
```

### 타임존 변경
```
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```
```
date
```

### Hostname 변경
- 여러 서버를 관리 중일 경우 IP만으로 어떤 서비스의 서버인지 확인이 어렵다. 총 두개의 hostname을 수정해야한다.
- 첫번쨰
```
sudo hostnamectl set-hostname 원하는이름.localdomain
```
```
sudo reboot
```

- 두번쨰
```
sudo vim /etc/hosts
```
```
다음을 타이핑한다
127.0.0.1 등록한 hostname
```

# 3. RDS 생성 
- Amazon Aurora
   - Amazon Aurora 교체 용이성 (Aurora는 AWS에서 MySQL과 PostgreSQL을 클라우드 기반에 맞게 재구성한 DB)
   - MySQL, PostgreSQL보다 월등한 성능
   - 하지만 Aurora는 프리티어 대상이 아님

- Maria DB
   - MySQL기반으로 만들어졌다.
   - 동일 하드웨어 사양으로 MySQL보다 향상된 성능

- 모나리자 EC2 인스턴스는 Maria DB(10.6.5) 다 

### RDS 생성하기
- 책을 참고하면서 되고 나는 처음할때 "데이터베이스 옵션"에 DB이름 등록을 안해서 나중에 문제점 찾느라 애먹었다.

### RDS 파라미터 그룹 생성 및 설정하기
- 책 참고하자. 설정할 내용은 "타임존, Character Set, Max Connection"
- 설정을 다하면 RDS에 연걸한다.

### 내 PC(인텔리제이)에서 RDS에 접속해보기
1. EC2에 사용된 보안그룹의 그룹 ID를 RDS 보안 그룹에 추가한다.
2. 그리고 인바운드의 첫번째는 방금 등록한 EC2 보안그룹을 등록한다.(EC2에서 RDS의 접근을 위해)
3. 인바운드 두번째는 현재 내 PC IP등록(인텔리제이에서 접속하기위함)

### 인텔리제이에서 RDS에 접근
- 접근 가능한 URL은 RDS 정보 페이지에 가면 '엔드 포인트'라고 있음. 엔드포인트가 접근가능한 URL다.
- DB 속성이 변경되지 않은게 있다. 이것도 변경해야함. (책 참고)

### EC2에서 RDS 접근확인
- putty SSH로 접한다.
- MySQL 설치(명령어 라인만 사용하기 위함)
```
sudo yum install mysql
```
- RDS 접속
```
mysql -u 계정 -p -h Host주소(=엔드포인트)
```

   
   
[출처] 스프링 부트와 AWS로 혼자 구현하는 웹 서비스, 이동욱   
[출처] https://developer-ping9.tistory.com/314
