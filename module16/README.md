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
!(./images/messageQue디렉토리.png)

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



## 3. MSA에서 메시지 큐 구현 시 고려사항

MSA에서 메시지 큐를 구현할 때 다음 사항들을 고려해야 합니다:

1. **메시지 포맷**: 일관된 메시지 구조와 직렬화 방식을 선택합니다.
2. **메시지 순서**: 순서가 중요한 경우, 순서 보장 메커니즘을 구현합니다.
3. **멱등성**: 중복 메시지 처리에 대비한 멱등성 처리 로직을 구현합니다.
4. **장애 처리**: 메시지 손실 방지와 재시도 메커니즘을 구현합니다.
5. **확장성**: 메시지 양 증가에 대비한 확장 계획을 수립합니다.
6. **모니터링**: 메시지 큐의 상태와 처리 현황을 모니터링합니다.
7. **보안**: 메시지 암호화 및 접근 제어를 구현합니다.



## 4. egovframe-msa-edu-kenu 프로젝트의 모니터링 방법

egovframe-msa-edu-kenu 프로젝트에서는 Spring Boot Actuator와 Prometheus를 사용하여 모니터링을 구현하고 있습니다.

### 디렉토리 구조
!(./images/모니터링디렉토리.png)

 
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
