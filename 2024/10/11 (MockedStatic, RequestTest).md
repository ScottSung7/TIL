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

- Request는 테스트 하려면 이렇게 하나의 값씩 모킹하는 포맷이 개인적으로는 제일 좋은 것 같다.
  ```java
      private SignUpController.SignUpForm form;
  
      @BeforeEach
      void setUp() {
          form = spy(SignUpFormFixture.create());
          //mock이 아닌 값이 들어있는 Fixture를 이용해 spy를 사용해야 한다.
      }
  
      @Test
      @DisplayName("이메일 없음.")
      void signUp_NoEmail() throws Exception {
          //given
          when(form.getEmail()).thenReturn(null);
  
          //when
          ResultActions resultActions = mockMvc.perform(post(requestURL)
                  .contentType(MediaType.APPLICATION_JSON)
                  .content(mapper.writeValueAsString(form)));
  
          //then
          resultActions.andExpect(status().isBadRequest());
          verify(signUpService, times(0)).signUp(any(Member.class));
      }
  ```
