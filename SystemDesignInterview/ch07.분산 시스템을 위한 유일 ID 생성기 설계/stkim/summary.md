# Ch 07. 분산 시스템을 위한 유일 ID 생성기 설계

- 가장 기초적인 ID 생성 방식은 auto_increment 사용

  - 분산 시스템에서 안먹힘

- 설계 조건

  - ID는 유일해야함.
  - ID는 숫자로만 구성되어야함.
  - ID는 64비트로 표현될 수 있는 값이어야 한다.
  - ID는 발급 날짜에 따라 정렬 가능해야한다.
  - 초당 10,000개의 ID를 만들 수 있어야한다.

- ID만드는 방법들

  - 다중 마스터 복제 (multi-master replication) :
    - 현재 사용 중인 서버 갯수 k 만큼 auto_increment 증가
    - 여러 데이터센터에 사용 불가능
    - ID 유일성은 보장되겠지만, 그 값이 시간 흐름에 맞춰 커지도록 보장 불가.
    - 서버 추가 또는 삭제할 때 잘 동작하는지 보장 안됨.
  - UUID (Universally Unique Identifier)
    - 128비트 수로 컴퓨터 시스템에 저장되는 정보를 유일하게 인식하기 위한 방법.
    - UUID 만드는 것은 단순하고, 규모 확장은 쉬움.
    - 그러나 128비트로 길고, 요구 사항은 64비트임.
    - 시간순 정렬이 안되고, 숫자가 아닌 값 포함될 가능성 높음.
  - 티켓 서버 (Ticket Server)
    - auto_increment 기능을 갖춘 DB 서버 (즉, 티켓서버)를 중앙 집중형으로 하나만 사용하는 것이다.
    - 유일성이 보장되는 오직 숫자로만 구성된 ID 쉽게 만들 수 있음.
    - 구현이 쉽고, 중소 규모 애플리케이션에 적합.
    - 티켓서버가 SPOF 되기 쉬움.
  - 트위터 스노플레이크 (Twitter Snowflake)
    - ID 구조를 여러 절로 분할함.
    - 사인 비트 : 1비트. 나중을 위해 남겨두고 음수, 양수 분별 가능
    - 타임스탬프 : 41비트.
    - 데이터센터 : 5비트. 데이터센터당 32개 서버 사용 가능.
    - 일련번호 : 12비트 할당. ID를 서버에서 생성할 때마다 1씩 증가. 1밀리초가 경과할 때마다 0으로 초기화.

![Twitter Snowflake](https://github.com/ttlqudan/WoowahanStudy/assets/40455392/2da39bc4-2595-4988-9582-1dbe9ccc1bc8)
