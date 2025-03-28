### Kafka

**기본**
- 파티션
  - 쉬운 확장성 / 병렬처리를 통해 처리량을 증가시키기와 부하 분산이 쉽다. 
  - 고가용성
- 키
  - 병렬처리: 키값 기반 파티셔닝을 통한 병렬처리
  - 순서보장 (오프셋) -> 파티션 사이는 아님. 전체의 경우 하나의 파티션을 사용해야 함.
- group id
  - 파티션의 오프셋 추적 (여러 토픽, 파티션 가능) // 병렬처리. 오프셋 관리.
- 리플리케이션 (2-3)
  - 복제할 리플리케이션 수 (ISR 그룹 -> In-Sync Replication)
  - Read/Write는 리더를 통해서 이루어진다. 리더에서 가져간다. 다운시 다른 팔로워가 리더가 된다.
  - 전부 다운 : 기다린다/ 빨리 올라온 팔로워가 리더.
- Producer
  - ack: 0(프로듀서 안 기다림), 1(리더 기록, 팔로워 확인 안함), all, -1 (팔로워로 부터 ack)
    - 0 시 리더 장애에 취약
- Consumer
  - 커밋 (누락, 중복)/ 자동 커밋 (5초) // 수동커밋이 더 안전.

- Retry 전략	- DLT 
  - 일괄적으로 적용하기 보다 Retrable한지 아닌지의 여부를 알고 적용하기. 재시도 불가능시 빠르게 DLT하는것도 좋다.
	-> retry하게 구현하려고 하면, 오히려 부하만 많아 질 수 있다.
  - Exponential, Random, Fixed. 
[Kafka Retry 정책](https://velog.io/@ce19f003/Kafka-Retry-%EC%A0%95%EC%B1%85-feat.-backoff-DLTDLQ)
[Consumer 코드](https://velog.io/@youmakemesmile/Spring-KafkaKafka-Consumer-%EC%A0%95%EB%A6%AC)
[get vs. join](https://luna-archive.tistory.com/32)

- Completable Future
	-  Blocking : get(checked ex.) / join(unchecked) 
	- Async 전략. <br>
[Completable Future in general](https://wbluke.tistory.com/50)

- MSA와 관련 있는 부분
  - 트랜잭션
    - 2PC 방지. (SAGA를 이용한 실패 이벤트 발행)
    [2PC](https://kadensungbincho.tistory.com/125)
    - Kafka. [Kafka-Transaction](https://incheol-jung.gitbook.io/docs/q-and-a/spring/transaction)
      - 1. @TransactionalEventListener
      - 2. @Transactional
      -> props.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, READ_COMMITTED);
      - 3. transaction-id-prefix (KafkaTransactionMager) isolation
      -> read committed로 변경 [db+](https://rudaks.tistory.com/entry/spring-kafka%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%A0-%EB%95%8C%EC%9D%98-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EC%B2%98%EB%A6%AC)
  - 서비스 매쉬
    - MS의 독립성 보장. -> 카프카를 통한 느슨한 결합
  - 장애처리
    - 서킷브레이킹, 타임아웃 (Retry 전략) <br>
[MSA 설계시 고려할 점](https://wso2.com/ko/whitepapers/microservices-in-practice-key-architectural-concepts-of-an-msa/#103)
