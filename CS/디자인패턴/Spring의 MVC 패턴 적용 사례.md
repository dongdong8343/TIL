## Spring의 MVC 패턴 적용 사례

- 자바 기반으로 애플리케이션 개발할 때 많은 기능들을 제공하는 프레임워크

<center>
    <img
            src="https://github.com/user-attachments/assets/e2fd6f82-17d7-4052-b76f-649501431fad"
            width="50%"
    />
</center>

1. 클라이언트가 요청하면 디스패처 서블릿이 이를 받는다.

   1. 컨트롤러의 @requestmapping을 참고함

2. 하나 이상의 handler mapping을 참고해서 적절한 컨트롤러 설정함. 이후 컨트롤러로 요청 보냄

3. 사용자에게 전달해야할 정보인 모델 생성

4. 뷰를 구현하기 위한 view resolver 참고

5. 해당 정보를 기반으로 뷰를 렌더링

6. 응답 데이터 보냄
