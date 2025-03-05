애플리케이션을 만들다보면 여러 곳에서 예외가 발생할 수 있다. 이 예외들을 한 곳에서 관리하고 처리할 수 있다면 얼마나 좋을까?

그래서 오늘은 GlobalExceptionHandler를 만들어보면서 만드는 과정을 기록에 남기려고한다.

GlobalExceptionHandler를 만들면 유지보수하기도 용이하고 어떤 예외가 발생했을 때 처리하는 코드를 동일하게 작성을 안해도된다는 장점이 있다.

그럼 만드는 순서를 알아보자.

## **1. ErrorCode 인터페이스 생성**

```java
public interface ErrorCode {
    String name();

    HttpStatus getHttpStatus();

    String getMessage();
}

```

특정 상황에서 발생하는 에러 코드를 도메인 별로 일관되게 관리할 수 있도록 interface를 통해서 생성했다.

예를 들면 인증 관련 에러 코드, 결제 관련 에러 코드 이런 식으로 도메인 별로 에러를 관리하고 싶을 때 해당 인터페이스를 상속받아서 도메인에 맞게 에러를 관리할 수 있게된다.

## 2. ErrorCode를 상속받은 Enum 생성

```java
@Getter
@RequiredArgsConstructor
public enum CommonErrorCode implements ErrorCode {
    INVALID_PARAMETER(HttpStatus.BAD_REQUEST, "Invalid parameter included"),
    RESOURCE_NOT_FOUND(HttpStatus.NOT_FOUND, "Resource not exists"),
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "Internal server error"),
    ;

    private final HttpStatus httpStatus;
    private final String message;
}
```

공통적으로 발생하는 에러코드를 enum으로 관리해서 에러가 발생할 때마다 새롭게 객체를 생성해서 관리를 안해도 된다.

그리고 위 코드와 마찬가지로 도메인 별로 enum을 만들어서 관리하면 에러를 관리하기 깔끔해진다.

예를 들면 결제 관련 에러를 관리하고 싶을 때 아래와 같이 작성할 수 있다.

```java
@Getter
@RequiredArgsConstructor
public enum PaymentErrorCode implements ErrorCode {
    PAYMENT_FAILED(HttpStatus.BAD_REQUEST, "Payment processing failed"),
    INSUFFICIENT_FUNDS(HttpStatus.BAD_REQUEST, "Insufficient funds");

    private final HttpStatus httpStatus;
    private final String message;
}
```

## 3. 사용자 정의 예외 만들기

```java
@Getter
@RequiredArgsConstructor
public class PostNotFoundException extends RuntimeException{
    private final ErrorCode errorCode;
}
```

사용자 정의 예외를 만드는 이유는 예외가 발생했을 때 그 예외가 구체적으로 왜 발생했는지 클래스 이름으로 알 수 있기 때문이다.

즉 오류 위치를 직관적으로 바로 알 수 있다는 장점이 있다.

## 4. 클라이언트에게 반환할 JSON 응답 형식 정의

```java
@Getter
public class ErrorResponse {
    private final String code;
    private final String message;

    @JsonInclude(JsonInclude.Include.NON_EMPTY) // error 비었다면 errors는 제외해라
    private final List<ValidationError> errors;

    private ErrorResponse(String code, String message, List<ValidationError> errors) {
        this.code = code;
        this.message = message;
        this.errors = errors;
    }

    public static ErrorResponse of(String code, String message) {
        return new ErrorResponse(code, message, null);
    }

    @Getter
    public static class ValidationError { // 유효성 검사 실패 시 발생하는 개별 필드의 오류 저장
        private final String field;
        private final String message;

        private ValidationError(String field, String message) {
            this.field = field;
            this.message = message;
        }

        public static ValidationError of(FieldError fieldError) {
            return new ValidationError(fieldError.getField(), fieldError.getDefaultMessage());
        }
    }
}
```

## 5. GlobalExceptionHandler 작성

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    // PostNotFoundException 처리
    @ExceptionHandler(PostNotFoundException.class)
    public ResponseEntity<Object> handleCustomException(PostNotFoundException e) {
        return handleExceptionInternal(e.getErrorCode());
    }

    // IllegalArgumentException 에러 처리
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Object> handleIllegalArgument(IllegalArgumentException e) {
        return handleExceptionInternal(CommonErrorCode.INVALID_PARAMETER, e.getMessage());
    }


    // 대부분의 에러 처리
    @ExceptionHandler({Exception.class})
    public ResponseEntity<Object> handleAllException(Exception ex) {
        return handleExceptionInternal(CommonErrorCode.INTERNAL_SERVER_ERROR);
    }

    // RuntimeException과 대부분의 에러 처리 메세지를 보내기 위한 메소드
    private ResponseEntity<Object> handleExceptionInternal(ErrorCode errorCode) {
        return ResponseEntity.status(errorCode.getHttpStatus())
                .body(makeErrorResponse(errorCode));
    }

    // 코드 가독성을 위해 에러 처리 메세지를 만드는 메소드 분리
    private ErrorResponse makeErrorResponse(ErrorCode errorCode) {
        return ErrorResponse.of(errorCode.name(), errorCode.getMessage());
    }

    private ResponseEntity<Object> handleExceptionInternal(ErrorCode errorCode, String message) {
        return ResponseEntity.status(errorCode.getHttpStatus())
                .body(makeErrorResponse(errorCode, message));
    }

    // 코드 가독성을 위해 에러 처리 메세지를 만드는 메소드 분리
    private ErrorResponse makeErrorResponse(ErrorCode errorCode, String message) {
        return ErrorResponse.of(errorCode.name(), errorCode.getMessage());
    }
}
```

@ExceptionHandler 어노테이션을 붙이면 특정 예외가 발생했을 때 예외에 해당하는 @ExceptionHandler가 붙은 메서드가 예외를 처리하게 된다.

@ExceptionHandler 어노테이션을 @RestControllerAdvice 어노테이션이 붙은 GlobalExceptionHandler에서 사용하면 예외를 한 곳에서 처리할 수 있다.

그리고 @RestControllerAdvice 어노테이션이 붙어서 응답은 JSON으로 해준다.

## 6. 내가 정의한 예외를 던지도록 Provider 계층의 코드 수정

```java
@Component
@RequiredArgsConstructor
public class PostProvider {
    private final PostsRepository postsRepository;

    public Post searchPost(Long id) {
        return postsRepository.findByIdAndIsDeletedIsNull(id).orElseThrow(() -> new PostNotFoundException(CommonErrorCode.INVALID_PARAMETER));
    }

    ...
}

```
