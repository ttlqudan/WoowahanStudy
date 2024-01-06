# Chapter 1. 사용자 수에 따른 규모 확장성

## 들어가기 전에...

- 천릿길도 한 걸음부터 : 복잡한 시스템도 모든 컴포넌트가 단 한 대의 서버에서 실행되는 간단한 시스템부터 설계됨

## 1. 단일 서버

![Single Node](https://github.com/ttlqudan/WoowahanStudy/assets/40455392/c758fc56-51d2-43e5-8b81-f6a894d4adbd)

- 사용자는 DNS를 사용해서 접속 시도함. 접속을 위해서는 DNS -> IP 로 변환하는 과정 필요함.

- DNS에서 IP 주소 반환됨.

- IP 주소로 HTTP 요청 전달됨.

- 요청 받은 웹 서버는 JSON or HTML 페이지로 응답이 반환됨.

## 2. 데이터베이스

![Database](https://github.com/ttlqudan/WoowahanStudy/assets/40455392/976bd233-0669-44e9-ae89-897b79ba1084)

- RDBMS : MySQL, Postgresql, Oracle ...

- NoSQL : 키-값 저장소 (key-value store), 그래프 저장소 (graph store), 칼럼 저장소 (column store), 그리고 문서 저장소 (document store)

  - 예시 소프트웨어: MongoDB, DynamoDB, HBase...

- 비관계형 DB를 사용해 볼 것을 고려해 볼 만한 상황

  - 아주 낮은 응답 지연시간 (low latency)

  - 다루는 데이터가 비정형 데이터일 때

  - 데이터(JSON, XML, YAML 등)를 직렬화하거나 역직렬화만 하면 되는 경우

  - 아주 많은 양의 데이터를 저장할 필요가 있을 때

## 3. 수직적 규모 확장 vs 수평적 규모 확장

- 수직적 규모 확장 (Scale Up): 고사양 자원을 추가하는 행위

  - 단점

    - 한 대의 서버에 CPU나 메모리를 무한대 추가 불가

    - Failover 상황에 대한 자동 복구나 다중화 방안 제시 못함. 서버 장애 발생 시, 웹/앱은 바로 중단됨.

- 수평적 규모 확장 (Scale Out): 더 많은 서버를 추가해서 성능을 개선하는 행위

- 웹 서버가 다운되면 접속이 안되기 때문에 부하 분산기 또는 로드 밸런서 (Load Balancer)를 사용함.

## 4. 로드 밸런서

![Load Balancer](https://github.com/ttlqudan/WoowahanStudy/assets/40455392/c3a86925-e93f-465a-8851-e754715f7ad0)

- 사용자는 로드밸런서의 public ip로 접속하고, 서버 간 통신에는 private ip를 사용함.

## 5. 데이터베이스 다중화

![multi-database](https://github.com/ttlqudan/WoowahanStudy/assets/40455392/8ac8fa98-d3dc-4179-a9b0-93bbc6aa32b5)

- 보통은 DB 서버 사이에 주(master)-부(slave) 관계 설정해서 원본은 주 서버에, 사본은 부 서버에 저장하는 방식 채용.

- 보통 이 구조는 2가지 장점을 가져감

  - 쓰기 작업은 주 DB에서, 읽기 작업은 부 DB에서 수행하기에 성능이 좋아짐.

  - 자연 재해 등으로 일부 서버가 고장나거나 데이터 유실 되어도 데이터 보존 가능함.

  - 데이터를 여러 지역에 복제해둠으로써 장애 발생해도 서비스 계속 유지 가능

![Load Balancer & DB](https://github.com/ttlqudan/WoowahanStudy/assets/40455392/94c649dd-57ed-441a-a65c-ce3cf6512e77)

- 일부 DB가 다운되서 나머지 DB에 부하가 몰릴 때, 그리고 주 DB가 다운되어 부 DB 중 하나가 새로운 주 DB가 되서 데이터가 최신화 되지 않았을 때, 예전에는 복구 스크립트를 돌렸음.

  - 다중 마스터 또는 원형 다중화 방식도 있으나 이 책에서는 다루지 않음.

- 그래서 위의 로드밸런서와 DB 다중화를 고려한 설계를 제안함.
