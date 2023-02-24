# RDS 테이블 및 yml 세팅
### RDS 테이블 생성
- RDS에 테이블을 생성해야한다. 생성하는 방법은 내 로컬에서 RDS에 접근이 가능하기때문에 테이블 sql을 복사해서 직접 생성하면 된다.
- 하면서 발생했던 에러는 다음과 같다.
    ***1. Book 엔티티의 "desc" 컬럼이 Maria DB 예약어라 실행되지 않는 에러***
    ***2. 대소문자 구분으로 인해 테이블을 못찾는 에러, 그래서 data.sql에서 insert문에 테이블이름을 대문자로 바꿔줬음***

### yml(properties) 파일 관리
- 내가 작업하는 로컬 서버 환경과 테스트 또는 실제서버의 환경은 다를 것이다. 지금 내 프로젝트만 해도 로컬은 H2, RDS는 Maria DB다.
- 그렇기때문에 실제 서버 환경 세팅을 위한 application-real.yml을 구성해서 커밋했다. 그리고 git 같은곳에 노출되면 안되는 정보들이 있다. 예를들어 db 호스트 주소, db 계정, 비밀번호 같은 것들이다.
보안 이슈가 되는 정보들은 ec2 서버에 yml을 생성해서 관리한다. 이름은 apllication-real-db.yml로 생성했다. 
- 결국은 application-real.yml을 활성화 시키면 안에서 include 명령어로 application-real-db.yml 파일이 함께 활성화 될 것이다. 이 작업은 배포 쉘 스크립트(.sh)에서 동작하게 했다
```
nohup java -jar \ -Dspring.config.location=classpath:/application.yml,/home/ec2-user/app/application-real-db.yml,classpath:/application-real.yml \
-Dspring.profiles.active=real \
$REPOSITORY/$JAR_NAME 2>&1 &
```
- ***참고로 나는 config.location에서 파일들 사이에 , 다음에 띄어쓰기를 하면 다음 파일을 인식못하는 문제가 있어서 시간을 좀 잡아먹었다 ㅠㅠ***

# S3
1. AWS IAM가서 AWS Key 발급 그리고 github action에 등록
2. AWS S3에 저장할 수 있도록 버킷 생성
3. 프로젝트를 S3에 저장하는 명령어를 github action yml 에 등록
여기까지하면 S3에 빌드 파일이 들어있을 것이다.

# CodeDeploy
1. AWS IAM 역할 추가하기. EC2가 CodeDeploy를 연동받을 수 있게 위함이다.
  - S3의 1.은 IAM 사용자 추가이고 CodeDeploy의 1.은 역할 추가이다. 
    두개의 차이는 사용자는 AWS 서비스 외의 사용할 수 있는 권한(로컬 PC,IDC 서버 등)을 추가하는거고
    역할은 AWS 서비스(EC2, CodeDeploy, SQS)에서만 할당할 수 있는 권한이다.
2. EC2에 들어가서 CodeDeploy 에이전트 설치
3. AWS IAM 역할 추가하기. CodeDeploy에서 EC2에 접근하기 위함이다.
4. AWS에 가서 CodeDeploy 생성
5. EC2에 들어가서 S3에서 넘겨받을 디렉토리 생성(CodeDeploy는 저장을 못하고 배포만 한다)
6. appspec.yml에는 CodeDeploy가 EC2 어느 디렉토리에 배포할건지에 대한 정보를 기입한다.
7. 새로운 파일이 commit 되면 이전 애플리케이션은 중단하고 업데이트 된 jar를 실행하기 위해 pid 값을 설정한다. 
8. EC2까지 배포된 jar 파일을 실행할 deploy.sh을 만든다.
9. appspec.yml에는 codedeploy가 자동 배포 할 수 있도록 설정한다.
