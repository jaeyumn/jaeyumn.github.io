---
title: "[Spring] Global Custom Exception 적용 "
categories:
  - spring
---

## 개발 환경

- Java 17
- SpringBoot 3.2.1

<br>

## 도입 배경

`Custom Exception`을 적용하게 된 계기는 다음과 같습니다.

바로 서버에서 예외가 발생했을 때, 클라이언트에게 어떤 문제가 발생했는지 더 명확하게 전달할 수 있다는 점입니다.

예를 들어, 400 에러나 404 에러가 발생할 이유는 다양합니다. 회원이 존재하지 않다거나 회원 이메일이 존재하지 않다거나 등등..

이럴 때 `Custom Exception`을 사용하면 클라이언트는 에러 핸들링 하기에 용이하며, 이는 곧 사용자에게 세분화된 에러 메시지를 보여줄 수 있게 됩니다.

따라서 ErrorCode를 문서화하고 전역처리를 하게 된다면 생산성 및 유지보수성이 높아질 것으로 기대합니다.

<br>

## Global 하게 적용하기

처음에는 다음 코드처럼 모든 예외를 `custom exception class`로 만들고 도메인 별로 `ErrorHandler`를 만들었습니다.

```java
@Slf4j
@DomainSpecificAdvice
public class MemberErrorHandler {

    @ExceptionHandler(AlreadyExistsMemberException.class)
    public ResponseEntity<ErrorResponse> alreadyExistsMemberExceptionHandle(AlreadyExistsMemberException e) {
        log.error("{}", e.getMessage());

        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(ErrorResponse.createError(ErrorCode.ALREADY_EXISTS_MEMBER));
    }

    @ExceptionHandler(MemberNotFoundException.class)
    public ResponseEntity<ErrorResponse> memberNotFoundExceptionHandle(MemberNotFoundException e) {
        log.error("{}", e.getMessage());

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(ErrorResponse.createError(ErrorCode.MEMBER_NOT_FOUND));
    }

    @ExceptionHandler(PasswordMismatchException.class)
    public ResponseEntity<ErrorResponse> passwordMismatchExceptionHandle(PasswordMismatchException e) {
        log.error("{}", e.getMessage());

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(ErrorResponse.createError(ErrorCode.PASSWORD_MISMATCH));
    }

    @ExceptionHandler(EmailNotFoundException.class)
    public ResponseEntity<ErrorResponse> emailNotFoundExceptionHandle(EmailNotFoundException e) {
        log.error("{}", e.getMessage());

        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(ErrorResponse.createError(ErrorCode.EMAIL_NOT_FOUND));
    }

    @ExceptionHandler(AlreadyExistsEmailException.class)
    public ResponseEntity<ErrorResponse> alreadyExistsEmailExceptionHandle(AlreadyExistsEmailException e) {
        log.error("{}", e.getMessage());

        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(ErrorResponse.createError(ErrorCode.ALREADY_EXISTS_EMAIL));
    }
}

```

하지만 이렇게 작성하게 되니 exception 별로 클래스를 만들어야 하고, 패키지가 지저분해지는 단점이 보이게 되었습니다.

따라서 Exception을 공통으로 처리할 `RuntimeException`을 상속받은 `CustomApiException`이라는 클래스를 만들었습니다.

이제 예외가 발생하면 클라이언트는 다음과 같은 응답을 받을 수 있습니다.

```json
//status: 400Bad Request

{
    "errorCode": "MEMBER_002",
    "errorDescription": "이미 존재하는 회원입니다."
}
```

응답으로 보내질 `HttpStatus`는 `200 OK`로 고정하는 것이 아닌, `400 Bad Request` 등의 적절한 `status`를 적용했습니다.

<br>

## 구현

### ErrorCode

```java
@Getter
@AllArgsConstructor
public enum ErrorCode {
    // Member
    MEMBER_NOT_FOUND(HttpStatus.NOT_FOUND, "MEMBER_001", "존재하지 않는 회원입니다."),
    IS_EXISTS_MEMBER(HttpStatus.BAD_REQUEST, "MEMBER_002", "이미 존재하는 회원입니다."),
    ;

    private final HttpStatus httpStatus;
    private final String errorCode;
    private final String errorDescription;
}
```

Custom Error Code 입니다.

Enum으로 만들고 `HttpStatus`, `Custom Error Code`, `Error Description`을 정의했습니다.

<br>

### ErrorResponse

```java
public record ErrorResponse (String errorCode, String errorDescription) {

    public ErrorResponse(ErrorCode errorCode) {
        this(errorCode.getErrorCode(), errorCode.getErrorDescription());
    }
}

```

`ResponseBody`에 들어갈 데이터입니다.

`errorCode`와 `errorDescription`을 담았습니다.

<br>

### CustomApiException

```java
@Getter
public class CustomApiException extends RuntimeException {

    private final HttpStatus httpStatus;
    private final ErrorResponse errorResponse;

    public CustomApiException(ErrorCode errorCode) {
        this.httpStatus = errorCode.getHttpStatus();
        this.errorResponse = new ErrorResponse(errorCode);
    }
}

```

`RuntimeException`을 상속받은 `CustomApiException`입니다.

<br>

### GlobalRestControllerAdvice

```java
@Slf4j
@RestControllerAdvice
public class GlobalRestControllerAdvice {

    @ExceptionHandler(CustomApiException.class)
    public ResponseEntity<?> customExceptionHandle(CustomApiException e) {
        log.error("Error occurs {}", e.getErrorResponse());
        return ResponseEntity.status(e.getHttpStatus()).body(e.getErrorResponse());
    }
}

```

전역으로 `Custom Exception`을 받아서 처리할 클래스입니다.

<br>

### 사용 예시

```java
@RequiredArgsConstructor
@Service
public class MemberService {

    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;

    /**
     * 회원 가입
     */
    public void signUp(SignUpRequestDto requestDto) {
        String username = requestDto.username();
        if (memberRepository.isExistsMember(username)) {
            throw new CustomApiException(ErrorCode.IS_EXISTS_MEMBER);
        }

        memberRepository.save(
                Member.builder()
                        .username(username)
                        .password(requestDto.password())
                        .name(requestDto.name())
                        .passwordEncoder(passwordEncoder)
                        .build()
        );
    }
}
```

위 코드와 같이 `CustomApiException`을 던지고 적절한 `ErrorCode`를 인자로 담아줍니다.

```json
{
    "errorCode": "MEMBER_002",
    "errorDescription": "이미 존재하는 회원입니다."
}
```