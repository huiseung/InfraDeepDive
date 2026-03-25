# 메세지 전송시 설정 사항
- 토픽(필수)
- 파티션(선택)
    - 따로 지정하지 않으면 파티셔너에 의해 결정
- 키(선택)
    - 파티셔너가 전송할 파티션을 결정할 때 사용할 데이터
    - 키, 파티션 둘다 지정하지 않으면 스티키 방식으로 분배한다
- ack(필수)
    - 브로커에게 전송후, 브로커에게 오는 kafka protocol 응답을 어떻게 대하까
- pid(자동 생성)
    - brocker가 producer에게 producer 식별자
- seq number(자동 생성)
    - producer는 파티션별로 sequence number를 관리하고 있다
    - 전송 성공시 1 증가

# 구조
- 직렬화 > 전송할 파티션 결정 > 배치에 모아 둠 > 전송

## 직렬화
- 사용자가 지정한 serializer에 의해 키랑 메세지 내용을 직렬화

## 파티셔너
- 기본: key 기반 해시
    - 전송할 파티션 번호 = hash(key)%파티션 수
- 키가 없을 때: 스티키 알고리즘
    - 하나의 배치가 가득 찰 때까지 해당 배치에 메세지를 쌓아두고 전송, 다음 배치에서 이를 반복

## 배치
- 메세지는 단건으로 파티션으로 전송되는게 아니라 일정 크기 또는 주기로 전송한다
- 파티션 별로 배치가 존재


# 신뢰성
- ack: 브로커에게 전송후, 브로커에게 오는 kafka protocol 응답을 어떻게 대하까
    - 0: 기다리지 않음
        - 지연최소
    - 1: leader brocker것만 기다림
        - 기본값
    - all(-1): 모든 브로커 응답 기다림
        - 멱등성, 트랜잭션을 위해 필수 

```
producer.send(record) 호출시 thread가 하는 일

ack=0
batch 전송
callback 실행

ack=1,-1
batch 전송
brocker ack 수신
전송 성공/실패 판정
retry/error 처리
callback 실행
```


# 정합성
## 재전송과 멱등성
```
send()
 └─ batch 전송
     ├─ ACK OK → 완료
     ├─ ACK FAIL
     │    ├─ retry 조건 OK → 재전송
     │    └─ timeout 초과 → 실패 확정
     └─ fatal error → 즉시 실패
```
- retry 하는 목적
    - brocker가 메세지를 받았지만 ack를 전솧하지 못해 producer가 제대로 전송했는지 알지 못함
- retry 트리거
    - 네트워크 오류
    - REQUEST_TIMED_OUT: 브로커 ack가 시간내에 오지 않음
    - NOT_LEADER_FOR_PARTITION: 받는 브로커가 전송할 파티션의 리더가 아니다
    - LEADER_NOT_AVAILABLE: 전송할 파티션의 리더가 정해지지 않았다 
- 멱등성
    - pid와 sequence number를 이용해 관리
    - retry는 전에 실패했던과 동일한 sequence number를 전송, brocker는 (pid, seq number) 조합으로 본인이 받은 메세지인지 확인해 ack를 보낸다
        - brocker는 마지막 성공 seq를 기록하고 있다
        - brocker는 마지막 성공 seq+1인 seq의 메세지만 받아들인다, 작은 값은 ack를 다시 보내고, 큰 값은 오류 처리한다


## 트랜잭션