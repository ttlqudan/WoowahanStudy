# Ch05. 안정 해시 설계

- 수평적 규모 확장성을 달성하기 위해서는 요청 또는 데이터를 서버에 균등하게 나누는 것이 중요하다.

- 안정 해시는 이 목표를 달성하기 위해 보편적으로 사용하는 기술.

## 해시 키 재배치 문제

- N개의 캐시 서버가 있을 때, 이 서버들에 부하를 균등하게 나누는 보편적인 방법은 아래 해시함수를 사용하는 것이다.

- serverIndex = hash(key) % N

- 예시 : hash(key0) % 4 = 1 이라면, 클라이언트는 캐시에 보관된 데이터를 가져오기 위해 서버 1번에 접속해야함.

  - 1번 서버 죽으면? 대규모 캐시 미스 문제 발생

## 안정 해시

- 해시 테이블 크기가 조정될 때, 오직 k/n개의 키만 재배치하는 해시 기술. 여기서 k는 키 갯수, n은 슬롯 갯수임.

### 기본 구현법의 두 가지 문제.

- 기본 절차

  - 서버와 키를 균등 분포 해시 함수를 사용해 해시 링에 배치

  - 키의 위치에서 링을 시계 방향으로 탐색하다 만나는 최초의 서버가 키가 저장될 서버다.

- 문제점

  - 서버가 추가되거나 삭제되는 상황에선 파티션 크기를 균등하게 유지하는 게 불가능

    - 좀 더 구체적으로 말하면, 어떤 서버는 굉장히 작은 해시 공간을 할당 받고, 어떤 서버는 큰 공간을 받는 문제 존재

  - 두번째는 키의 균등 분포 달성이 어렵다.

- 문제에 대한 접근법으로 가상노드 제시

### 가상 노드

- 실제 노드 또는 서버를 가리키는 노드로, 하나의 서버는 링 위에 여러 개의 가상 노드를 둘 수 있음.

- 가상 노드 갯수를 늘리면 키의 분포는 점점 더 균등해짐.

### 안정 해시 사용 시의 장점

- 서버가 추가되거나 삭제될 때 재배치되는 키의 수가 최소화됨.

- 데이터가 보다 균등하게 분포하게 되므로 수평적 규모 확장성 달성이 쉬움.

- 핫스팟 키 문제 줄임. 단, 특정한 샤드에 대한 접근이 지나치게 빈번하면 서버 과부하 문제 발생 가능성 높음.

- Dynamo DB, Apache Cassandra, Discord 채팅 앱, 아카마이 CDN, 매그레프 네트워크 분산기.

### ChatGPT에게 좀 더 쉽게 개념 설명해달라고 질의응답한 결과

```
Me > 컴퓨터 공학 기법 중 안정 해시 설계에 대한 개념을 어린아이도 이해할 수 있게 쉽게 설명해주고, 특정 IT 기업에서 사용하는 use case를 함께 알려줘.

ChatGPT > 안정 해시(Consistent Hashing)는 컴퓨터 공학에서 널리 사용되는 기법 중 하나입니다. 이를 어린아이도 이해할 수 있게 쉽게 설명해 보겠습니다.

상상해 보세요. 여러분이 가지고 있는 장난감들을 여러 개의 상자에 나눠 담아야 한다고 합시다. 우리의 목표는 장난감들을 공평하게 나눠서 각 상자에 넣는 것이에요. 그런데 나중에 새로운 상자를 추가하거나 상자 하나를 없애야 할 때가 있어요. 이때, 우리는 가능한 적은 수의 장난감을 다른 상자로 옮기고 싶어요. 왜냐하면 모든 장난감을 다시 정리하는 것은 시간이 많이 걸리기 때문이죠.

안정 해시는 바로 이 문제를 해결해주는 방법이에요. 이 방법을 사용하면, 새로운 상자를 추가하거나 없앨 때, 오직 몇 개의 장난감만 다른 상자로 옮겨야 해요. 나머지 장난감들은 그대로 둘 수 있어요. 이렇게 하면, 장난감들을 다시 정리하는 데 드는 시간과 노력을 크게 줄일 수 있어요.

이제, 안정 해시의 특정 IT 기업에서의 사용 사례를 살펴보겠습니다. 구글, 아마존, 넷플릭스와 같은 대형 IT 기업들은 이 기법을 사용해요. 예를 들어, 넷플릭스는 전 세계에 걸쳐 수많은 서버를 운영하고 있어요. 넷플릭스는 안정 해시를 사용하여 각 사용자의 데이터를 서버에 효율적으로 분배해요. 이렇게 함으로써, 서버를 추가하거나 제거할 때, 사용자 데이터를 다른 서버로 옮기는 작업을 최소화할 수 있어서, 네트워크 트래픽과 처리 시간을 줄일 수 있어요.

예를 들어, 넷플릭스에서 사용자들이 영화나 TV 쇼를 스트리밍할 때, 이 서비스는 사용자의 위치와 가장 가까운 서버에서 비디오를 제공해야 해요. 만약 어떤 지역의 서버에 문제가 생겨서 그 서버를 교체해야 한다면, 안정 해시 덕분에 해당 서버에 저장된 모든 비디오를 다른 서버로 옮길 필요가 없어요. 오직 일부 비디오만 새 서버나 다른 서버로 옮기면 되기 때문이죠. 이렇게 함으로써 넷플릭스는 서비스 중단 시간을 최소화하고, 사용자에게 빠르고 효율적인 스트리밍 서비스를 제공할 수 있습니다.

이처럼 안정 해시는 대규모 네트워크에서 데이터를 효율적으로 관리하고, 서비스 중단을 최소화하는 데 매우 유용한 기법입니다.
```
