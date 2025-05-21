# Uncommited Read
- 한 트랜잭션이 쓰기 이후 커밋하지 않은 데이터를 다른 트랜잭션이 읽기 가능
- Read View를 만들지 않음

# Commited Read
- 한 트랜잭션이 쓰기한 데이터는 커밋 후에야 다른 트랜잭션이 읽기 가능
- Dirty Read 방지
- 동작 방식
  - select 시점마다 Read View 생성
  - 이 행을 읽는 트랜잭션의 ID가 행의 trx_id 보다 작은가
    - 작다: 행의 데이터를 읽는다
    - 크다: undo 로그에 있는 데이터를 읽는다

# Repeatable Read
- 한 트랜잭션에서 동일 읽기를 할 때 같은 결과가 나오게 보장
- Dirty Read/Non Repeatable Read 방지
- 동작 방식
  - 데이터를 처음 읽을 때 Read View를 생성, 이를 트랜잭션 끝날떄까지 사용
- InnoDB 엔진 특수 케이스: 락을 명시 하지 않아도 insert 을 막는 경우
  - InnoDB 엔진은 Repeaable Read level의 트랜잭션에서 인덱스를 사용한 컬럼에 범위 조건을 걸어 조회를 할 경우 해당 인덱스에 범위 조건 만큼 gap lock이 걸린다

# Serialize
- 한 트랜잭션이 커멧 될 동안 다른 트랜잭션은 대기
- Dirty Read/Non Repeatable Read/Phantom Read 방지
- 동작 원리
  - two phase locking
    - 잠금 단계에는 잠그기만, 해제 단계에는 해제만 한다
      - 트랜잭션 진행중에는 잠금 단계, 트랜잭션 종료 직전에 해제 단계를 수행
    - 읽기 작업시 공유락, 쓰기 작업시 배타락을 건다

