
### Kafka SASL/PLAINTEXT 인증

Kafka에서 허가되지 않은 사용자의 접근을 막기 위해 사용하는 방식 중 JAAS 기반의 SASL을 사용

New Consumer(Kafka Consumer)에서 SASL을 설정하는 방법은 다음과 같다.

#### `Kafka Server`

- server_jaas.conf

```
KafkaServer {
	org.apache.kafka.common.security.plain.PlainLoginModule required
	username="admin"
	password="admin-secret"
	user_admin="admin-secret"
	user_sphong="sphong-secret";
};
```

`-Djava.security.auth.login.config=/sever_jaas.conf/path/` 을 추가하여 Kafka Server를 실행한다.



- server.conf

  ```
   listeners=SASL_SSL://host.name:port
   security.inter.broker.protocol=SASL_PLAINTEXT
   sasl.mechanism.inter.broker.protocol=PLAIN
   sasl.enabled.mechanisms=PLAIN
  ```

#### `Kafka Client`

- consumer.properties

  ```
  security.protocol=SASL_PLAINTEXT
  sasl.mechanism=PLAIN
  sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="sphong" \
  password="alice-secret";
  ```


#### Kafka Old Consumer vs New Consumer

Kafka Consumer는 Old Consumer와 New Consumer로 나누어진다. 두 컨슈머의 차이는 offset 관리를 Kafka에서 하는지, Zookeeper에서 하는지이다. Kafka 0.9 버전부터 성능상의 이슈로  Consumer의 Offset 저장을 Kafka Topic `_consumer_offsets`에 보관하도록 변경되었다.


#### Graylog Issue
Graylog는 Old / New Kafka Consumer를 지원하고 있지만, 두 버전의 Client 모두 0.9 버전이다. SASL/PLAINTEXT 인증 방식은 kafka 0.10 버전부터 적용되었기 때문에 위의 설정을 적용할 경우, `org.apache.kafka.common.security.plain.PlainLoginModule`을 찾을 수 없다는 에러가 발생한다.

다른 시도는 Old Consumer를 사용하고 Zookeeper의 SASL/PLAINTEXT 방식을 사용하는 방법. 
이는 Zookeeper의 인증은 수행하지만 Old Kafka Consumer를 Graylog에서 사용하는 경우 consumer.properties를 override 할 수 없기 때문에, security.protocol을 SASL_PLAINTEXT 변경하지 못하고 Default 값인 PLAINTEXT를 사용하게 된다. 이에 따라 `kafka.common.BrokerEndPointNotAvailableException: End point PLAINTEXT not found for broker 0` 라는 에러가 발생하게 된다.

#### 참고자료
https://kafka.apache.org/documentation/#security_sasl_plain
