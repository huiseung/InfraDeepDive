# 토픽 Topic
- 메세지 분류 논리 단위

# 파티션 Partition
- 메세지를 저장하는 물리 단위(폴더)
- 하나의 토픽은 하나 이상의 파티션으로 구성
- 한 파티션안에 메세지들은 offset으로 순서 관리
- 하나 토픽에 여러 파티션을 두는 목적
    - 토픽에 메세지 쓰기/읽기 처리량을 높이기 위해

## 파티션 폴더 구성 파일
- .log
    - 메세지 저장 파일
    - 파티션내 모든 메세지를 하나의 파일에 저장하지는 않고 세그먼트 단위로 나눠 여러 .log 파일에서 관리한다
- .index
    - offset 별 .log의 파일내 byte 위치
    - 모든 offset이 아니라 일부 offset을 듬성 듬성 저장해 저장 공간 효율을 높이고 이분 탐색으로 .log 파일 탐색 시작 위치를 찾는다
    - .index가 관리하는 offset은 파티션에 전체 순서가 아니라 대응하는 .log 파일내 상대적 순서다
- .timeindex
    - timestamp 별 offset
    - 특정 시간 전송된 메세지 찾는데 활용
    - .timeindex가 관리하는 offset도 대응하는 .log 파일내 상대적 순서다
- lear-epoch-checkpoint
    - 리더 변경 정보

- (consumer group, topic, partition) -> offset

# leader & replica
- replica: 하나의 파티션 복제해 보유한 브로커
- leaer: 원본 파티션을 보유한 브로커, 쓰기 주체
- 목적: 장애 대비
- 프로듀서는 브로커로 부터 (토픽, 파티션, 리더) 정보를 받아, 전송할 파티션의 리더 브로커에게 메세지를 전송한다
- bootsrap.servers: 초기 프로듀서가 브로커 정보를 받을 브로커들
