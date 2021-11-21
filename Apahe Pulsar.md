## Apache Pulsar

### 요약
- Pulsar는 Apache Kafka를 대체할 수 있다
- Pulsar는 대부분의 케이스에서 Kafka보다 벤치마크 성능이 좋으나, 케이스에 따라 Kafka가 뛰어남
- Pulsar는 `Zookeeper + BookKeeper`를 사용함
- Kafka는 `Pull`형태의 동작이며, Apache Pulsar는 `Push`형태의 동작
  - Pull: Consumer가 broker로부터 데이터를 가져가는 형태
  - Push: Broker가 Consumer에게 데이터를 밀어넣는 형태
- Client API, Community 측면에서 Kafka가 Pulsar보다 우세
- Pulsar의 경우 Presto SQL 엔진 지원
- Pulsar는 replication에 적합함
  - `Kafka`의 비해 클러스터 복제를 원천적으로 지원
- Pulsar는 k8s와 같은 클라우드 환경에서 더 잘 활용 가능
  - 스케일링 및 proxy layer 등을 지원

### Broker Architecture
- Kafka
  - broker로부터 consumer에게 메세지가 전달되는 구조(pulled from the Kafka broker)
- Pulsar
  - broker가 consumer에게 밀어내는 구조(pushed to the subscribing consumer)
  - topic의 수에 이점 존재 
    - kafka의 경우 broker당 4K의 파티션 제한
    - 전체 클러스터에 걸쳐 200K의 파티션 제한 존재
    - Pulsar는 그런 제한이 존재하지 않음
    - 데이터가 broker내가 아닌, 외부적으로 [BookKeeper](https://bookkeeper.apache.org/)에 저장
    - topic 확장에 제약이 없음

### Apache Zookeeper
- 두 클러스터 모두, 클러스터 조정에 사용함
- 물론 Kafka에서는 이 내용이 제거될 예정
  - 자체 운영

### Multi Data center Replication
- Pulsar
  - 즉시 지리적 복제를 제공
  - 복제된 클러스터는 여러 데이터 센터에 걸쳐 생성 가능
  - 메세지를 복제하고 승인시까지 로컬 클러스터에서 사용하지 못하도록 차단 가능
- Kafka의 경우 `Mirror maker 2`와 `Confluent Replicator`를 사용하여 복제는 가능
  - Mirror maker를 사용할경우, 시간이 오래 걸림
  - Confluent의 경우 라이선스 구매가 필요
- 복제된 Kakfa에서는 offset 처리가 어려움
  - Confluent의 유료 툴(multi DC operation)이 지원될 예정이긴 함

### Service Discovery
- Kafka
  - configuration, bootstrap, broker list 등의 설정 정보를 모두 알아야 함
- Pulsar
  - proxy layer를 제공(클러스터에 주소를 지정할 수 있는 프록시 계층)
  - 브로커에 직접 접근이 불가한 k8s등으로 구성시에 큰 이점
  - 원하는 만큼의 Pulsar proxy를 실행 가능
  - 로드 밸런서를 사용하여 단일 지점 엑세스가 가능
  - Cloud based deployment, 클러스터 쉽게 관리 및 액세스 가능

### Scaling Clusters
- Kafka
  - broker 추가는 쉬운일이 아님
  - 클러스터 추가 이후, 메세지 데이터를 새 broker로 다시 분할하고, 복제하는 수동 process 수행 필요
  - 메세지 볼륨에 따라 시간이 많이 걸릴 수 있음
- Pulser
  - Bookkeeper 사용
  - 승인되지 않은 메세지, 복제 및 메세지 지속성을 broker로부터 분리
  - broker를 추가하지 않고, 필요한 만큼 Bookkeeper 인스턴스를 추가하면 됨
  - 디스크에 데이터를 쓰지 않고, memory에 있는 비영구적은  항목을 사용할 수 있는 옵션 제공
    - 단, Pulsar broker가 cluster에서 연결이 끊어질 경우
    - 브로커에 저장되거나, 메세지 손실이 발생할 수 있음

#### 소비 방법의 차이
- Kafka
  - 불변 로그, 소비자가 읽을 수 있는 가장 최근의 메세지인 `Offset Controlling`을 지원
  - offset commit을 원하지 않는다면, Kafka client API를 사용하여 처리 가능
- Pulsar
  - 두가지 방법의 Consume 지원
    - The Consumer Interface
      - 로그에서 사용가능한 최신 메세지를 읽은 Kafka consumer와 동일한 방식
    - The Reader Interface
      - App이 로그의 순서와 관계없이, 특정 메세지를 읽을 수 있음

### Reading and Persisting Data from Exceternal Sources
- Kafka
  - Kafka Connect System이 데이터를 토픽으로 소싱하거나 싱크대로 데이터를 유지하는 편리한 방법 제공
- Pulsar
  - Pulsar IO라는 유사한 방법 지원
  - 데이터를 수집하거나 유지하는 source/sink 방식이 동일
  - 단, External source에 대한 지원이 단점
- Pulsar IO에 비해 Kafka Connect에서 지원되는 벤더가 더 많음

### SQL Type Operation
- 메세지 스트림에 SQL과 같은 쿼리 사용
- Kafka
  - Confluent Community License로 지원
- Pulsar
  - Presto SQL 엔진을 사용
  - 스키마 레지스터에 저장된 스키마를 사용하여 메세지 쿼리

### Clients - Producing and Consuming
- Kafka의 지원범위가 Pulsar보다 넓음
- ![image](https://user-images.githubusercontent.com/10006290/142751072-e5d0402b-a84b-4002-9702-f1385b11a8e0.png)


### 참고
- https://pandio.com/blog/pulsar-vs-kafka/
- https://digitalis.io/blog/technology/apache-kafka-vs-apache-pulsar/
- https://www.confluent.io/kafka-vs-pulsar/