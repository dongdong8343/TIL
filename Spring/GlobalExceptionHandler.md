애플리케이션을 만들다보면 여러 곳에서 예외가 발생할 수 있다. 이 예외들을 한 곳에서 관리하고 처리할 수 있다면 얼마나 좋을까?

그리고 예외 이름만 봐서는 왜 예외가 발생했는지 쉽게 파악하기 어렵다는 단점이 있다.

그래서 오늘은 GlobalExceptionHandler를 만들어보면서 위에서 말한 단점들을 해결해보려고 한다.

GlobalExceptionHandler를 만들면 유지보수하기 용이해진다.

그럼 만드는 순서를 알아보자.

패키지 구조는 아래와 같다.

<center>
  <img
    src="https://github.com/user-attachments/assets/0be72a56-47fe-49dd-a5ce-e62a42be56cc"
    width="50%"
  />
</center>

## 1. config - error - ErrorCode enum 파일 생성

```java
@Getter
@RequiredArgsConstructor
public enum ErrorCode {
    INVALID_INPUT_VALUE(HttpStatus.BAD_REQUEST, "E1", "올바르지 않은 입력값입니다."),
    METHOD_NOT_ALLOWED(HttpStatus.METHOD_NOT_ALLOWED, "E2", "잘못된 HTTP 메서드를 호출했습니다."),
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "E3", "서버 에러가 발생했습니다."),
    NOT_FOUND(HttpStatus.NOT_FOUND, "E4", "존재하지 않는 엔티티입니다."),

    POST_NOT_FOUND(HttpStatus.NOT_FOUND, "A1", "존재하지 않는 게시글입니다.");

    private final HttpStatus status;
    private final String code;
    private final String message;
}
```

발생할 수 있는 예외 메시지들을 한 곳에서 관리할 수 있다.

하나의 상태 코드가 여러 상황에서 발생할 수 있다. 그래서 어떤 상황에서 발생한 것인지 에러 코드를 정의해서 넣으면 어떤 경우에서 발생한 예외인지 쉽게 파악할 수 있다는 장점이 있다.

## 2. ErrorResponse.java 파일 생성

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ErrorResponse {
    private String message;
    private String code;

    private ErrorResponse(final ErrorCode code) {
        this.message = code.getMessage();
        this.code = code.getCode();
    }

    private ErrorResponse(final ErrorCode code, final String message) {
        this.message = message;
        this.code = code.getCode();
    }

    public static ErrorResponse of(final ErrorCode code) {
        return new ErrorResponse(code);
    }

    public static ErrorResponse of(final ErrorCode code, final String message) {
        return new ErrorResponse(code, message);
    }
}
```

에러 메시지와 에러 코드를 반환하는 객체를 만들어서 예외가 발생했을 때 JSON 형태로 응답을 받게 된다.

```
{
  "message": "존재하지 않는 게시글입니다.",
  "code": "A1"
}
```

## 3. error - exception - BusinessBaseException.java 파일 생성

```java
public class BusinessBaseException extends RuntimeException {
    private final ErrorCode errorCode;

    public BusinessBaseException(String message, ErrorCode errorCode) {
        super(message);
        this.errorCode = errorCode;
    }

    public BusinessBaseException(ErrorCode code) {
        super(code.getMessage());
        this.errorCode = code;
    }

    public ErrorCode getErrorCode() {
        return errorCode;
    }
}

```

이 예외 클래스는 비즈니스 로직을 작성하다가 발생할 수 있는 예외들의 최상위 클래스다. 그래서 나중에 비즈니스 로직 중에 예외가 발생하면 메서드 하나만으로 다 처리할 수 있게된다. 이 부분은 이따가 확인할 수 있다.

그냥 RuntimeException을 사용하면 안되냐?라는 질문을 할 수 있다.

RuntimeException만을 사용해서 비즈니스 로직에서 발생하는 예외들을 처리하게되면 어떤 에러 메시지인지만 가지게되고 어떤 상황에서 발생한 예외인지 알려주지 못하는 상황이 발생한다.

하지만 RuntimeException을 상속받은 BusinessBaseException을 만들어서 ErrorCode 필드를 넣어주면 어떤 상황에서 발생한 예외인지 다룰 수 있게 된다.

## 4. exception - 발생 가능한 예외 작성하기

```java
public class NotFoundException extends BusinessBaseException {
    public NotFoundException(ErrorCode errorCode) {
        super(errorCode);
    }

    public NotFoundException() {
        super(ErrorCode.NOT_FOUND);
    }
}
```

```java
public class PostNotFoundException extends BusinessBaseException {
    public PostNotFoundException() {
        super(ErrorCode.POST_NOT_FOUND);
    }
}
```

비즈니스 로직 중에 발생가능 한 예외들은 BusinessBaseException을 상속 받게해서 작성한다.

## 5. error - GlobalExceptionHandler 작성

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    // protected로 접근 제한자를 한 이유 - 외부에서 호출하는 것을 막기 위해서
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    protected ResponseEntity<ErrorResponse> handle(HttpRequestMethodNotSupportedException e) {
        return createErrorResponseEntity(ErrorCode.METHOD_NOT_ALLOWED);
    }

    @ExceptionHandler(BusinessBaseException.class)
    protected ResponseEntity<ErrorResponse> handle(BusinessBaseException e) {
        return createErrorResponseEntity(e.getErrorCode());
    }

    @ExceptionHandler(Exception.class)
    protected ResponseEntity<ErrorResponse> handle(Exception e) {
        return createErrorResponseEntity(ErrorCode.INTERNAL_SERVER_ERROR);
    }

    private ResponseEntity<ErrorResponse> createErrorResponseEntity(ErrorCode errorCode) {
        return new ResponseEntity<>(
                ErrorResponse.of(errorCode),
                errorCode.getStatus());
    }
}
```

@ExceptionHandler 어노테이션을 붙이면 특정 예외가 발생했을 때 예외에 해당하는 @ExceptionHandler가 붙은 메서드가 예외를 처리하게 된다.

@ExceptionHandler 어노테이션을 @RestControllerAdvice 어노테이션이 붙은 GlobalExceptionHandler에서 사용하면 예외를 한 곳에서 처리할 수 있다.

그리고 BusinessBaseException을 상속받아 비즈니스 로직 중에 발생하는 예외를 만들어놔서 위 코드를 보면 @ExceptionHandler(BusinessBaseException.class)라고 작성을 함으로써 비즈니스 로직 중에 발생하는 예외를 이 메서드에서 다 처리할 수 있게됐다.

@RestControllerAdvice을 붙이면 전역에서 발생하는 예외를 해당 클래스에서 다 처리할 수 있게된다.

그리고 내부를 보면 아래와 같이 작성이 됐다.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ControllerAdvice
@ResponseBody
public @interface RestControllerAdvice {
    ...
}
```

@ResponseBody가 있어서 Http 본문 응답 객체를 반환할 수도 있다.

## 6. 내가 정의한 예외를 던지도록 Provider 계층의 코드 수정

```java
@Component
@RequiredArgsConstructor
public class PostProvider {
    private final PostsRepository postsRepository;

    @Transactional(readOnly = true)
    public Post searchPost(Long id) {
        return postsRepository.findByIdAndIsDeletedIsNull(id).orElseThrow(PostNotFoundException::new);
    }

    ...
}
```
