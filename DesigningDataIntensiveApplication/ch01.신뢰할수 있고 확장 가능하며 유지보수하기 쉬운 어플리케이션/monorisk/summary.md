## 신뢰할 수 있고 확장 가능하며 유지보수하기 쉬운 애플리케이션
애플리케이션은 기능적 요구사항과 비기능적 요구사항을 만족해야 함
세 가지 비기능적 관심사를 중심으로 설명

### 신뢰성
무언가 잘못되더라도 지속적으로 올바르게 동작함

잘못될 수 있는 것을 결함이라고 함
결함을 예측하고 대처할 수 있는 시스템을 내결함성 또는 탄력성을 지녔다고 정의
> 결함과 장애는 동일하지 않다. 장애는 사용자에게 필요한 서비스를 제공하지 못하고 시스템 전체가 멈춘 경우

고의적으로 결함을 유돟마으로써 내결함성시스템을 지속적으로 훈련하고 테스트하여
결함이 자연적으로 발생했을 때 올바르게 처리할 수 있는 자신감을 높임
> 카오스 몽키가 한 가지 예

보안 같은 영역은 결함 예방책이 해결책보다 더 필요하다.

신뢰성이 떨어지면 경제적 손실이 발생할 수 있음
하지만 자원 절감을 위해 신뢰성을 희생해야 하는 경우도 있으며, 이때 적절한 트레이드 오프를 이해하고 있어야 함


#### 하드웨어 결함
과거에는 크티티컬한 시스템이 아니고 다운 타임이 길지 않은 이상 다중 장비 중복을 이용한 고가용성은 불필요했음.

하지만 애플리케이션이 요구하는 하드웨어의 양들이 늘어감에 따라 다운타임또한 증가해가고 있음
이에따라 소프트웨어 내결함성 기술을 응용하거나 하드웨어 중복성을 추가해 전체 장비 손실을 견딜 수 있는 시스템으로 가고 있음


#### 소프트웨어 결함
소프트웨어 결함은 체계적이며 신속한 처리가 어려움
아래의 방법들이 해결에 도움을 줌

- 시스템의 가정과 상호작용에 대해 주의 깊게 생각하기
- 빈틈없는 테스트
- 프로세스 격리
- 죽은 프로세스의 재시작 허용
- 프로덕션 환경에서 시스템 동작의 측정
- 모니터링
- 분석하기


#### 인적 오류
인적 오류도 오류의 한 축을 차지함

이런 오류를 줄이기 위해 인지적 부하를 낮추고 조작을 간단하게 할 수 있으며
복구의 기회를 주고 쉽게 테스트를 할 수 있는 환경을 주어 오류를 줄일 수 있음



### 확장성
증가하는 부하에 대처하는 시스템 능력

#### 부하 기술하기
부하에 대해 기술해야 우리의 상태가 어떤지, 앞으로 무엇을 해야 하는지를 알 수 있다.
가장 적합한 부하 매개변수는 시스템에 따라 다르다.


#### 성능 기술하기
부하가 기술 되면 이 부하를 해결하기 위한 성능을 기술해야 함
아래의 두 가지 질문으로 탐색해볼 수 있음
- 부하 매개변수를 증가시키고 시스템 자원은 변경하지 않고 유지하면 시스템 성능은 어떤 영향을 줄까?
- 부하 매개변수를 증가시켰을 때 성능이 변하지 않고 유지되길 원한다면 자원을 얼마나 많이 늘려야 할까?

#### 부하 대응 접근 방식
용량 확장과 규모 확장 방식이 있음
용량 확장은 간단하나 고사양 장비의 값비싼 비용이 문제가 됨
규모 확장이 대두됨
상태 비저장 애플리케이션은 확장이 용이하나, 상태 저장 애플리케이션은 확장이 어려움
그래서 데이터 저장소등의 애플리케이션은 최후까지 규모 확장을 지연하는 경우가 많음
데이터와 트랜잭션의 성질에 따라 확장 방법을 선택해야 함




### 유지보수성
소프트웨어의 대부분 비용은 유지보에 들어감
비용을 줄이기 위해 세 가지 원칙을 고려할 것
1. 운용성 -> 쉽게 운용할 수 있게 만들 것
2. 단순성 -> 시스템의 복잡도를 낮추어 인지 부하를 낮출 것
3. 발전성 -> 추후에 시스템이 쉽게 확장될 수 있도록 할 것
