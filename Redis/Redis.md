# 역사
- Remote Dictionary Server의 약자
- 2009년 살바토레 산필리포(salvatore sanfilippo)가 개발
- 풀고 싶은 문제
  - 기존 RDB 데이터베이스의 처리 속도보다 빠른 데이터베이스
  - 복잡한 쿼리 없이 다양한 자료구조의 쓰기/읽기를 지원하는 저장소
- 2010년 pubsub, transaction, lua script 기능 추가


# 아키텍처
- 외부 라이브러리 없는 순수 c언어 라이브러리로만 작성, c컴파일러만 있으면 소스코드 컴파일 가능
- Network IO multi flexing
- in memory store
  - 모든 데이터를 ram에 저장(휘발성)
  - 매우 빠른 IO 지원
- event loop: 만료 처리

- 스레드 구조
    - 네트워크 읽기/쓰기는 멀티 스레드
    - 명령 수행/이벤트 루프 처리등 데이터를 건드리는 처리는 싱글 스레드
    - 백그라운드 스레드
        - RDB BGSAVE, AOF fsync 명령어는 별도 스레드가 처리

# 자료 구조
## String

## 만료 Expire
- redis의 만료 키 제거는 2가지 방식으로 진행된다
- 주기적 샘플링 제거
    - 전체 키중 샘플링해 만료 시간 지난 경우 제거
    - 제거 키 비율이 일정 비율 이상이면 이를 반복
- 지연 삭제
    - 접근시 만료 시간을 확인해 지난 경우 제거

## List

## Set

## SortedSet

## Map

## Stream

## PubSub


# 영속화
## AOF, Append Only File
- 수행 명령어 로그

## RDB File
- 저장소 스냅샷 파일
- SAVE: 동기 저장
- BGSAVE: 비동기 저장

# 가용성
## Sentinel
- Replica 구조 + 복수 Sentinel 노드
- Replica 구조는 하나의 쓰기 처리를 담당하는 Master 노드, 읽기 처리를 담당하는 복수의 Worker 노드로 구성
- Sentinel은 Master 장애를 처리

- Master 장애 대응 과정
    - Sentinel은 ping으로 Master의 장애 확인
    - 정족수 이상의 Sentinel이 Master가 장애라 판단시 fail Worker 중 하나를 Master로 승격

## Cluster
- 전체 데이터를 샤딩해 여러 노드에 나눠 저장

### slot
- 16384개 slot으로 데이터를 관리
    - 데이터의 key를 해싱한 값에 따라 slot 번호 부여
    - 어떤 데이터가 어떤 slot에 할당될지는 key에만 영향을 받고 노드 수에는 영향을 받지 않는다
    - 각 노드는 slot이 어떤 노드에 관리되고 있는지 알고 있다
    - slot은 (key, value)를 hash table으로 관리중이다
- slot 결정 식
    - 키에 hashtag를 포함 시키면 hashtag만으로 slot 결정
        - 같은 hashtag를 가진 키는 같은 slot에 배정
    - hashtag가 없다면 키 전체로 slot 결정
    - slot index = CRC16(hashtag) % 16384
- node 결정 식
    - node index = slot index % node 수
- 쓰기
    - slot 결정식에 따라 
- 읽기
    - key가 있는 slot을 가진 노드에 요청하면 응답
    - key가 있는 slot을 가지지 않은 노드에 요청하면 클라이언트에게 slot을 가진 노드를 알려줘 redirect 진행
    - client는 읽기를 진행할 때마다 key를 가진 node 번호를 캐싱

### node 수 변경시 slot 재배치
- 과정
    - 각 노드의 이동해야할 slot들 탐색
    - 이동할, 이동해 올 slot 상태를 노드의 로컬에 업데이트
    - slot에서 key들을 하나씩 이동
    - 모든 이동 대상 slot이 이동 완료하면 각 노드의 slot:node 테이블 업데이트

```
rebalance 를 수행하는 클라이언트에서 다음을 진행

slots = 이동 대상 slot 탐색
for(slot in 이동 대상 slots){
    for(key in slot){
        migrate(key, slot.migrating_node, slot.importing_node)
    }
}
update_slot_node_table()
```

- slot 상태
    - 노드에는 로컬로 자신이 가진 slot들의 상태를 관리
    - stable: 정상
    - migrating X: 기존 노드에서 X 노드로 이동 중
    - importing X: X 노드에서 새 노드에 들어가는 중

- slot 500이 node A에서 node B로 이동하고 keyA는 이동하지 않음 keyB는 이동 완료했을 때 검색 예시
    - nodeA는 migrating B 상태 slot500 가지고 있다
    - nodeB는 importing A 상태 slot500 가지고 있다
    - slot:node table에는 slot500:nodeA 이다
    - nodeA에서 keyA 조회
        - keyA 탐색 응답
    - nodeA에서 keyB 조회
        - slot500에 keyB가 없음을 확인 migrating B 상태를 보고 클라이언트에게 redirect 
        - 클라이언트는 keyB 관련 요청 보낼 node 정보 캐시를 업데이트
    - nodeB에서 keyA 조회
        - slot500 소유인 nodeA로 redirect
    - nodeB에서 keyB 조회
        - slot500 소유인 nodeA로 redirect
        - nodeA가 redirect로 온 keyB 요청에 대해 응답 처리
    - nodeC에서 keyA 조회
        - slot500 소유인 nodeA로 redirect
        - nodeA에서 keyA 조회 진행
    - nodeC에서 keyB 조회
        - slot500 소유인 nodeA로 redirect
        - nodeA에서 keyB 조회 진행