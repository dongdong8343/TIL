### 내장 was

was는 web application server를 의미한다. 스프링 부트는 기본적으로 내장 was를 사용한다.

```java
implementation 'org.springframework.boot:spring-boot-starter-web'
```

해당 라이브러리에 이미 톰캣이라는 내장 was가 포함되어 있다.

was는 http 요청을 받아서 스프링 부트 코드를 실행시키고 http 응답을 보내는 역할을 한다.

내장 was는 jvm 위에서 스프링부트 애플리케이션이 같이 동작한다.
