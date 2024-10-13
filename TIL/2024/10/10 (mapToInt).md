10/10 TIL

### 1 mapToInt vs. map :
- map의 경우 Stream\<T\>로 받기 때문에 Stream\<Integer\>로 래퍼 클래스에 결과 값을 받게 된다. 그래서 힙영역을 사용하게 되고 GC를 통해 제거된다.
- 이에 반해 mapToInt는 int 기본형으로 받아서 스택프레임에 저장되기에 최종 연산 후 바로 폐기 될 수 있어 mapToInt를 사용하면 힙영역에 부담을 줄일 수 있다. (특히, 양이 많을 수록)
