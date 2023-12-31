# [2. 데이터 모델과 질의 언어](https://rainy-lavender-396.notion.site/2-ss-75eb7ed07ed349d7828900eeb82805f1?pvs=4)
데이터 모델은 문제를 어떻게 생각해야 하는지에 대해서도 지대한 영향을 미친다.

## 관계형 모델과 문서 모델

### 관계형 모델

- 데이터는 관계(relation)로 구성
- 각 관계는 순서없는 튜플(tuple)의 모음
- 비지니스 데이터 처리에 근원이 있음.
- 요즘엔 웹에서 볼 수 있는 대부분의 서비스(온라인 게시물, 토론, 소셜 네트워크, 전자 상거래, 게임, SaaS)는 여전히 관계형 데이터베이스를 통해 제공

### NoSQL 의 탄생

- 대규모 데이터셋이나 매우 높은 쓰기처리량 달성을 관계형데이터베이스보다 쉽게 할 수 있는 확장성 필요
- 무료 오픈소스 소프트웨어 선호 확산
- 관계형 모델에서 지원하지 않는 특수 질의
- 관계형 스키마의 제한에 대한 불만과 동적이고 표현력 풍부한 데이터 모델에 대한 바람

### 객체 관계형 불일치

- 임피던스(impedance) 불일치 → 우리가 주로 사용하는 OOP 언어와 sql 간에 데이터 구조 차이
    - 문서 디비는 json 으로 표현하면 된다. 하지만 관계형 디비는 여러 테이블을 만들고 테이블 간 연관관계를 설정해야 한다.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/93f2a490-9bb2-4daf-a57e-ad251bd791ab/ee364164-e54b-45f8-8d0b-e1555abab1ac/Untitled.png)
    
- json 모델은 RDBMS 의 다중 테이블 스키마 보다 더 나은 지역성을 갖는다.
    - 위와 같은이력서 데이터를 저장 하기 위해
        - 관계형 DB는 multi table 스키마로 정규화하고 join
        - NoSql은 Json 형식으로 저장
    - 여러 테이블간 난잡한 다중 조인 vs json 으로 한번에 질의

### 다대일과 다대다 관계

- 다대일
    - 중복된 데이터를 정규화 하려면 다대일 관계가 필요한데 문서 모델은 다대일 관계에 적합하지 않다.
        - 문서모델은 조인에 대한 지원이 보통 약함.
    - 애플리케이션에서 다중 질의를 해서 join을 흉내내야 함
    - 초기에 join 없는 문서 모델에 맞게 애플리케이션을 만들더라도 애플리케이션이 발전하면서 데이터는 점차 상호 연결되는 경향이 있음.

## 문서 데이터베이스는 역사를 반복하고 있나?

초기 계층 모델 → 관계형 모델 vs 네트워크 모델

### 네트워크모델

- 레코드에 접근하는 유일한 방법은 최상위 레코드에서부터 연속된 연결 경로를 따르는 방법 → 접근 경로
- 레코드가 다중 부모를 가진다면 애플리케이션 코드는 다양한 관계를 모두 추적해야함.(복잡도가 n)
- 데이터베이스 질의와 갱신을 위한 코드가 복잡하고 유연하지 못함.

### 관계형 모델

- 접근 경로를 개발자가 아닌 질의 최적화기가 자동으로 만든다.
    - 하나 만들어놓으면 범용으로 사용 가능

### 문서 데이터베이스와의 비교

- 계층 모델 처럼 상위 레코드 내에 중첩된 레코드를 저장함.
- 다대다 관계 표현은 관계형데이터베이스 처럼 고유한 식별자를 통해 참조함. → 코다실의 전철을 밟지 않고 있음.

## 관계형 데이터베이스와 오늘날의 문서데이터베이스

- 데이터 모델의 차이에만 집중해보면
- 문서 데이터모델은 스키마 유연성, 지역성에 기인한 더 나은 성능을 가짐
- 일부 애플리케이션의 경우 애플리케이션에서 사용하는 데이터 구조와 더 가깝다.
- 관계형데이터베이스 ㅁ델은 조인, 다대일, 다대다 관계를더 잘 지원함.

### 어떤 데이터모델이 애플리케이션 코드를더 간단하게 할까?

- 데이터가 문서와 비슷한구조 라면 문서모델을 사용!
- 미흡한 조인지원이 문제 되지 않는다면~
- 상호연결이 많은 데이터의 경우 문서모델보다는 관계형 모델이 무난. 그래프모델은 매우 자연스러움

### 문서 모델에서의 스키마 유연성

- 문서모델은 암묵적으로 읽기 스키마 구조를 가지고 있다.
    - 관계형 db는 쓰기 스키마
- 쓰기 스키마 형태의 경우 스키마 변경 할 때  성능이 안좋다.
    - 새로운 컬럼을 추가할 때(mysql 은 수분까지 걸릴 수 있음)

### 질의를 위한 데이터 지역성

- 웹 페이지 상에서 문서를 보여주는 동작 처럼 애플리케이션이 자주 전체문서에 접근해야 할 때 저장소 지역성을 활용하면 성능 이점이 있다.
- 지역성의 이점은 한 번에 해당문서의 많은부분을 필요로 하는 경우에만 적용됨.(그렇지 않으면 낭비)
- 오라클은 다중 색인 클러스터 테이블, 빅테이블 데이터 모델 칼럼 패밀리 개념이 지역성관리와 유사한 목적이 있음.

### 문서 데이터베이스와 관계형 데이터베이스의 통합

- 대부분의 관계형 데이터베이스시스템(mysql 제외)은 xml을 지원한다.
    - json 도 지원함.
- 문서 db 에서 보면 리싱크DB는 관계형 조인을지원하고 몽고DB는 자동으로 데이터베이스 참조(클라이언트 측에서 지원하는 것이기 때문에 dbms에서 지원하는것보다 느림)
- 각 데이터모델이 서로 부족한 부분을 보완해나가고 있음.

## 데이터를 위한 질의 언어

- 선언형 vs 명령형
- 선언형은
    - 명령형 api 보다 더 간결하고 쉽게 작업할 수 있음
    - 질의 변경 없이 시스템 성능 향상 가능
    - 병렬 실행에 적합

### 웹에서의 선언형 질의

- css(선언) vs js(명령)
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/93f2a490-9bb2-4daf-a57e-ad251bd791ab/d1c013ee-0cad-41d1-ba55-51c0d967de9f/Untitled.png)
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/93f2a490-9bb2-4daf-a57e-ad251bd791ab/0054107b-48bf-4691-9b77-f3ca7980510f/Untitled.png)
    
    - 선언형에 비해서 명령형이 코드량이 많고 오해하는 데 더 오래걸림.
    - 새로운 성능 좋은 api를 사용하려면 직접 변경해줘야함.

### 맵리듀스 질의

- 선언형 질의 언어도 아니고 명령형 질의 API도 아닌 중간.
- 전반적인 맵리듀스는 10장에서. 지금은 몽고DB의 예시
- 몽고DB 의 map 과 reduce
    - 순수 함수여야 함. → 데이터베이스가 임의 순서로 어디서나 이 함수를 실행 할 수 있고 장애가 발생해도 함수를 재실행 가능.
    - 질의 중간에 js 사용 가능
    - 집계 파이프라인 이라 부르는 선언형 질의 언어 지원 추가(SQL과 유사 하지만 json 기반 구문 사용)

## 그래프형 데이터 모델

데이터에서 다대다 관계가 매우 일반적인 경우 적합.

- 그래프는 정점(노드나 엔티티) 와 간선(관계나 호) 두 객체로 이루어짐
    - 정점은 동종 데이터가 아니어도 됨(사람, 장소, 이벤트, 체크인, 사용자가 작성한 코멘트 모두가 정점이 될 수도 있다.)

- 사용하는 예 는다음과 같다.
    - 소셜 그래프 - 정점은사람이고, 간선은 사람들이 알고 있음을 나타냄.
    - 웹 그래프 - 정점은 웹 페이지고 간선은 다른 페이지에 대한 HTML 링크
    - 도로나 철도 네트워크 - 정점은 교차로이고 간선은 교차로 간 도로나 철로 선을 나타냄

### 속성 그래프

- 정점(vertex) 의 요소
    - 고유한 식별자
    - 유출(outgoing edge) 간선 집합
    - 유입(incoming edge) 간선 집합
    - 속성 컬렉션(키-값 쌍)
- 간선(edge) 의 요소
    - 고유한 식별자
    - 간선이 시작하는 정점(꼬리정점)
    - 간선이 끝나는 정점(머리정점)
    - 두 정점 간 관계 유형을 설명하는 레이블
    - 속성 컬렉션(키-값 쌍)
- 장점
    - 데이터 모델링을 위한 많은 유연성을 제공
    - 확장이 쉬움

### 사이퍼 질의언어

- 사이퍼: 속성 그래프를위한 선언형 질의 언어

### SQL의 그래프 질의

- 그래프 데이터를 관계형 구조로 넣고 SQL 을 사용해 질의 가능(약간 어렵다)
- 할 수는 있지만 데이터 종류에 적절한 데이터 모델을 선택하는 것이 효율적

### 트리플 저장소와 스파클

- 속성 그래프 모델과 거의 동등
- 모든 정보를 주어, 서술어, 목적어처럼매우간단한 세 부분 형식으로 저장함.
    - 주어는 정점과 동일
    - 목적어는
        - 문자열이나 숫자 같은 원시 데이터타입의 값
        - 그래프의 다른 정점

### 시멘틱 웹

- 2000년대 초반에 과대평가 되었고 지금까지 현실에서 실현된 흔적이 없음

### RDF 데이터 모델

### 스파클 질의 언어

- RDF 데이터 모델을 사용한 트리플 저장소질의 언어

### 초석: 데이터로그

- 스파클이나 사이퍼 보다 훨씬 오래된 언어
- 일부시스템에서 사용
    - 데이토믹 - 질의 언어로 사용
    - 캐스캘로그 - 데이터로그의 구현체. 하둡의 대량 데이터셋에 질의를 위한 용도

## 정리

- 역사적으로 데이터를 하나의 큰 트리(계층 모델) 로 표현하려고 노력했지만 다대다 관계를 표현하기에 트리가 적절하지 않았다.
- 이 문제를 해결하기 위해 관계형 모델이 고안
- 관계형 모델에 적합하지 않은 애플리케이션이 있었고
- NoSQL 이 나옴
- NoSQL 은
    - 문서 데이터베이스
    - 그래프 데이터베이스
    
    로 나뉨
    
- 문서 및 그래프데이터베이스는 데이터를 위한 스키마를 강제하지 않아 변화하는 요구사항에 맞춰 애플리케이션을 쉽게 변경할 수 있다는 장점이 ㅇㅆ음.
- 각 데이터 모델은 고유한 질의 언어나 프레임워크를 제공하는데
    - SQL, 맵리듀스, 몽고DB의 집계 파이프라인, 사이퍼, 스파클 데이터로그가 있음
- CSS 와 XSL/XPath 도 있는데, 데이터베이스 질의언어와 유사점이 있음
