- 역직렬화시 OBJECT_AND_NON_CONCRETE로 객체와 추상클래스 @class로 사용가능. 
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
