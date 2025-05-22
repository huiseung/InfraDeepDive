# Invert Index
- 필드 값 조건에 맞는 도큐먼트를 인덱스에서 빠르게 찾기 위해 만들어 두는 자료구조
- Term Dictionary 와 Posting List 로 구성

## Term Dictionary
- 필드에 등장하는 모든 고유값(term)들을 저장한 자료구조
    - 타입에 맞춰 FST 와 BKD Tree 사용
- Posting List와 연결되어 있다

### FST 
- Finite State Transducer
- text, keyword, boolean 필드 저장에 사용
- 빠른 존재여부 파악, prefix 검색에 최적화
- Trie 처럼 term에 포함되는 문자 하나를 노드에 값으로 삼아 구축, 말단에 Posting List와 연결하는 포인터를 둔다

### BKD Tree
- Block K-Dimession Tree
- numeric, date 필드 저장에 사용
- 범위 검색에 최적화
- 이진 트리 & 각 노드는 K 차원 벡터 & 말단은  Posting List와 연결

## Posting List
- 필드에 등장하는 모든 고유값(term)들에 대해 고유값이 등장한 document id 목록
- posting list의 길이가 해당 term의 df다
- TF(term, document, field) 를 계산해 저장하고 있다
