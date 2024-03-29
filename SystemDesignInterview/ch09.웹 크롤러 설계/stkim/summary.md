# Ch09. 웹 크롤러 설계

- 크롤링 사용 목적

  - 검색 엔진 인덱싱 : 웹 페이지를 모아 검색 엔진을 위한 로컬 인덱스 생성.

  - 웹 아카이빙 : 나중에 사용할 목적으로 장기 보관하기 위해 웹에서 정보 모으는 절차

  - 웹 마이닝 : 유용한 지식 도출

  - 웹 모니터링 : 크롤러를 사용하면 인터넷에서 저작권이나 상표권 침해되는 사례 모니터링 가능.

## 1. 문제 이해 및 설계 범위 확정

- 검색 엔진 인덱싱에 사용

- 10억 (1Billion) 웹 페이지 수집

- 새로 만들어진 웹페이지나 수정된 웹 페이지도 고려

- 수집한 웹 페이지는 5년간 저장이 필요함.

- 중복된 콘텐츠를 갖는 페이지는 무시해도 무방함.

- 그 외 웹 크롤러를 구성하기 위해 필요한 속성들

  - 확장성 : 병행성을 사용해서 효과적인 웹 크롤링 수행

  - 안정성 : 잘못 작성된 HTML, 아무 반응이 없는 서버, 장애, 악성 코드가 붙어 있는 링크 등에 잘 대응

  - 예절 : 짧은 시간 동안 너무 많은 요청 보내면 안됨

  - 확장성 : 새로운 형태 콘텐츠 지원이 용이해야함.

### 개략적 규묘 추정

- 매달 10억개 웹 페이지 다운로드

- QPS = 10억 / 30일 / 24시간 / 3600초 = 대략 400페이지/초

- 최대 QPS = 2 \* QPS = 800

- 웹 페이지 최대 크기 평균은 500k

- 10억 페이지 x 500k = 500TB/Month

- 5년 간 저장하면 30PB 필요 (500TB x 12Month x 5years)

## 2. 개략적 설계안 제시 및 동의 구하기

### 시작 url 집합

- 크롤러 시작 지점.

- 크롤러가 가능한 많은 링크 탐색할 수 있는 url 선택 필요

- 일반적으로 전체 url 공간을 작은 부분집합으로 나누는 전략 필요.

### 미수집 url 저장소

- 현대 웹 크롤러의 상태는 2가지로 나눠 관리함.

  - 다운로드할 url. 또는 미수집 url 저장소라고도 부름

  - 다운로드 된 url

### HTML 다운로더

- 인터넷 웹페이지를 다운로드하는 컴포넌트

### 도메인 이름 변환기

- 웹 페이지를 다운로드 받으려면 url을 ip 주소로 변환하는 절차가 필요함.

### 콘텐츠 파서

- 콘텐츠를 다운로드하면 Parsing과 Validation이 필요함.

- 이상한 웹 페이지는 문제 발생할 여지가 다분하고, 저장 공간 낭비 가능성이 높음.

### 중복 콘텐츠인가?

- 웹의 29% (편의상 30%)는 중복 컨텐츠임

- 자료 구조를 도입해서 데이터 중복을 줄이고, 데이터 처리에 소요되는 시간을 줄인다.

- 효과적인 방법으로 해시 값 비교가 있음.

### 콘텐츠 저장소

- 저장 데이터 유형, 크기, 저장소 접근 빈도, 데이터 유효 기간 등을 종합적으로 고려함.

- 데이터 양이 너무 많으므로 대부분의 콘텐츠는 디스크에 저장

- 인기 있는 컨텐츠는 메모리에 둬서 접근 지연시간을 줄인다.

### URL 추출기

- 상대 경로는 전부 origin url(예를 들어 https://en.wikipedia.org)를 붙여 절대 경로로 변환.

### URL 필터

- 특정 콘텐츠 타입이나 확장자, 접속 시 오류가 발생하는 url, 접근 제외 목록에 포함된 url 등을 크롤링 대상에서 배제하는 역할

### 이미 방문한 URL?

- 이미 방문한 url이나 미수집 url 저장소에 보관된 url을 추적할 수 있도록 하는 자료 구조 사용.

- 같은 url을 여러 번 처리하는 일을 방지 가능.

- 자료 구조로 해시 테이블이나 블룸 필터가 널리 사용됨.

### URL 저장소

- 이미 방문한 url을 보관하는 저장소.

### 웹 크롤러 작업 흐름

![workflow](https://github.com/ttlqudan/WoowahanStudy/assets/40455392/b073eecc-ae08-4c04-9d06-34773cb1eb6c)

1. 시작 url들을 미수집 url 저장소에 저장.

2. HTML 다운로더는 미수집 url 저장소에서 url 목록을 가져온다.

3. HTML 다운로더는 도메인 이름 변환기를 사용해서 URL의 ip 주소를 알아내고, 해당 ip 주소로 접속하여 웹 페이지를 다운로드 받음.

4. 콘텐츠 파서는 다운된 HTML 페이지를 파싱하여 올바른 형식을 갖춘 페이지인지 검증한다.

5. 콘텐츠 파싱과 검증이 끝나면 중복 콘텐츠인지 확인하는 절차를 개시한다.

6. 중복 콘텐츠인지 확인하기 위해서, 해당 페이지가 이미 저장소에 있는지 본다.

   - 이미 저장소에 있는 컨텐츠인 경우에는 처리하지 않고 버린다.

   - 저장소에 없는 컨텐츠인 경우 저장소에 저장한 뒤 url 추출기로 전달.

7. url 추출기는 해당 HTML 페이지에서 링크를 골라낸다.

8. 골라낸 링크를 url 필터로 전달한다.

9. 필터링이 끝나고 남은 url만 중복 url 판별 단계로 전달.

10. 이미 처리한 url인지 확인하기 위해, url 저장소에 보관된 url인지 살핀다. 이미 저장소에 있는 url은 버린다.

11. 저장소에 없는 url은 url 저장소에 저장할 뿐 아니라 미수집 url 저장소에도 전달한다.

## 3. 상세 설계

- 가장 중요한 컴포넌트와 그 구현 기술

  - DFS vs BFS

  - 미수집 url 저장소

  - HTML 다운로더

  - 안정성 확보 전략

  - 확장성 확보 전략

  - 문제 있는 콘텐츠 감지 및 회피 전략

### DFS vs BFS

- 그래프가 어느 정도 이상 크면 DFS로 얼마나 깊이 있게 탐색할지 가늠이 어려움.

- BFS를 그래서 보통 많이 사용. FIFO 기반 큐.

  - 단, 2가지 문제 존재함

    - 같은 호스트의 많은 링크 다운받느라 바빠져, 이 때 링크들을 병렬로 처리하게 된다면 위키피디아 서버는 수많은 요청으로 과부하 걸리게 됨.

    - 모든 웹페이지가 같은 수준의 품질, 같은 수준의 중요성 안가지므로 페이지 순위 (page rank), 사용자 트래픽, 업데이트 빈도 등 여러 척도에 비추어 처리 우선순위를 구별하는 게 온당함.

### 미수집 URL 저장소

- '예의(politeness)' 갖춘 크롤러, url 사이의 우선순위와 신선도를 구별하는 크롤러를 구현

#### 예의

- 너무 많은 요청을 보내는 것은 무례한(impolite) 일이며, 때로는 DoS 공격으로 간주되기도함.

- 동일 웹 사이트에 대해서는 한 번에 한 페이지만 요청한다는 것.

  - 같은 웹 사이트의 페이지를 다운로드 받는 태스크는 시간차를 두고 실행하도록 하면 될 것이다.

  - 이 요구사항을 만족시키려면 웹 사이트의 호스트명과 다운로드를 수행하는 작업 스레드 사이의 관계를 유지하면 됨.

    - 큐 라우터 (queue router): 같은 호스트에 속한 url은 언제나 같은 큐로 가도록 보장하는 역할.

    - 매핑 테이블 (mapping table): 호스트 이름과 큐 사이의 관계를 보관하는 테이블.

    - FIFO 큐(b1부터 bn까지): 같은 호스트에 속한 url은 언제나 같은 큐에 보관함.

    - 큐 선택기 (queue selector): 큐 선택기는 큐들을 순회하면서 큐에서 url을 꺼내서 해당 큐에서 나온 url을 다운로드하도록 지정된 작업 스레드에 전달하는 역할

    - 작업 스레드 (worker thread): 작업 스레드는 전달된 url을 다운로드하는 작업을 수행.

#### 우선순위

- 유용성에 따라 url의 우선순위를 나눌 때는 페이지랭크 (page rank), 트래픽 양, 갱신 빈도 등 척도 사용 가능

- 우선순위를 고려하여 변경한 설계

  - 순위 결정 장치(prioritizer): url을 입력으로 받아 우선순위를 계산

  - 큐(f1, ..., fn): 우선순위별로 큐가 하나씩 할당된다.

  - 큐 선택기: 임의 큐에서 처리할 url을 꺼내는 역할 수행.

#### 예의 + 우선순위 반영한 전체 설계

- 전면 큐(front queue): 우선순위 결정 과정 처리

- 후면 큐(back queue): 크롤러가 예의 바르게 동작하도록 보증.

![final](https://github.com/ttlqudan/WoowahanStudy/assets/40455392/917f9923-8788-4323-89b0-6630ad8c0f00)

#### 신선도

- 웹 페이지는 수시로 추가 / 삭제 / 변경됨.

- 데이터 신선함을 유지하기 위해 이미 다운로드한 페이지도 주기적으로 재수집(recrawl) 필요.

  - 이 작업 최적화를 위한 전략

    - 웹 페이지의 변경 이력 활용

    - 우선순위 사용해서, 중요한 페이지는 더 자주 수집.

#### 미수집 url 저장소를 위한 지속성 저장장치

- 대부분의 url은 디스크에 두지만 IO 비용을 줄이기 위해 메모리 버퍼에 큐를 두는 것.

- 버퍼에 있는 데이터는 주기적으로 디스크에 기록함.

### HTML 다운로더

- http 프로토콜을 통해 웹 페이지를 내려 받음.

- 함께 알면 좋을 것 : robot.txt 제외 프로토콜

  - 주기적으로 다운로드 받아 캐싱. 다운로드 받을 수 없는 것들에 대한 리스트 나열되어 있음.

#### 성능 최적화.

- 분산 크롤링 : 크롤링 작업을 여러 서버에 분산

- 도메인 이름 변환 결과 캐시 :

  - 도메인 이름 변환기 (DNS Resolver)는 크롤러 성능의 병목 중 하나임. 원인은 이는 DNS 요청을 보내고, 결과를 받는 작업의 동기적 특성때문.

  - DNS 요청 처리는 보통 10ms ~ 200ms 소요됨.

  - 크롤러 스레드 가운데 어느 하나라도 이 작업을 수행하면 다른 스레드의 DNS 요청은 전부 블록(block) 된다.

  - 따라서 DNS 조회 결과로 얻어진 도메인 이름과 IP 주소 사이의 관계를 캐싱해 보관해 넣으면 크론잡 등을 돌려 주기적으로 갱신하도록 하면 성능을 효과적으로 높일 수 있음.

- 지역성 : 크롤러 작업 수행하는 서버를 지역별 분산. 크롤 서버, 캐시, 큐, 저장소 모두에 활용 가능함.

- 짧은 타임아웃 :

  - 어떤 웹 서버는 응답이 느리거나 아얘 응답을 안하므로, 대기 시간을 짧게 가져가는 것이 좋음.

  - 이 시간 동안 서버가 응답하지 않으면 크롤러는 해당 페이지 다운로드 중단 후 다음 페이지로 넘어감.

### 안정성

- 안정 해시 : 다운로드 서버들이 부하를 분산할 때 사용하는 기술

- 크롤링 상태 및 수집 데이터 저장 : 장애 발생에 유연하게 대응하기 위해 크롤링 상태 및 수집된 데이터를 지속적 저장 장치에 기록해 둘 수 있음.

- 예외 처리 : 장애 발생 후에도 전체 시스템 중단 없이 우아하게 이어갈 수 있어야함.

- 데이터 검증 : 시스템 오류 방지.

### 확장성

- 진화하지 않는 시스템은 없고, 이런 시스템 설계에는 새로운 유형의 컨텐츠를 쉽게 지원할 수 있도록 신경써야함.

  - 예) PNG 다운로더, 웹 모니터

### 문제 있는 콘텐츠 감지 및 회피

- 중복 콘텐츠 : 해시나 체크섬(check-sum)을 사용하면 중복 콘텐츠를 보다 쉽게 탐지

- 거미 덫(spider trap): 크롤러를 무한 루프에 빠지도록 설계한 웹 페이지.

  - 수작업으로 덫 확인하고 찾아내서 크롤러 탐색 대상에서 제외하거나 url 필터 목록 추가

- 데이터 노이즈 : 광고, 스크립트 코드, 스팸 url 등 제외 필요.

## 마무리

- 서버 측 렌더링 : 많은 웹 사이트가 JS, Ajax 등을 사용해 링크를 즉석으로 만들어 내는데, 페이지 파싱 전 서버 측 렌더링을 적용하면 해결 가능

- 원치 않는 페이지 필터링 : 스팸 방지 컴포넌트를 둬서 품질이 조악하거나 스팸성 페이지를 걸러내도록 해두면 좋음.

- 데이터 베이스 다중화 및 샤딩

- 수평적 규모 확장성

- 가용성, 일관성, 안정성

- 데이터 분석 솔루션
