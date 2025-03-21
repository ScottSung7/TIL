10/9 TIL

### 1. Fixture를 만들었으면 값을 테스트하는 것이 좋다. 
  - Mock을 쓰는것에 익숙하다보니 값을 테스트 할 수 있는 상황에 verify로 구현 세부사항과 결합된 테스트를 써 버리는 경우가 있다. 
  - 구현 세부사항과 결합된 코드는 깨지기 쉽다.

### 2. (Q)Repository를 Stub 클래스로 만들어 사용하는 것이 좋을까 매번 Mockito로 stubbing 하는 것이 좋을까?
  - stub 클래스를 쓰면 테스트들에서 일관성 있는 결과를 받을 수 있고 재사용도 가능하지만, stub 클래스 코드를 잘못 작성 시 전체 테스트가 stub 통과를 위해 잘못된 방향으로 작성 될 수 있다. 
  - 초기에 stub 클래스를 integrated test로 테스트하고 사용하는 것도 좋을 것 같다.<br>
  -> 최악의 상황을 가정한 것이지만, 이 경우도 테스트에서 빼먹은 부분이나 오류가 stub에 그대로 반영 되어 실제 동작과 다를 수도 있다. <br>
  -> 그렇지만 mockito로 stubbing시 이 위험이 매 테스트 작성때 마다 있고, stub 클래스과 stub 클래스를 테스트하여 사용하면서 경험이 쌓이면 장기적으로는 stubbing이 잘못된 로직으로 되는 것이 적어지지 않을까?

### 3. Jackson라이브러리는 필드가 하나인 클래스를 기본 생성자 없이 역직렬화하지 못한다.
  - jackson-module-parameter-names이 기본 생성자 없어도 "인식한 필드가 들어갈만한 파라미터가 있는 생성자"가 있으면 역직렬화때 객체 생성 도와 주지만, 이때 필드가 하나면 안된다. ([링크1](https://velog.io/@happyjamy/Jackson-%EC%9D%B4-%EA%B0%9D%EC%B2%B4%EB%A5%BC-%EB%A7%8C%EB%93%9C%EB%8A%94-%EB%B2%95-InvalidDefinitionException))
  - inner static 클래스는 static 이기에 기본 생성자를 만들어 주지 않는다. <br>
  (즉, AllArg나 RequiredArg가 있어도 NoArg안 붙여주고 필드가 하나면 Jackson역직렬화에 실패한다.) 
  - 또한 Jackson라이브러리는 inner 클래스 사용때도 조심해야 한다. static이 없다면 Jackson이 초기화 할 수 있는 방법 없어서 역직렬화 실패한다. 
  ([링크 1](https://klyhyeon.tistory.com/299) , [링크 2](https://www.cowtowncoder.com/blog/archives/2010/08/entry_411.html))
  - inner 클래스 사용시 주의사항 ([static 선언](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9E%90%EB%B0%94%EC%9D%98-%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-static-%EC%9C%BC%EB%A1%9C-%EC%84%A0%EC%96%B8%ED%95%98%EC%9E%90))
