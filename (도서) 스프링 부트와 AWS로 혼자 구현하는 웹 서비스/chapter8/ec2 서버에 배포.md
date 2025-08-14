## EC2 프로젝트에 clone 받기

1. EC2에 깃 설치

   `sudo yum install git`

2. 설치 상태 확인

   `git --version`

3. 깃 설치 후 git clone으로 프로젝트를 저장할 디렉토리 생성

   `mkdir ~/app && mkdir ~/app/step1`

4. 생성된 디렉토리로 이동

   `cd ~/app/step1`

5. 본인의 깃허브 프로젝트 주소 복사

6. git clone 진행

   `git clone 복사한 주소`

7. 클론된 프로젝트로 이동해서 파일들이 잘 복사되었는지 확인

   `cd 프로젝트명`

   `ll`

8. 코드들이 잘 수행되는지 테스트로 검증

   `./gradlew test`

   -> permission denied 발생했다.. (실행 권한 없음)

   `chmod +x ./gradlew` 명령어로 실행 권한 추가하고 다시 테스트 수행하면 됨.

   그런데 이번에는 테스트가 진행되다가 먹통이 되는 현상이 발생했다.

   아래 링크를 통해 해결 방법을 확인할 수 있다.

   [ec2 프리티어 메모리 부족 현상 해결 방법](<https://github.com/dongdong8343/TIL/blob/main/(%EB%8F%84%EC%84%9C)%20%EC%8A%A4%ED%94%84%EB%A7%81%20%EB%B6%80%ED%8A%B8%EC%99%80%20AWS%EB%A1%9C%20%ED%98%BC%EC%9E%90%20%EA%B5%AC%ED%98%84%ED%95%98%EB%8A%94%20%EC%9B%B9%20%EC%84%9C%EB%B9%84%EC%8A%A4/chapter8/ec2%20%ED%94%84%EB%A6%AC%ED%8B%B0%EC%96%B4%20%EB%A9%94%EB%AA%A8%EB%A6%AC%20%EB%B6%80%EC%A1%B1%20%ED%98%84%EC%83%81.md>)

## 배포 스크립트 만들기

1. deploy.sh 파일 생성

   매번 배포할 때마다 개발자가 명령어를 하나하나 치는건 매우 귀찮다.

   그래서 이를 스크립트로 작성해 해당 스크립트를 실행해서 과정이 순서대로 진행되도록 할 수 있다.

   `vim ~/app/step1/deploy.sh`

2. 다음 코드 추가

```
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step1
PROJECT_NAME=SpringBootStudy // 수정 필요! 내 프로젝트 이름으로

cd $REPOSITORY/$PROJECT_NAME/

echo "> Git Pull"

git pull

echo "> 프로젝트 Build 시작"

./gradlew build

echo "> step1 디렉토리로 이동"

cd $REPOSITORY

echo "> Build 파일 복사"

cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

echo "> 현재 구독 중인 애플리케이션pid 확인"

CURRENT_PID=$(pgrep -f ${PROJECT_NAME}.*.jar)

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
        echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
        echo "> kill -15 $CURRENT_PID"
        kill -15 $CURRENT_PID
        sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
```

3. 스크립트에 실행 권한 추가

   `chmod +x ./deploy.sh`

4. 실행 권한 추가 확인

   `ll`

5. 스크립트 실행

   `./deploy.sh`

6. 로그 확인

   `vim nohup.out`

## 외부 시큐리티 파일 등록하기

1. app 디렉토리에 yml 파일 생성

`vim /home/ec2-user/app/application-oauth.yml`

2. 생성한 파일에 application-oauth.yml 파일 내용 그대로 복붙하기

   복붙하고 저장하고 나온다(:wq)

3. 방금 생성한 yml 파일을 사용하도록 deploy.sh 파일 수정

   ```
   nohup java -jar \
   -Dspring.config.location=classpath:/application.properties, /home/ec2-user/app/application-oauth.properties \
   $REPOSITORY/$JAR_NAME 2>&1 &
   ```

4. 다시 deploy.sh 실행

## 스프링부트 프로젝트로 RDS 접근하기

현재 MariaDB를 사용 중이다. MariaDB에서 스프링부트 프로젝트를 실행하기 위해서는 아래 작업들을 해줘야 한다.

1. 프로젝트 설정

   - mariadb 드라이버를 build.gradle에 등록

   `implementation 'org.mariadb.jdbc:mariadb-java-client'`

   - application-real.yml 파일 작성

   실제 운영될 환경이기 때문에 보안/로그상 이슈가 될 만한 설정들을 모두 제거

   작성 후 push함

   ```yml
   spring:
     profiles:
       include:
         - oauth
         - real-db

     jpa:
       properties:
       hibernate:
         dialect: org.hibernate.dialect.MySQLDialect

     session:
       store-type: jdbc
   ```

2. EC2 설정

   rds 접속 정보도 보호해야할 정보라서 ec2 서버에 설정 파일을 작성한다.

   app 디렉토리에 application-real-db.yml 파일 생성

   `vim ~/app/application-real-db.yml`

   ```yml
   spring:
   jpa:
     hibernate:
     ddl-auto: none
   datasource:
     url: jdbc:mariadb://DB주소:포트번호/DB이름
     username: db계정
     password: db비번
     driver-class-name: org.mariadb.jdbc.Driver
   ```

   deploy.sh가 real profile을 사용할 수 있도록 수정

   (참고) \는 줄바꿈을 의미함

   ```
    ...
    nohup java \
    -Dspring.profiles.active=real \
    -Dspring.config.location="classpath:/application.yml,file:/home/ec2-user/app/application-oauth.yml,file:/home/ec2-user/app/application-real-db.yml,classpath:/application-real.yml" \
    -jar "$REPOSITORY/$JAR_NAME" \
    > "$REPOSITORY/app.log" 2>&1 &
   ```

## EC2에서 소셜 로그인하기

1. EC2에 8080포트로 배포됐으니 8080 포트가 보안 그룹에 열려있는지 확인

<center>
  <img
    src="https://github.com/user-attachments/assets/ee46b7a7-5b9d-4a95-b766-1594feae2581"
    width="50%"
  />
</center>

2. AWS EC2 도메인으로 접속

   EC2의 퍼블릭 DNS를 복사해서 :8080을 붙여 브라우저로 접속해본다. 그럼 접속이 잘되는것을 확인할 수 있다.

    <center>
        <img
            src="https://github.com/user-attachments/assets/6ba72ea7-7e42-40d8-aa74-2558608c5535"
            width="50%"
        />
    </center>

   그러나 현재 구글과 네이버 로그인은 동작하지 않는다. 그 이유는 해당 서비스에 EC2의 도메인을 등록하지 않았기 때문이다.

3. 구글에 EC2 주소 등록

   구글 API 콘솔로 접속 > 브랜딩 > 승인된 도메인에 EC2의 퍼블릭 DNS 등록

   퍼블릭 DNS를 등록하려고 하니까 잘못된 도메인이라고 뜨는 것이다.. 그래서 아래 링크에 해결 방법을 작성해놨다.

   [EC2와 구매한 도메인 연결](https://github.com/dongdong8343/TIL/blob/main/aws/EC2%EC%99%80%20%EA%B5%AC%EB%A7%A4%ED%95%9C%20%EB%8F%84%EB%A9%94%EC%9D%B8%20%EC%97%B0%EA%B2%B0.md)

   그리고 아래 사진에 구매한 도메인을 입력해주면 된다.

    <center>
        <img
            src="https://github.com/user-attachments/assets/aaa0cbac-0ee5-4bb7-bfd5-2a1dfaf52a30"
            width="50%"
        />
    </center>

   프로젝트 > 사용자 인증정보 > 프로젝트 클릭 > 승인된 리디렉션 URI에 추가

   `http://구매한 도메인/login/oauth2/code/google`

    <center>
        <img
            src="https://github.com/user-attachments/assets/7e5b9b36-bfe2-4a35-95c8-c90699b17040"
            width="50%"
        />
    </center>

4. 네이버에 EC2 주소 등록

   네이버 개발자 센터 > 내 프로젝트 > API 설정 > 서비스 URL, Callback URL 수정

   퍼블릭 DNS를 넣어주면 됨

   Callback URL은 `퍼블릭 DNS:8080/login/oauth2/code/naver` 붙여준다.

<center>
     <img
        src="https://github.com/user-attachments/assets/af2391fa-ba53-46f1-bb63-9253cf0fba7a"
        width="50%"
     />
</center>
