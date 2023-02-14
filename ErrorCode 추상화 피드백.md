# ErrorCode (수정전)
```
@Getter
public enum ErrorCode {

    // common
    INVALID_INPUT_VALUE(400, "C001", " Invalid Input Value"),
    METHOD_NOT_ALLOWED(405, "C002", " Method Not Allowed"),
    HANDLE_ACCESS_DENIED(403, "C006", "Access is Denied"),

    // user
    USER_NOT_FOUND(400, "U001", "User Not Found!"),

    // book
    BOOK_NOT_FOUND(400, "B001", "Book Not Found!"),
    BOOK_ALREADY_REGISTER(400, "B002", "Book Already Register!"),
    IS_NOT_MY_BOOK(400, "B003", "Is Not My Book!");

    private final String code;
    private final String message;
    private int status;

    ErrorCode(final int status, final String code, final String message) {
        this.status = status;
        this.message = message;
        this.code = code;
    }
}
```
- 사용자 에러 + 공통 에러를 모두 ErrorCode enum에 정의하고 사용했었다.
### 문제
- 위 코드처럼 관리하면 에러코드가 점점 늘어날 경우 ErrorCode enum이 너무 비대해진다.
  
# ErrorCode (수정후)
```
public interface ErrorCode {

    String getMessage();
    int getStatus();
    String getCode();
}
```

```
@Getter
@RequiredArgsConstructor
public enum CommonErrorCode implements ErrorCode {

    INVALID_INPUT_VALUE(400, "C001", " Invalid Input Value"),
    METHOD_NOT_ALLOWED(405, "C002", " Method Not Allowed"),
    HANDLE_ACCESS_DENIED(403, "C006", "Access is Denied");

    private final int status;
    private final String code;
    private final String message;
}
```

```
@Getter
@RequiredArgsConstructor
public enum UserErrorCode implements ErrorCode {

    USER_NOT_FOUND(400, "U001", "User Not Found!");

    private final int status;
    private final String code;
    private final String message;
}
```

- ErrorCode를 인터페이스로 만들고 이를 구현하는 CommonErrorCode, UserErrorCode로 세분화하였다. 이렇게 함으로서 하나의 클래스가 비대해지는걸 피할 수 있고, 확장에도 용이하게 되었다.

[참고사이트] https://mangkyu.tistory.com/205
