# chapter 4. 부호화와 발전
- 하위호환성: 새로운 코드는 예전 코드가 기록한 데이터를 읽을수 있어야 한다.
- 상위호환성: 예쩐 코드는 새로운 코드가 기록한 데이터를 읽을수 있어야 한다.
### 데이터 부호화 형식
#### 언어별 형식
언어별로 각각의 Serialization 방법이 있지만, 다른 언어에서 처리하는 것이 어렵고, 보안 문제를 일으 킬수 있다.
그래서 일반적으로는 좋지 않은 선택이다.

#### JSON, XML, 이진변형
##### JSON, XML
- human readable
- XML의 경우 number와 number string을 구분 할수 없다
- JSON의 경우 정수와 부동소수점 수를 구별할 수 없고, 정밀도를 지정하지 않는다.
  - 트위터에서는 Json number, String 둘다 보내는 방법을 이용
- Schema가 따로 필요 없음.

#### 이진부호화
- Schema가 따로 필요하다
##### Thrift, Protobuf
- Schema에 있는 Field Tag를 통해 데이터 맵핑
- 새로운 Field Tag를 추가함으로써 Schema를 발전시킬수 있다.
  - optional 혹은 required & default value 지정
  - required는 필드 삭제 못함.
  - Protobuf에는 array 데이터타입 대신 repeated 표시자가 있다. repeated -> optional로 변경 가능

##### Avro
- Schema에 Field Tag가 없다. 
- 쓰기 스키마와 읽기 스키마를 따로 운용 가능하고, 이름으로 필드 맵핑.
  - 읽기 스키마에서 이름 변경하고 싶으면 alias 사용
- Read시 쓰기 스키마를 어떻게 알수 있나?
  - 많은 레코드가 있는 대용량 파일: 파일의 시작부분에 한번만 스키마 포함
  - 개별적으로 기록된 레코드를 가진 데이터베이스
    - 레코드에 버전을 추가하고, 버전별로 스키마를 DB에 관리
  - 네트워크 연결을 통해 레코드 보내기
    - 두 프로세스가 양방향 네트워크 연결을 통해 통신할 때 연결 설정에서 스키마 버전 합의
- Field Tag가 없기 때문에, 스키마를 동적으로 생성할수 있고, RDBMS 데이터를 Avro로 쉽게 생성할수 있다.

##### 스키마의 장점
- Field name 생략 가능하기 때문에 데이터 사이즈를 크게 줄일수 있다.
- 스키마는 유용한 문서화 형식이다. 복호화를 할 때 스키마가 필요하기 때문에 스키마가 최신 상태인지를 확인할수 있다.
- 스키마 데이터베이스를 유지하면 스키마 변경이 적용되기 전에 상위, 하위호환성 체크가 가능
- 정적 타입 언어인 경우 스키마로부터 코드를 생성하는 기능을 이용하여 컴파일 시점에 타입체크를 할수 있다.

### Data Flow 모드

#### 데이터베이스를 통한 Data Flow
DB에 데이터를 쓰기, 읽기
- New version 쓰기, Old Version 읽기 후 쓰기 시 데이터 유실될수 있다는 점을 고려해야 한다.

#### 서비스를 통한 Data Flow: REST, RPC
웹서비스
- REST: HTTP의 원칙을 토대로 한 설계 철학 -> Restful API
- SOAP: WSDL을 통해서 서비스
RPC: 원격 네트워크 서비스 요청을 같은 프로세스 안에서 함수나 메서드 호출하는 것처럼 동일하게 사용 가능(location transparency)
- 로컬함수는 예측 가능하지만, 네트워크 함수는 예측이 어려워서 문제가 많다.
  - timeout
  - retry시의 중복 처리 -> 멱등성 적용 필요
  - 지연 시간
  - 객체 전달의 비효율성
  - 다른 언어와의 데이터타입 불일치 문제있
차세대 RPC 프레임워크들은 원격 요청이 로컬함수와 다르다는 사실을 분명히 한다.

#### Message 전달 Data Flow
- 비동기 메시지 전달 시스템
- 
##### Message Broker
- Message Broker를 두고 비동기로 메시지를 전달
  - - ex) Kafka, RabbitMQ
- publish, subscribe
- subscriber가 문제가 발생하더라도 Message Broker가 버퍼처럼 동작
- 재전송이 가능하기 때문에 메시지 유실 방지 가능
- 송신자가 수신자의 ip 주소나 포트 번호를 알 필요가 없다.
- 하나의 메시지를 여러 수신자로 전송 가능
- 논리적으로 송신사, 수신자 분리


##### 분산 액터 프레임워크
- Actor model에 기반
- Actor는 메시지 전달을 보장하지 않는다.
- Actor model은 단일 프로세스 안에서도 메시지가 유실될수 있다고 이미 가정하기 때문에 Location transparency가 RPC보다 actor model에서 더 잘 작동한다.
- ex) 아카, 올리언스, 얼랭 OTP