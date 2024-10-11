- MockedStatic을 쓴다면 
  - 메소드가 끝나기 전까지 해제는 안되지만 편리하기는 이렇게 쓰면 편하다.
  ```java
   private static MockedStatic<MemberDto> memberDto;

    @BeforeEach
    static void setUp() {
        memberDto = mockStatic(MemberDto.class);
    }

    @AfterEach
    static void tearDown() {
        memberDto.close();
    }
  ```
  - 그렇지만 범위를 좀 더 제한하고 싶다면 사용 후 바로 해제 되는 try-with-resources를 사용하자.
  ```java
  try(MockedStatic<MemberDto> memberDto = mockStatic(MemberDto.class)){
    when(MemberDto.from(any(Member.class))).thenReturn(updatingMember);
  }
  ```
