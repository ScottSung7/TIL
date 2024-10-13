
### 1. Mock으로 만든 stub 과 실제 동작이 다른 경우나 나타났다. DB를 이용할 때는 UUID를 자동 추가 하는데 내가 해준 stub 은 그렇지 않았다. 매번 모킹 stub 동작을 만들어야 하다 보니 내가 만든 stub이 실제 동작과 다를 경우 False Positive 에 취약해 진다는 생각이 들었다.
이 외에도 내부 ID값을 UUID로 변경후 아이디는 UUID임을 확인해야 하지만 일반 스트링인데도 통과하는 경우들이 있었다. 의도한 것도 있었으나, 테스트가 현실과 다르게 동작해서 통과를 할 수 있다는건 테스트가 제대로 알림 역할을 못하고 있다고 생각한다. 
그래서 "Stub 클래스"를 만들어 사용하는 것이 **반복되는 같은 모킹 stub 제작에서 실수를 할 가능성을 더 줄여 준다는 생각**이다.

``` java
    @Test
    @DisplayName("회원가입 - 성공.")
    void save_Mock() {
        //given
        Member member = MemberFixture.create();
        MemberSpringJPARepository memberSpringJPARepository = mock(MemberSpringJPARepository.class);
        sut = new MemberJPARepository(memberSpringJPARepository);
        given(memberSpringJPARepository.findByEmail(member.getEmail())).willReturn(Optional.empty());
        given(memberSpringJPARepository.save(member)).willReturn(member);
        //given(memberSpringJPARepository.save(member)).willReturn(MemberFixture.createMemberWithId(UUID.randomUUID().toString()));
    
        //when
        String id = sut.save(member);
    
        //then
        assertNull(member.getId());
        assertNotNull(id);
        UUID.fromString(id);
    }

```

  ### 2. 그렇지만 Stub 클래스도 완벽하지는 않은게, 메서드마다 stubbing 해주다가 실수를 하는 것은 줄여주어도 Stub 클래스가 잘못되어 있다면 또 테스트가 의도 대로 동작하지 않을 수 있다. 그래서 생각한것은 Stub 클래스를 통합 테스트 하는 것이다. 
  조금 번거로워도 이렇게 하니 Stub 클래스를 사용하니 예전에 도메인이나 응용 계층에서 테스트 할 때, **들어가는 값이 정확할 지 염려되어 다시 통합 테스트를 해보는 경우가 있었는데 Stub자체를 믿을 수 있으니까 그럴 필요가 없어졌다.**
  Stub 클래스의 테스트는 자체도 구현이 어렵지 않고 컴파일때는 Disabled 해 놓아도 된다. 다른 테스트에 사용되는 Stub 클래스는 내부 컬렉션으로 구현 하기에 컴파일에 부담도 적다. 
``` java
   Set<Member> members = new HashSet<>();

    @Override
    public <S extends Member> S save(S entity) {
        UUID uuid = UUID.randomUUID();
        Member member = MemberFixture.createMemberToSave(uuid.toString(), entity);
        members.add(member);
        return (S) member;
    }
```
  
  ``` java
        @Test
        @DisplayName("save - 성공.")
        void save_Success() {
            //given
            Member member = MemberFixture.create();

            //when
            Member dbResult = memberSpringJPARepository.save(member);
            Member stubResult = stub.save(member);

            //then
            UUID dbId = UUID.fromString(dbResult.getId());
            UUID stubId = UUID.fromString(stubResult.getId());
            assertThat(dbResult.getEmail()).isEqualTo(stubResult.getEmail());
            assertThat(dbResult.getName()).isEqualTo(stubResult.getName());
            assertThat(dbResult.getPhone()).isEqualTo(stubResult.getPhone());
            assertThat(dbResult.getLocation()).isEqualTo(stubResult.getLocation());
            assertThat(dbResult.getPoint()).isEqualTo(stubResult.getPoint());
        }
  ```
