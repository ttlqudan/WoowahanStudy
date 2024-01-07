# 1. 사용자 수에 따른 규모 확장성

## scale up vs scale out
- scale-up: 사양 올리기
- scale-out: 서버 추가하기

### 로드밸런서
- HA
- scale-out을 통한 성능 향상

### 데이터베이스 다중화
- 주로 Primary, Replica로 구성
- 더 나은 성능: 대부분의 applicatoin은 read가 훨씬 많기 때문에 read replica에서 read를 담당하고 write는 primary에서 처리
- 안정성: 장애 발생시 fail-over를 통해 HA 구성

## cache
- 자주 사용되는 연산 결과를 따로 저장해둬서 데이터베이스 쿼리를 줄이기 위한 기법.

### 캐시 사용시 유의할 점
- 데이터 갱신은 자주 일어나지 않고, 참조는 빈번한 경우 효과 좋음
- 영속적으로 보관할 데이터는 캐시하지 않는 것이 좋다
- expire 정책을 잘 세워야 한다. 
  - 짧으면, db query 증가
  - 길면, data sync가 맞지 않을 확률이 높아짐
- 일관성: origin data와 cache data가 같도록 해야 한다. 시스템이 확장될수록 쉽지 않은 문제가 될 수 있다.
- 장애 대처: cache 가 SPOF가 될수 있다.
  - cache 서버 분산으로 해결
- cache memory size
  - 작으면, 자주 eviction될 수 있다.
- eviction 정책

## CDN
- 정적 콘텐츠 캐시를 위해서 주로 사용.
- CloudFront, Akamai

### CDN 사용시 고려해야 할 사항
- 비용
- 적절한 만료 시한 설정
- CDN 장애 시 대처 방안
  - origin에서 컨텐츠 제공 => 이러면 origin이 죽을 수도 있는데???
- 콘텐츠 무효화 방법
  - CDN 서비스에서 제공하는 API 이용
  - object versioning
Cache와 비슷함.

## stateless web 계층
- state를 shared storage에 저장하고, webserver는 stateless하게 구성 => 확장성, 가용성에 용이
- sticky session
- autoscailing

## data center
- 유저에게 가장 가까운 데이터센터로 연결.
- 데이터 센터 장애시 다른 데이터 센터로 우회.
### 기술적 난제
- 트래픽 우회: 올바른 데이터센터로 트래픽을 어떻게 효과적으로 보낼지?
  - least latency?
- 데이터 동기화: 데이터센터 장애시 다른 데이터센터로 우회했을때, 데이터를 어떻게 동기화할 것인가?
  - 여러 데이터센터에 데이터 다중화
- 테스트와 배포
  - 자동화

## message queue
- 비동기로 메시지를 처리
- 서비스들을 loosely coupling하게 할수 있어서, 확장성이 좋다.

## log, metric, 자동화
- log: centrlized logging 을 하면 검색, 조회가 용이
- metric
  - host 단위 metric
  - aggregated metric
  - 핵심 비즈니스 metric
- 자동화: CI/CD 자동화

## 데이터베이스의 규모 확장
- scale-up은 한계까 있다. scale-out을 해야 한다. scale-out을 sharding이라고 함.

### sharding 유의사항
- sharding key and algorithm
- resharding: shard exhaustion시, 데이터를 다시 분배 하는 것 => 5장 consistent hashing 기법에서 소개.
- 유명인사 문제: hotspot 문제, 특정 샤드에 몰리는 것
- 조인과 비정규화: 여러 샤드에 걸친 데이터 조인이 어려움
  - 데이터 비정규화를 하는 것도 하나의 방법
