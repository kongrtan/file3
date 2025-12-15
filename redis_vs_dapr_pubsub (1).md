# Redis Pub/Sub vs Dapr Pub/Sub Comparison

| 항목 | Redis Pub/Sub (직접) | Dapr Pub/Sub (Redis Component) |
|------|-------------------|------------------------------|
| 경로 | 앱 → Redis → 앱 | 앱 → Dapr 사이드카 → Redis → Dapr 사이드카 → 앱 |
| 지연 시간 | μs~수백 μs | ms 수준 (1~10배 정도) |
| 메시지 보장 | 없음 | 있음 (옵션 설정 가능) |
| 재시도 / 영속화 | 없음 | 있음, DLQ, 재시도 기능 제공 |
| 구성 / 유연성 | 단순, 제어 가능 | 사이드카 기반, 멀티 플랫폼 통합 가능 |
| 확장성 | Pod 수 증가 시 Redis 채널 구독 필요 | Pod 수 증가에도 자동 처리, Publisher/Subscriber decoupling |
| 운영 복잡도 | 낮음 | Dapr 사이드카 필요, 약간 높음 |
| 장점 | 초저지연, 단순 구조 | 안정적 메시지 전송, 재시도, 멀티 클라우드, Kafka 등 다른 Pub/Sub 시스템으로 변경 가능 |
| 단점 | 메시지 손실 가능, 재시도 없음 | Redis 대비 약간의 지연, 구조 복잡 |

**추가 장점 (Dapr)**
- Pub/Sub backend를 Redis에서 Kafka, RabbitMQ 등 다른 시스템으로 쉽게 변경 가능
- Publisher와 Subscriber 코드 분리로 애플리케이션 코드 단순화
- Pod restart, 스케일링, 멀티 언어 지원 용이

