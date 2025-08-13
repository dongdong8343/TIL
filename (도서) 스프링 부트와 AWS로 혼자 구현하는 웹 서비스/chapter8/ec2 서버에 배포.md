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

nohup java -jar \
	-Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties,classpath:/application-real.properties
	-Dspring.profiles.active=real \
	$REPOSITORY/$JAR_NAME 2>&1 &
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

1.
