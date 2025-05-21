- 외부 라이브러리 없는 순수 c언어 라이브러리로만 작성, c컴파일러만 있으면 소스코드 컴파일 가능
- in memory store
  - 모든 데이터를 ram에 저장(휘발성)
  - 매우 빠른 IO 지원
- Network IO multi flexing
- event loop

# Thread 사용 구조
- main thread 1개
  - 요청 수신, 명령 수행, 만료 작업
- 응답 Network IO Thread N개
  - 응답 쓰기 병렬 처리
- AOF 백그라운드 스레드


# event loop
- 레디스는 IO Multiplexing으로 epoll, kqueue, select 모두 구현되어 있다 실행 os에 맞춰 가장 성능 좋은 IO Multiplexing을 이용
  - linux: epoll
  - mac: kqueue
  - 나머지: select