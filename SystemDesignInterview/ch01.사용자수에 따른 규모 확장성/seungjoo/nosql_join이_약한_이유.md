https://chupin-tech.tistory.com/55

중복된 데이터를 정규화하려면 다대일(many-to-one)관계가 필요합니다.
관계형 데이터베이스에서는 조인이 쉽기 때문에 ID로 다른 테이블의 로우를 참조하는 방식은 일반적이지만, 보통 문서 데이터베이스에서는 조인에 대한 지원이 약합니다. 그러므로 다대일 관계는 안타깝게도 문서 모델에 적합하지 않습니다.


NoSQL에서 Join에 대한 지원이 약한 이유는 다음과 같습니다.

### 1. Schema Flexibility
NoSQL 데이터베이스에서는 각 레코드(MongoDB의 경우 문서)가 레코드마다 다를 수 있는 자체 구조를 가질 수 있습니다.
이러한 유연성으로 인해 서로 다른 스키마를 가진 여러 테이블의 데이터를 결합해야 하는 기존 SQL 스타일 조인을 수행하기가 어렵습니다.

### 2. Distributed Architecture
NoSQL 데이터베이스는 여러 노드나 서버에 분산되도록 설계되는 경우가 많습니다.
이러한 배포는 여러 노드와 잠재적으로 네트워크 전체에서 데이터를 수집하는 기존 조인을 비효율적이고 구현하기 복잡하게 만들 수 있습니다.


### 3. Scalability
NoSQL 데이터베이스는 수평 확장성에 최적화되어 있습니다. 즉, 여러 노드에 분산하여 대량의 데이터를 처리할 수 있습니다.
이는 성능과 규모면에서는 훌륭하지만 다양한 분산 노드에서 데이터를 가져와 결합해야 하므로 기존 조인을 효율적으로 구현하기가 더 어려울 수 있습니다.

### 4. Performance
NoSQL 데이터베이스는 성능을 우선시하며, 복잡한 조인은 여러 쿼리 및 데이터 처리가 필요하기 때문에 성능에 영향을 미칠 수 있습니다.
NoSQL 데이터베이스는 읽기 및 쓰기 작업에 최적화되는 경우가 많으며, 이러한 성능 목표를 달성하기 위해 복잡한 조인 기능을 희생해야 할 수도 있습니다.

### 5. Data Denormalization
NoSQL 데이터베이스는 쿼리 성능을 향상시키고 조인의 필요성을 줄이기 위해 데이터 중복을 허용하는 비정규화된 데이터 모델을 장려하는 경우가 많습니다. 이는 정규화가 데이터 중복을 줄이는 핵심 원칙인 관계형 데이터베이스와 대조됩니다.


느끼셨다시피 NoSQL의 특징(혹은 장점)으로 인해 Join 연산이 약해지는 것을 알 수 있습니다.


NoSQL의 특징은 간략하게 다음과 같습니다.

* NoSQL은 엄격한 스키마나 일반적인 관계형 DBMS 테이블 구조가 필요하지 않습니다(Schemaless). 
* 읽기/쓰기 성능에 최적화 되어있어서 엄청난 양의 구조화되지 않은 데이터가 들어오는 대로 저장하고 나중에 구조화할 수 있습니다(Schema on Read). 
서버(노드)를 수평으로 확장할 수 있는 것 또한 특징입니다.

이런 특징으로 인해 데이터 세트간의 관계를 설정하고, 분산된 노드에 있는 데이터들을 모아서 쿼리하는게 어려울 수밖에 없어 보입니다. 

