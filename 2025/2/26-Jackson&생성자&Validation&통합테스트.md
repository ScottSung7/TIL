#### 1. Jackson은 역직렬화시 기본적으로 기본생성자를 이용한 리플랙션 방식으로 객체를 생성한다.
  - 그렇기에 생성자에 로직이 있다면 사용되지 않는다.
  - 만약 생성자의 로직을 사용하고 싶다면 @Jacksonized를 사용 할 수 있다.
    - 이 어노테이션은 빌더를 이용해서 역직렬화하기에 @Builder가 필요하다.
    - 이 기능 사용시 기본생성자가 있지 않아도 된다.
    - 이 기능이 불변 객체를 만들기 더 편하다.
    - 그러나 롬복 버전 체크가 필요하다.
  - @Jacksonized를 사용할수 없다면 @JsonCreator과 함께 @JsonProperty를 이용해야 생성자로 생성할 수 있다.

#### 2. 스프링 Validation은 "생성"후에 일어난다
  - 그렇기에 생성자의 로직 중에 NPE등 다른 예외가 나온다면 Validation의 MethodArgumentNotValid가 아니라 다른 예외가 먼저 터진다.
  - 어차피 validation으로 널체크가 된다면 필요시 try catch로 NPE만 캐치해 놓을 수 있다.

### 3. @CachePut에서도 Jackson을 이용한다.
  - Optional을 사용한다면 직렬화 지원하지 않아 않아서 직렬화 되지 않는다.
  - SpEL을 사용하거나 Jackson의 Optional지원 모듈을 사용할 수 있다.
```java
@Configuration
public class CacheConfig {
    @Bean
    public RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer(objectMapper())
                )
            );
    }
    
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new Jdk8Module()); // Optional 지원
        return mapper;
    }
}
```

    
#### 4. 통합테스트의 필요성
  - 단위 테스트 위주로 작성을 하면 모킹 때문에 flase negative 가능성이 있다.
  - PostMan으로 테스트 중 단위 테스트는 통과 했는데 실제 요청에서 NPE가 나오는 경우가 있었다.
  - 이는 디비의 로직이 모킹으로 가려져서 디비의 실제 동작은 달랐던것.
    - 그렇기에 대부분 단위 테스트로 커버가 되지만 주요 시나리오를 따라가는 통합 테스트도 꼭 필요하다.
```java
 @Test
    void findById() {
        // Given
        Long id = 1L;
        CheckIn checkIn = CheckInFixtures.CheckInT.create();
        CheckInEntity sut = CheckInFixtures.CheckInEntityT.create(checkIn);
        when(checkInSpringJPARepository.findById(id)).thenReturn(Optional.of(sut));
        //fixme: 원래 이 테스트에서 에러가 났어야 하는데 실제 디비랑 다르게 스터빙 되어있다 보니 에러가 안 났다.

```
