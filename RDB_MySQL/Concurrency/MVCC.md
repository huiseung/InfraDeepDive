# MVCC
- Multi Version Concurrency Control
- 여러 트랜잭션이 동시에 같은 데이터에 접근할 떄 발생할 수 있는 문제를 락 없이 해결할 수 있는 기술

## trx_id
- 어떤 행을 최종 수정한 트랜잭션 ID를 기록

## roll ptr
- 어떤 행의 변경 기록에 관한 undo log chain 자료구조를 가르키는 포인터

## Read View
- MVCC기반으로 트랜잭션이 데이터를 읽을 떄 사용하는 스냅샷
- trx_ids array
  - 아직 커밋 하지 않은 트랜잭션 ID 목록
- up limit id
  - trx_ids에서 가장 작은 값
  - Read View 생성 시점에 커밋된 트랜잭션 중 가장 큰 ID + 1
- low limit id
  - Read View 생성 시점 아직 할당되지 않은 트랜잭션 ID
- 데이터 읽기 가능 결정
  - 트랜잭션이 어떤 행을 읽으려 할 때 해당 행의 trx_id를 기준으로
  - trx_id < up_limit_id 이면 Read View 생성 이전 커밋된 trx_id 이므로 현재 테이블의 값 읽기
  - trx_id >= low_limit_id 이면 Read View 생성 이후 커밋된 trx_id 이므로 Undo Log Chain 값 읽기
  - trx_id가 trx_ids 에 포함되어 있을 경우, 아직 커밋되지 않고 진행 중인 trx_id 이므로 Undo Log Chain 값 읽기

## Undo Log Chain
- 하나의 행의 값 변경사항 기록을 "trx_id:value" 형태로 연결한 자료구조
- 이 자료구조는 해당 자료구조가 필요한 트랜잭션 종료시 삭제 마킹, 백그라운드 스레드가 삭제 마킹된걸 정리한다