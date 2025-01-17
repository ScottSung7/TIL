### 1. Request Fixture
Request Fixture에서 테스트를 위해 추가되는 @Builder와 @AllArgsConstructors 가 신경 쓰였다. 테스트와 비즈니스 코드는 분리 되어야 한다. 그래서 Request의 필드값을 상수로 만들고 @RequiredArgConstructors를 붙이는게 좋다고 생각했다. Request는 값의 변동이 없어야 되기에 상수가 더 어울리고 필요한 생성자만 넣어주게 되니까 더 좋은것같다.
```java
@RequiredArgsConstructor(access = AccessLevel.PROTECTED)
    public static class SignUpRequest {

        @Email(message = EMAIL_FORMAT_VALIDATION_MESSAGE)
        @NotBlank(message = EMAIL_VALIDATION_MESSAGE)
        private final String email;
```
