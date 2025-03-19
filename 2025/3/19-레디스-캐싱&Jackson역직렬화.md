- 역직렬화시 OBJECT_AND_NON_CONCRETE로 객체와 추상클래스 @class로 사용가능.
   - [링크](https://isaac1102.github.io/2021/04/28/jackson) 
```java
   ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.activateDefaultTyping(
                BasicPolymorphicTypeValidator.builder()
                        .allowIfBaseType(Object.class)
                        .build(),
                ObjectMapper.DefaultTyping.OBJECT_AND_NON_CONCRETE,
                JsonTypeInfo.As.PROPERTY
        );
```
> 옵션
> - JAVA_LANG_OBJECT	선언된 유형으로서 객체가 포함된 프로퍼티(명시적 타입을 제외한 제네릭 타입을 포함한다.)
> 정보를 제공하겠다.
> - NON_CONCRETE_AND_ARRAYS	OBJECT_AND_NON_CONCRETE에 포함된 모든 타입의 프로퍼티와 이러한 요소의 배열 타입 정보를 제공하겠다.
> - NON_FINAL	JSON으로부터 올바르게 유추된 String, Boolean, Integer, Double을 제외한 모든 non-final 타입정보를 제공하겠다. 뿐만 아니라 non-final 타입의 모든 배열의 프로퍼티 정보도 제공하겠다.
> - OBJECT_AND_NON_CONCRETE	선언된 타입의 객체이거나 abstract 타입의 프로퍼티 정보를 제공하겠다.

- JsonTypeInfo를 통해 역직렬화할 클래스 정보를 넣어준다.
   - 다만, 이 방법 사용시 클래스 정보를 같이 저장해야하는 단점이 있다.
   - [링크1](https://velog.io/@youakdl12/JsonTypeInfo-JsonSubType), [링크2](https://alstn113.tistory.com/26)
```java
 @Getter
    @RequiredArgsConstructor
    @NoArgsConstructor(force = true)
    @JsonTypeInfo(use = JsonTypeInfo.Id.CLASS, include = JsonTypeInfo.As.PROPERTY, property = "@class")
    public static final class CheckInSelectDTO {
```

> 옵션
> @JsonTypeInfo
> - use : 다형성을 나타내는 정보를 어떻게 사용할지 지정
>   - JsonTypeInfo.Id.NAME : 타입 이름을 사용하여 다형성 처리
>    - JsonTypeInfo.Id.CLASS : 클래스 정보를 사용하여 다형성 처리
>   - JsonTypeInfo.Id.MINIMAL_CLASS : 최소한의 클래스 정보만 사용하여 다형성 처리. 클래스 이름을 속성으로 포함
>   - JsonTypeInfo.Id.NONE: 다형성 정보를 사용하지 않음.
> - include : 다형성 정보를 어떤 방식으로 포함할지 지정
>   - JsonTypeInfo.As.EXISTING_PROPERTY: 기존의 JSON 속성을 사용하여 다형성 정보 포함
>   - JsonTypeInfo.As.WRAPPER_OBJECT: 다형성 정보를 JSON 객체로 감싸서 포함
>   - JsonTypeInfo.As.WRAPPER_ARRAY: 다형성 정보를 JSON 배열로 감싸서 포함
>   - JsonTypeInfo.As.EXTERNAL_PROPERTY: 외부 속성에 다형성 정보를 포함
>   - JsonTypeInfo.As.PROPERTY: 다형성 정보를 JSON 속성으로 포함 (기본)
> - property : 다형성 정보를 포함할 JSON 속성의 이름을 지정
> - visible : 다형성 정보가 JSON에 노출되어야 하는지 여부를 지정(기본값은 true)
>
> @JsonSubTypes
> - value : 매핑할 하위 클래스를 지정
> - name : 해당 하위 클래스에 대한 이름 지정(생략가능)
