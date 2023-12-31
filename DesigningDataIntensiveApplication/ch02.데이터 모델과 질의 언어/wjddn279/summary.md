# 2장. 데이터 모델과 질의 언어

## 관계형 모델과 문서 모델

관계형 모델의 대표적인 모델은 관계형 데이터베이스

테이블에서 관계로 구성되고 순서가 없는 tuple(row) 형태, 관계형 모델을 기반으로한 sql

관계형 데이터베이스

- 메인프레임 컴퓨터에서 수행된 비즈니스 데이터 처리
    - 트랜잭션 처리 / 일괄 처리

### NoSQL 의 탄생

2010년대에 등장했으며 관계형 모델의 우위를 뒤집으려는시도

NoSQL 채택의 원동력

- 대규모 데이터셋이나 매우 높은 쓰기 처리량을 달성하기 위한 확장성 필요
- 관계형 모델에서 동작하지 않는 특수 질의 동작
- 더욱 동적이고 표현력이 풍부한 데이터 모델에 대한 바람

### 객체 관계형 불일치

임피던스 불일치 - 애플리케이션 코드와 데이터 베이스 모델 객체의 전환 계층이 필요하고 이러한 분리를 말함 

ORM 과 같은것이 예시

### 다대일과 다대다 관계

정규화를 통한 1:N, N:1 의 관계 설명 (33p에 해당 관계로 설정함의 이점이 잘 나와있음)

예시: 각 tree 구조로 되어 있는 정보를 json 원본으로 저장할지, 정규화된 테이블로 저장할지의 비교

### 어떤 모델이 애플리케이션 코드를 더 간단하게 할까?

문서 모델 (json 원본으로 저장)

- 데이터가 문서와 비슷한 구조(일대다 관계 트리로 보통 한번에 전체 틍리 적재) 일 경우 유리
- 여러 테이블로 나눠서 저장하는 경우 복잡한 코드를 생성해야 함
- 저장과 조회가 단순함, 그대로 json 형태로 저장하기 때문에
- 변경 시 불편함, M:N 관계일 경우 불리함
- 조인이 안됨

정규화된 테이블 → 관계형 데이터 모델 

- 지역성이 떨어짐( 하나의 정보를 얻기 위해 연쇄적으로 조인 쿼리를 실행해야함)
- 하나의 업데이트 시 전부 처리 할  수 있음

## 데이터를 위한 질의 언어

명령형 언어

- 특정 순서로 특정 연산을 수행하게끔 컴퓨터에 지시한다.

선언형 질의 언어

- 목표를 달성하기 위한 방법이 아닌 알고자하는 데이터의 패턴
- 결과가 충족해야 하는 조건이나 데이터의 변환 방법을 정한다.

선언형 질의 언어의 이점

- 순서를 보장하지 않으므로 순서가 바뀌어도 상관없다 (내부적으로 쿼리 옵티마이저가 존재하므로)
- 병렬 실행에 적합하다.
    - 명령형 코드는 특정 순서로 수행하게 끔 지정하므로 병렬처리가 어렵다.
    - 선언형은 알고리즘을 지정하는게 아닌 결과의 패턴만 지정하므로 병렬처리 가능성이 상대적으로 높다.