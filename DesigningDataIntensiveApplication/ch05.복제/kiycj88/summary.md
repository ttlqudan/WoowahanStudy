# Part 2. 분산 데이터
### 분산데이터가 필요한 이유
- 확장성
  - 부하를 여러장비로 분배.
- 내결함성/고가용성
  - 장비가 죽더라도 애플리케이션이 계속 동작.
- 지연 시간
  - 가까운 곳에서 데이터를 제공함으로써 네트워크 지연시간 축소. 
    
# chapter 5. 복제
복제란 네트워크로 연결된 여러 장비에 동일한 데이터의 복사본을 유지

#### 복제가 필요한 이유
- Reduce Latency
- HA
- Read Throughput

#### 복제 알고리즘 종류
- single-leader
- multi-leader
- leaderless

### 리더와 팔로워 (single-leader)
- 모든 Write는 single-leader에서 처리하고 나머지 follower로 데이터 복제
- 데이터 변경을 replication log나 change stream의 일부로 팔로워에게 전송해서 복제

#### 동기식 vs 비동기식 복제
- 동기식: Write 할 때, 팔로워까지 변경
  - 팔로워에게도 최신 데이터를 보장할 수 있다. 팔로워가 문제 생기면 Write 처리를 못함.
- 비동기식: Write할 때, 리더에만 반영하고, 팔로워에게는 비동기로 데이터 복제

보통 비동기식을 선호한다고 함.

#### 새로운 팔로워 설정
1. leader snapshot을 덤프
2. snapshot 팔로워에 반영
3. snapshot 이후 데이터 반영 - ex) mysql binlog
4. 데이터 전부 반영되면 follower group에 추가.

#### 노드 중단 처리
- 팔로워 장애: 따라잡기 복구 -마지막 트랜잭션 이후로 다시 데이터 받으면 됨
- 리더 장애: 장애 복구 - 팔로워 중 하나를 새로운 리더로 승격시키고, 클라이언트에서 새로운 리더로 쓰기를 처리하도록 설정 변경 (**FailOver**)
- 
##### Fail-Over 시나리오
1. 리더가 장애인지 판단 - 주기적으로 health check
2. 새로운 리더 선정 - 이전 리더의 최신 데이터 변경사항을 가진 팔로워가 최우선 후보
3. 새로운 리더로 시스템 재설정

##### Fail-Over에서 발생할 수 있는 문제
- 새로운 리더가 이전 리더의 일부 트랜잭션을 받지 못하는 경우
- 두 노드가 모두 자신이 리더라고 생각 하는 경우 (**SplitBrain**)
- Leader의 적절한 health check 설정. 
  - 타임아웃 너무 길게 잡으면, 복구까지 오래 걸림
  - 타임아웃 짧게 잡으면, 오류가 아닌데도 오류로 판단해서 FailOver 진행될 수 있다.

#### 복제 로그 구현
##### Statement-based Replication
- Statement를 그대로 팔로워에게 전달

문제점
- NOW(), RAND() 같은 비결정적 함수 호출시 값이 달라질수 있다.
- 자동증가 칼럼 이용시 정확히 같은 순서로 실행되어야 함. 
- 부수효과를 가진 구문(Trigger, Stored-Procedure, User-Defined Function)은 부수효과가 완벽하게 결정적이지 않으면 문제 발생할 수 있음.

일반적으로 다른 복제 방법을 선호..

##### WAL 배송
- 리더와 완전히 동일한 WAL을 팔로워에게 전송
- low level로 데이터를 기술하기 때문에, 저장소 엔진과 coupling이 심해서 소프트웨어 업데이트가 쉽지 않다.
  - 팔로워가 리더보다 새로운 소프트웨어 버전을 사용하게끔 복제 프로토콜이 허용한다면 팔로워를 먼저 업그레이드함으로써 중단 시간 없이 소프트웨어 업데이트가 수행 가능.

##### Row-based Replicatoin
- Row 단위의 데이터를 그대로 전달
- CDC(change data capture) 를 이용하면 다른 외부 시스템에 데이터베이스 전달하기에 용이

##### Trigger-based Replication
- 트리거를 이용하여 데이터 변경을 분리된 테이블에 로깅하고, 외부 프로세스가 분리된 테이블로부터 Read.
ex) Oracle용 Databus, PostgresQL용 Bucardo

### 복제지연문제
- Strongly Consistency: Write한 내용을 바로 Read 가능
- Eventually Consistency: Write한 내용을 바로 Read 가능하지 않을수 있음.

##### 자신이 쓴 내용 읽기
- 쓰기 후 읽기 일관성: 자신이 Write한 내용을 바로 Read 할수 있어야 한다.

리더기반 복제에서 이를 보장하려면?
- Read from leader
- last modified time을 찾아서, 얼마동안은 Read from leader 

##### 단조 읽기(monotonic read)
- 클라이언트가 여러번 호출시 팔로워들간의 복제지연 차이로 인해, 데이터를 응답받았다가, 다시 응답받지 못하는 문제가 발생할수 있다. 이 문제를 발생하지 않도록 보장하는 것이 **단조 읽기**
- 단조읽기를 보장하는 방법을 각 사용자가 항상 동일한 Replica에서 Read하도록 하는 것

##### 일관된 순서로 읽기
- 일련의 쓰기가 특정 순서로 발생한다면 이 쓰기를 읽는 모든 사용자는 같은 순서로 쓰여진 내용을 보게 됨을 보장
- 파티셔닝된 데이터베이스에서 발생하는 특징적인 문제
- 해결책은 서로 인과성이 있는 쓰기는 동일한 파티션으로 기록되게 하는 것.

### 다중 리더 복제
여러 리더에서 Write 처리
#### 다중 리더 복제의 사용 사례
- 다중 데이터센터 운영: 지리적으로 가까운 데이터센터에 Write, Read 함으로써 성능 향상을 꾀할수 있다.
- 오프라인 작업을 하는 클라이언트: 인터넷 연결이 끊어진 동안 애플리케이션이 계속 동작해야 하는 경우
- 협업 편집: 동시에 여러 사람이 문서를 편집할 수 있는 애플리케이션

#### 쓰기 충돌 다루기
##### 동기 vs 비동기 충돌 감지
single leader에서는 첫번째 쓰기가 완료될때까지 두번째 쓰기를 차단하거나 중단시키고, 사용자에게 쓰기를 재시도하게 할수 있다. 반면 multi-leader에서는 이후 특정 시점에 비동기로만 감지한다.
이론적으로 충돌 감지를 동기적으로 할수 있지만, 그러면 multi-leader의 장점(각 복제 서버가 독립적으로 쓰기를 허용)을 잃는다.

##### 충돌 회피
충돌을 아예 안 만드는 방법
- 특정 레코드의 모든 쓰기는 동일한 리더를 거치도록 애플리케이션이 보장하면 충돌을 방지할수 있다.

##### 일관된 상태 수렴
- 각 쓰기에 고유 ID(ex, timestamp)를 부여하고 가장 높은 ID를 가진 쓰기를 고르고, 다른 쓰기를 버린다. timestamp를 사용하는 경우를 최종 쓰기 승리 (last write wins(LWW))

##### 사용자 정의 충돌 해소 로직
애플리케이션 코드로 충돌 해소 로직을 작성
- 쓰기 수행중: 충돌 핸들러 호출
- 읽기 수행중: 모든 충돌 쓰기를 저장하고, 여러 버전의 데이터를 반환하고, 애플리케이션에서 충돌 해소

#### 다중 리더 복제 토폴로지
- 전체 연결(all-to-all): 전체를 다 연결
- 원형 토폴로지(circular topology): chaining해서 연결
- 별 모양 토폴로지: 하나를 중심으로 다른 노드들 연결
- 원형과 별 모양 토폴로지는 여러 노드를 거쳐야 한다. 무한 루프를 방지하기 위해 각 노드에 고유 식별자를 태깅.
- 원형과 별 모양 토폴로지의 문제점은 하나의 노드에 장애 발생하는 다른 노드간 복제 메시지 흐름에 방해를 줄수 있다.

### 리더 없는 복제
리더 없이 모든 복제 서버가 클라이언트로부터 쓰기를 직접 받는다. (a.k.a. dynamo-style)

#### 노드가 다운됐을 때 데이터베이스에 쓰기
- 리더 없는 복제에서는 Fail-Over가 필요 없다.

##### 읽기 복구와 안티 엔트로피
- 읽기복구: 클라이언트가 여러 노드에서 병렬로 읽기를 수행해서, outdated data를 감지하면 업데이트쳐준다. (Read를 자주 하는 경우에 적합)
- 안티 엔트로피 처리: 백그라운드 프로세스를 두고 복제 서버간 데이터 차이를 지속적으로 찾아서 outdated data를 처리.
 

##### 읽기와 쓰기를 위한 정족수
- w + r > n

#### 정족수 일관성의 한계
w + r > n 으로 설정하더라도 outdated 값을 응답하는 edge case가 있다.
- 느슨한 정족수를 사용한다면, w개의 쓰기는 r개의 읽기와 다른 노드에서 수행될 수 있으므로 r개의 노드와 w개의 노드가 겹치는 것을 보장하지 않는다.
- 두 개의 쓰기가 동시에 발생하면, 어떤 쓰기가 먼저 일어났는지 분명히 알수 없다.
  - 동시 쓰기 합치기
  - timestmap를 사용한다면 last write win - clock skew로 인해 쓰기가 유실될 수 있다.
- 쓰기와 읽기가 동시에 발생하면 쓰기는 일부 복제 서버에만 반영 될수 있다. 읽기가 예전 값인지 최신값 중 무엇인지 알수 없다.

##### 최신성 모니터링
- leaderless 복제에서 최신 결과에 대한 모니터링이 어렵다..

#### 느슨한 정족수와 암시된 핸드오프
노드가 n개 이상인 클러스터에서 네트워크 장애 상황에서 일부 정족수(n) 구성에 들어가지 않는 노드에 연결될 가능성이 있다. 

이 경우의 트레이드오프. 
- 요청에 오류를 반환?
- 일단 쓰기를 받아들이고 값이 보통 저장되는 n개 노드에 속하지는 않지만 연결할 수 있는 노드에 기록? => **느슨한 정족수**
  - 네트워크 장애 상황이 해소되면, 임시적으로 연결한 노드에서 원래의 노드로 쓰기를 전송 => **암시된 핸드오프**
  - **느슨한 정족수**는 쓰기 가용성을 높이는데 특이 유용, 다만, w + r > n인 경우에도 키의 최신값을 읽는다고 보장하지 않는다. 최신 ㄱ밧이 일시적으로 n 이외의 일부 노드에 기록 될수 있기 때문.
  
#### 동시쓰기 감지
동시에 같은 키에 쓰기를 한 경우, 충돌 발생할 수 있다.

##### 충돌 해소 방법
- 최종 쓰기 승리: 가장 최신의 값으로 덮어쓰기 (LWW)
- "이전 발생" 관계와 동시성
  - 두 작업이 동시에 발생했는지 이전에 발생했는지 여부를 결정.
    1. 서버가 모든 키에 대한 버전 번호를 유지하고 키를 기록할때 마다 버전 번호를 증가. 기록한 값은 새로운 버전 번호를 가지고 저장.
    2. 클라이언트가 키를 읽을떄는 최신 버전과 덮어쓰지 않는 모든 값을 반환
    3. 클라이언트가 키를 기록할 때는 이전 익기과 버전 번호를 포함하고 이전 읽기에서 받은 모든 값을 합친다.
    4. 서버가 특정 버전 번호를 가진 쓰기를 받을 때 해당 버전 이하 모든 값을 덮어쓸 수 있다. 하지만 높은 버전 번호의 모든 값은 유지
- 동시에 쓴 값 병합
  - 여러 작업이 동시에 발생하면 클라이언트는 동시에 쓴 값을 정리해야 하는데, 합리적인 접근 방식은 합집합을 취하는 것.
  - 삭제는 툼스톤으로 표시.
- 버전 벡터: 각 복제본 당 자체적으로 버전 번호를 증가.