# 메시지 큐

## 0. 메시지 지향 미들웨어(MOM)

응용 소프트웨어 간의 비동기적 데이터 통신을 위한 소프트웨어로, 메시지를 전달하는 과정에서 다음과 같은 장점을 가집니다:

- 메시지의 백업 기능을 유지함으로 지속성을 제공하기에 송수신 측은 동시에 네트워크 연결을 유지할 필요가 없습니다.
- 메시지 라우팅을 수행하기에 하나의 메시지를 여러 수신자에게 배포가 가능합니다.
- 송수신 측의 요구에 따라 전달하는 메시지를 변환할 수 있습니다.

## 1. 메시지 큐

큐 자료구조를 채택해서 메시지를 전달하는 시스템이며, 메시지 지향 미들웨어(MOM)를 구현한 시스템입니다.

메시지 큐는 메시지를 발행하고 전달하는 부분인`Producer`와 메시지를 받아서 소비하는 부분인`Consumer` 사이의 메시지 전달 역할을 하는 매개체입니다.

## 2. 메시지 큐의 종류
| 이름 | 특징 | 사용 시나리오 | 사용 예시 |
|------|------|---------------|-----------|
| Kafka | • 초고속 대용량 데이터 처리에 특화<br>• 데이터 유실 방지를 위한 복제 기능<br>• 확장이 용이한 분산 시스템 구조 | • 대용량 실시간 로그 및 이벤트 처리<br>• 스트림 처리 애플리케이션 | • LinkedIn: 사용자 활동 추적 및 로그 관리<br>• Netflix: 실시간 추천 시스템 및 모니터링 |
| RabbitMQ | • 다양한 메시징 방식 지원(1대1, 1대다 등)<br>• 유연한 라우팅으로 복잡한 메시지 흐름 처리 가능 | • 복잡한 라우팅이 필요한 워크플로우<br>• 다양한 시스템 간 연동 필요 시 | • Mozilla: 데이터 동기화 및 알림 시스템 |
| ActiveMQ | • JMS 완벽 지원<br>• 다양한 언어 및 프로토콜 지원 | • JAVA 기반 엔터프라이즈 애플리케이션<br>• 다중 프로토콜 지원이 필요한 환경 | • Amazon: 주문 처리 시스템 일부 |
| Redis | • 인메모리 데이터 구조로 빠른 처리 속도<br>• 단순하고 가벼운 메시징 시스템 | • 실시간 알림 시스템 구축 시<br>• 낮은 지연 시간, 처리 속도 중요 시 | • Twitter: 실시간 타임라인 업데이트 |



## 3. 메시지 브로커 VS 이벤트 브로커

메시지 큐에서 데이터를 운반하는 방식에 따라 메시지 브로커와 이벤트 브로커로 나눌 수 있습니다.

### 메시지 브로커

- 메시지 브로커는 Producer가 생산한 메시지를 메시지 큐에 저장하고, 저장된 메시지를 Consumer가 가져갈 수 있도록 합니다.
- 메시지 브로커는 Consumer가 메시지 큐에서 데이터를 가져가면 짧은 시간 내에 메시지 큐에서 데이터가 삭제되는 특징이 있습니다.
- 주로 작업 큐나 요청-응답 패턴에 적합합니다.
- ex) RabbitMQ, ActiveMQ, AWS SQS, Redis

### 이벤트 브로커

- 이벤트 브로커가 관리하는 데이터를 이벤트라고 합니다.
- Consumer가 메시지 큐에서 데이터를 가져가도 삭제되지 않으며, 필요한 경우 다시 소비할 수 있습니다.
- 메시지 브로커보다 대용량 데이터를 처리할 수 있는 능력이 있습니다.
- 실시간 스트리밍 처리와 데이터 파이프라인 구축에 적합합니다.
- ex) Apache Kafka, Apache Pulsar

## 4. 메시징 패턴
### One-way Messaging
![One-way Messaging](images/pattern-one-way.jpeg)
- point-to-point messaging
- `Producer`는 `Consumer`가 특정 시점에 메세지 검색하고 처리할 것을 기대하고 Queue에 메세지를 보냅니다
- `Consumer`는 `Queue`에서 메세지를 검색하고 처리하며, 여기서 `Producer`는 `Consumer`의 존재나 메세지가 어떻게 process 되는지 알지 못하며 `Consumer`의 응답을 기다리지 않습니다. 즉 `Consumer`에 응답에 의존적이지 않습니다.

### Request/Response Messaging
![Request/Response Messaging](images/pattern-request-response.jpeg)
- `Consumer`가 Response message를 보낼 별도의 Message Quqeue 형태의 Communication channel이 필요합니다
- `Producer`는 Reuqest Queue에 메세지를 보낸 뒤 Reply Queue로부터 Response를 기다립니다.
- `Consumer`는 메시지를 처리한 다음에 Reply Queue에 Response 메시지를 전달합니다
- 만약 Response가 설정해놓은 time interval 안에 도착하지 않는다면 Producer는 둘 중 하나를 선택할 수 있습니다:
    - 메시지를 다시 보냅니다
    - Timeout 처리를 합니다


### Pub/Sub
![Pub/Sub](images/pattern-pub-sub.jpeg)
- `Publisher`는 Topic에 메세지를 발행하고, 누가 받는지는 알 필요가 없습니다
- `Subscriber`는 관심 있는 Topic을 구독하고, 해당 Topic에 발행된 모든 메세지를 수신합니다
- 하나의 메세지가 여러 `Subscriber`에 전달될 수 있습니다(1:N)
- `Subscriber`는 언제든 구독을 시작하거나 중단할 수 있으며, `Publisher`의 동작에는 영향을 주지 않습니다.

---
# MSA에서의 메시지 큐와 모니터링

## 1. MSA에서 메시지 큐의 역할

마이크로서비스 아키텍처(MSA)에서 메시지 큐는 다음과 같은 중요한 역할을 수행합니다:

- **비동기 통신**: 서비스 간 비동기 메시지 교환을 가능하게 합니다.
- **느슨한 결합**: 서비스 간 직접적인 의존성을 줄여 유연성을 높입니다.
- **부하 분산**: 트래픽 피크 시 메시지를 버퍼링하여 시스템 안정성을 유지합니다.
- **장애 격리**: 한 서비스의 장애가 전체 시스템에 미치는 영향을 최소화합니다.
- **확장성**: 서비스의 독립적인 확장을 용이하게 합니다.



## 2. egovframe-msa-edu-kenu 프로젝트의 메시지 큐 패턴 사례

egovframe-msa-edu-kenu 프로젝트에서 메시지 큐 패턴이 적용된 사례를 살펴보겠습니다.

### 디렉토리 구조
![messageQue디렉토리](images/image1.png)

### 구현 코드

**KafkaConsumerConfig.java**
```
@Configuration
@EnableKafka
public class KafkaConsumerConfig {
    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "group-id");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```


**KafkaConsumerService.java**
```
@Service
public class KafkaConsumerService {
    @KafkaListener(topics = "reservationTopic", groupId = "group-id")
    public void consume(String message) {
        System.out.println("Received message: " + message);
        // 메시지 처리 로직
    }
}
```

**ReservePaymentService.java**
```
@Service
public class ReservePaymentService {
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void processPayment(String reservationData) {
        // 결제 처리 로직
        kafkaTemplate.send("paymentTopic", reservationData);
    }
}
```



이 예시에서는 Kafka를 사용하여 예약 및 결제 처리를 위한 메시지 큐 패턴을 구현하고 있습니다. KafkaConsumerConfig는 Kafka 소비자 설정을, KafkaConsumerService는 메시지 수신 및 처리를, ReservePaymentService는 결제 처리 및 메시지 발행을 담당합니다.

## 3. RabbitMQ를 사용한 메시지 큐 구현

### 디렉토리 구조
![RabbitMQ디렉토리](images/image3.png)

**RabbitConfiguration.java**
```
@Configuration
public class RabbitConfiguration {

    @Bean
    public Queue queue() {
        return new Queue("reservationQueue", false);
    }

    @Bean
    public TopicExchange exchange() {
        return new TopicExchange("reservationExchange");
    }

    @Bean
    public Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("reservation.#");
    }
}
```

이 설정 클래스는 RabbitMQ를 위한 기본 설정을 제공합니다:
- 'reservationQueue'라는 이름의 큐를 생성합니다.
- 'reservationExchange'라는 이름의 토픽 교환기를 생성합니다.
- 큐와 교환기를 'reservation.#' 라우팅 키로 바인딩합니다.


**ReservationService.java**
```
@Service
@RequiredArgsConstructor
public class ReservationService {

    private final RabbitTemplate rabbitTemplate;

    public void sendReservationMessage(ReservationDto reservationDto) {
        rabbitTemplate.convertAndSend("reservationExchange", "reservation.created", reservationDto);
    }

    @RabbitListener(queues = "reservationQueue")
    public void receiveReservationMessage(ReservationDto reservationDto) {
        // 메시지 처리 로직
        System.out.println("Received reservation: " + reservationDto);
    }
}
```

이 서비스 클래스는 RabbitMQ를 통한 메시지 송수신을 담당합니다:
- sendReservationMessage 메소드는 예약 정보를 RabbitMQ로 전송합니다.
- receiveReservationMessage 메소드는 RabbitMQ로부터 메시지를 수신하고 처리합니다.


**application.yml**
```
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```
이 설정 파일은 RabbitMQ 연결 정보를 제공합니다.

이 구현에서는 RabbitMQ를 사용하여 예약 시스템의 메시지 큐를 구현하고 있습니다. 예약이 생성되면 메시지가 RabbitMQ로 전송되고, 다른 서비스에서 이 메시지를 수신하여 처리할 수 있습니다. 이러한 방식으로 MSA 환경에서 서비스 간 비동기 통신을 구현하여 시스템의 확장성과 유연성을 향상시킬 수 있습니다.


## 4. MSA에서 메시지 큐 구현 시 고려사항

MSA에서 메시지 큐를 구현할 때 다음 사항들을 고려해야 합니다:

1. **메시지 포맷**: 일관된 메시지 구조와 직렬화 방식을 선택합니다.
2. **메시지 순서**: 순서가 중요한 경우, 순서 보장 메커니즘을 구현합니다.
3. **멱등성**: 중복 메시지 처리에 대비한 멱등성 처리 로직을 구현합니다.
4. **장애 처리**: 메시지 손실 방지와 재시도 메커니즘을 구현합니다.
5. **확장성**: 메시지 양 증가에 대비한 확장 계획을 수립합니다.
6. **모니터링**: 메시지 큐의 상태와 처리 현황을 모니터링합니다.
7. **보안**: 메시지 암호화 및 접근 제어를 구현합니다.



## 5. egovframe-msa-edu-kenu 프로젝트의 모니터링 방법

egovframe-msa-edu-kenu 프로젝트에서는 Spring Boot Actuator와 Prometheus를 사용하여 모니터링을 구현하고 있습니다.

### 디렉토리 구조
![모니터링디렉토리](images/image2.png)

 
**ActuatorConfig.java**
```
@Configuration
public class ActuatorConfig {
    @Bean
    public MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
        return registry -> registry.config().commonTags("application", "message-queue-consumer");
    }
}
```


**#application.yml**
```
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```        

**prometheus.yml**
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'spring-actuator'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']
```      



이 설정을 통해 Spring Boot Actuator는 애플리케이션의 상태, 메트릭 등을 노출하고, Prometheus는 이를 수집하여 모니터링할 수 있습니다. Grafana와 같은 도구를 추가로 사용하면 수집된 데이터를 시각화할 수 있습니다.
이러한 모니터링 설정을 통해 메시지 큐의 처리량, 지연 시간, 오류율 등을 실시간으로 관찰하고 분석할 수 있습니다.      
