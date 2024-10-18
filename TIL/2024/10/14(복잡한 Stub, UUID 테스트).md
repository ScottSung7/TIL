### 1. 복잡한 Stub
쿼리를 컬랙션으로 만들어야 하는 Stub 클래스의 특성상 복잡한 쿼리의 경우 구현이 까다로울수 있다. 이럴때 시간의 제약이 있다면 Mockito Stubbing을 쓰는 것이 좋을 것 같다. 

### 2. UUID 테스트
- UUID인지 테스트를 UUID.fromString()을 통해 IllegalArgumentException의 여부에 따라 검사했지만, Regex를 통해서 검사하면 Exception을 일부러 내지 않고 테스트를 할 수 있다.
```java

public class UUIDTester {

    static Pattern UUID_REGEX =
            Pattern.compile("^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}$");

    public static boolean isUUID(String uuid) {
        return UUID_REGEX.matcher(uuid).matches();
    }
}


```
