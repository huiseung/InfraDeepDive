# Elastic Search 역사
- 1999년 doug cutting 이 검색엔진 lucene을 개발
- 2001년 lucene 이 apache lucene 이름으로 오픈소스화
- 2010년 shay banon이 apache lucene를 기반으로 elastic search 첫 버전 개발, 아파치 라이센스 2.0
- 2013년 logstash, kibana가 elastic search와 많이 쓰이며 ELK stack이라 불림
- 2018년 엘라스틱 서치를 활용해 클라우드 사에서 수익을 얻는 것에 대응하기 위해 Elastic License 도입
    - 코어 기능은 오픈 소스지만, 고급 기능에는 제한을 두었다
- 2021년 1월, 엘라스틱 서치는 AWS가 자사 이익은 얻지만 오픈소스로 프로젝트에 기여하지 않는다 생각해, 사용자가 SSPL 혹은 Elastic License 2.0 두 종류의 라이센스 중 하나를 선택하도록 변경
    - SSPL은 소스를 사용시, 사용하는 서비스 코드 역시 공개해야 합니다
    - Elasitc License 2.0은 소스를 이용해 관리형 서비스를 만드는 걸 제한합니다
- 2021년 4월, AWS는 이에 대응해 elastic search 오픈소스 시절 코드에서 fork해 OpenSearch 프로그램을 개발