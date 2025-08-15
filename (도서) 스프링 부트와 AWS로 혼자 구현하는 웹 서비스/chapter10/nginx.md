네이버나 카카오톡을 보면 배포하는 동안 서비스가 중지되는 일이 없다.

배포하는 동안 서비스가 중지되면 사용자 입장에서 큰 불편을 느낄 수밖에 없다.

그래서 무중단배포를 하는 것이다.

## nginx

여기서는 엔진엑스를 사용한다. 가지고 있는 기능으로는 웹 서버, 리버스 프록시, 캐싱, 로드밸런싱 등이 있다.

그 중에서도 리버스 프록시를 통해 요청을 받고 해당 요청을 백엔드 서버로 전달하는 작업을 하게 만들 것이다.

무엇보다 중요한 점! 저렴하고 쉽다.

구조는 아래와 같다.

<center>
  <img
    src="https://github.com/user-attachments/assets/494a1782-c0c0-4c01-a691-08c006813c40"
    width="50%"
  />
</center>

여기서 스프링부트 Jar를 왜 2대나 사용하지? 라는 의문이 생길 수 있다.

이것이 바로 무중단 배포의 핵심이다. 한 곳은 기존 서버를 서비스하고 다른 한 곳에 배포를 해서 서비스를 끊기지않게 할 수 있는 것이다.

다른 한 곳에 배포가 끝나면 배포된 서버가 제대로 구동 중인지 확인한다.

확인하고 nginx reload 명령어를 통해 배포된 서버를 바라보도록 한다.(0.1초 내에 완료됨)

## 엔진엑스 설치와 스프링 부트 연동하기

1. EC2에 접속해 다음 명령어로 엔진엑스 설치

   `sudo yum install nginx`

2. 엔진엑스 실행

   `sudo service nginx start`

3. 보안 그룹 추가

   엔진엑스의 포트 번호는 80번이기 때문에 보안 그룹에 추가해준다.

   ec2 > 보안 그룹 > 보안 그룹 선택 > 인바운드 편집

   <center>
       <img
           src="https://github.com/user-attachments/assets/4c598c79-bb3a-498c-a875-032218330dd3"
           width="50%"
       />
   </center>

4. 리다이렉션 주소 추가

   기존 도메인으로 접근하되 8080 포트를 제거해준다.

   - 구글 리디렉션 주소 수정

   <center>
       <img
           src="https://github.com/user-attachments/assets/0bbdb15b-33b1-4bc3-8b85-11d052248e0c"
           width="50%"
       />
   </center>

   - 네이버 리디렉션 주소 수정

   <center>
       <img
           src="https://github.com/user-attachments/assets/a4605335-9add-4626-9957-bb0ac69945cd"
           width="50%"
       />
   </center>

5. 도메인 주소로 접속하면 아래와 같이 나오는 것을 확인할 수 있다.

<center>
    <img
        src="https://github.com/user-attachments/assets/b4d02d9b-d04f-40fb-a96b-70587ddc21b7"
        width="50%"
    />
</center>

6. 엔진엑스가 스프링 부트를 바라볼 수 있도록 프록시 설정

   `sudo vim /etc/nginx/nginx.conf`

   location / 부분을 찾아서 아래와 같이 추가한다.

   <center>
       <img
           src="https://github.com/user-attachments/assets/89ff8453-cdde-4b03-aa52-8c025b1ab73a"
           width="50%"
       />
   </center>

7. 엔진엑스 재시작 후 브라우저로 다시 접속하면 아래와 같이 잘 보이는 것을 확인할 수 있다.

   `sudo service nginx restart`

   <center>
       <img
           src="https://github.com/user-attachments/assets/415b6a1a-e5eb-4eb6-8c2d-b6e421e4ffc8"
           width="50%"
       />
   </center>
