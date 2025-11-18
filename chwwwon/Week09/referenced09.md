# 9주차 발표자료

[Designing a Scalable Notification System: From HLD to LLD](https://medium.com/%40tanushree2102/designing-a-scalable-notification-system-from-hld-to-lld-e2ed4b3fb348)

# **Designing a Scalable Notification System: From HLD to LLD**

---

- 확장 가능한 알림 시스템 설계 방법
- HLD 관점의 큰 구조 이해
- LLD 관점의 큰 구조 이해
- 아키텍처 결정 요소, 확장성 패턴, 실제 코드 등

### Functional Requirements (기능 요구사항)

- Email, SMS, Push **알림 발송 지원**
- 알림 유형별 **사용자 설정(수신 동의/거부)** 지원
- 실패 시 **재시도 처리**
- **예약 발송(미래 시점 알림)** 지원
- **전송 결과 로그** 수집 (분석 및 감사를 위해)

### Non-Functional Requirements (비기능 요구사항)

- **높은 확장성(Scalability)** & **고가용성(Availability)**
- 실시간 발송을 위한 **낮은 지연(Latency)**
- **관측 가능성(Observability)** — 로그, 메트릭, 알림
- 향후 **새로운 채널(예: 카카오 알림톡)** 확장 용이성

### System Requirements and Assumptions (**시스템 요구사항 및 가정)**

- **처리량(Throughput)**: 피크 시간대에 **초당 10,000건**의 알림 처리
- **채널(Channels)**: Email, SMS, Push 등 **멀티 채널 발송** 지원
- **복원력(Resiliency)**: 알림 전송 실패 시 **재시도**, 계속 실패하면 **DLQ(Dead Letter Queue)** 로 이동
- **확장성(Scalability)**: **수평 확장(Horizontal Scaling)** 가능하며, 채널별 워커를 **독립적으로 확장**할 수 있어야 함

## 1. Components Overview (**구성 요소 개요)**

---

1. **Notification API: 사용자 알림 요청이 들어오는 시스템의 진입점**
    - 사용자 선호 설정, 메시지 내용, 타깃 채널 등이 포함된 요청을 처리
    - 요청을 **검증 및 정규화**한 후
    - **메시지 큐(Kafka/SQS 등)** 에 알림 메시지를 퍼블리시
2. **Validator & Router**
    - 요청 **유효성 검사**
    - 필수 값 확인
    - 알림을 적절한 **큐 또는 채널 워커로 라우팅**
3. **Message Queue (Kafka / SQS)**
    - **프로듀서(API)** 와 **컨슈머(워커)** 간 **디커플링**
    - 장애 대응력 향상
    - 시스템 확장성 강화
4. Workers per Channel
    - 큐에서 메시지 소비
    - 외부 서비스(Twilio/SMS, SendGrid/Email 등) 호출하여 실제 발송 수행
    - 채널별 독립 확장 가능
5. Retry Mechanism & Dead Letter Queue
    - 전송 실패 시 **지수 백오프(Exponential Backoff)** 기반 재시도
    - 반복 실패 시 **DLQ로 이동**하여 모니터링, 장애 원인 분석 또는 수동 처리 대상이 됨

## 2. Back-of-the-Envelope Calculations (대략적 용량 계산)

---

1. **초당 알림 수 (NPS: Notifications per Second)**
    - 초당 **10,000건의 알림**을 처리한다고 가정
    - 워커(Worker) 하나가 **초당 100건**을 처리할 때 → 부하를 감당하려면 **100개의 워커**가 필요
2. **큐 지연 시간 (Queue Latency)**
    - Kafka나 SQS를 사용할 때
    - 메시지당 **지연 시간(latency)이 약 10ms**라고 가정
    - 큐가 적절하게 설정되어 있다면
        
        → 초당 10,000건의 알림도 **큰 병목 없이 처리 가능**
        
3. **워커 스케일링 (채널별)**
    - 각 알림 채널(Email, SMS, Push)마다 **별도의 워커 세트**가 필요
    - 전체 10,000 NPS를 3개 채널에 나누면 → 채널당 약 **3,333 NPS**
    - 워커 하나가 **초당 100건**을 처리한다 가정 → 채널당 **약 34개 워커 필요**

### 워커 수를 줄이는 방법

1. **배치 전송이 가능한가?**
    
    : 알림을 배치로 묶어서 보내는 경우
    
    - 워커 하나가 **이론적으로 초당 1,000건** 처리 가능 → 약 4개의 워커로 사용 가능
    - 실제 알림 서비스(Firebase Push, Twilio Messaging Services)는 대량 전송 API 지원
2. **메시지가 균일하게 오는가?**
    
    : 트래픽은 burst 현상 자주 발생
    
    - 과도한 자원 깔아두기 금지
    - 자동 스케일링, 일시적 큐 적체 처리 가능
3. **우리가 보는 수치는 최대치(peak)인가, 평균(average)인가?**
    
    : NPS를 시스템 피크값보다 낮게 설계
    
    - retry, backpressure, queue buffering 등의 메커니즘으로 보완
4. **워커가 멀티스레드 또는 비동기(async)로 동작 가능한가?**
    
    : 하나의 워커가 여러 작업 동시 처리시 필요 워커 인스턴스 개수 절약 가능
    
    - 비동기 I/O 이용하여 외부 API 응답 대기 중 다른 작업 처리 가능
    - 동일한 NPS를 훨씬 적은 수의 워커로 커버 가능

5. **당신의 지연 허용 범위(Latency Tolerance)는?**

: 혀용 가능한 딜레이 정도를 고려

- 실시간 필요X → 알림 버퍼링 후 천천히 전송 가능
- 즉시성 중요 → 더 많은 리소스 제공

## 3. 필요 API 목록

---

1. **POST /notifications:** 클라이언트가 **알림 발송**을 요청하는 API
    - 전달 페이로드에 사용자 선호 정보, 메시지 내용, 발송 채널 등 포함
    
    ```java
    {
    	"userId": "user123",
    	"channels": ["email", "sms"],
    	"message": "Your order has shipped",
    	"notificationType": "order_shipped",
    	"preferences": {"email": true, "sms": false}
    }
    ```
    
2. **GET /notifications/{notificationId}:** 특정 알림의 **전송 상태(Status) 조회** API
    - 응답에 전송 상태(성공/실패/대기 등), timestamp, 사용된 채널 포함
3. **POST /retry:** 전송 실패한 알림에 대해 **재시도(retry)** 를 트리거하는 API
    - 내부 관리용(Admin) API로도 운영됨

# HLD (High-Level Design)

---

## 알림 처리 기본 흐름

> API와 발송 로직을 분리하여 **장애 격리**, **확장성 확보**, **지연 최소화**
> 

1. **Client**
    
    : 사용자가 알림을 요청
    
    - 예: 주문 완료, OTP 발송
2. **Notification Service**
    
    : 알림 요청을 받아 **검증 후 큐에 넣음**
    
3. **Queue**
    
    : 알림 메시지 버퍼링하여 비동기 처리 가능하게 함
    
4. **Workers (Email / SMS / Push)**
    
    : 각 채널별 워커가 큐에서 메시지를 읽어 실제 제3자 알림 서비스 호출해 메시지 전송
    

![알림 처리 기본 흐름](9%EC%A3%BC%EC%B0%A8%20%EB%B0%9C%ED%91%9C%EC%9E%90%EB%A3%8C/image.png)

알림 처리 기본 흐름

## 안정성과 중복 방지 흐름

> **중복 방지 + 재시도 + 장애 분리**로 신뢰도 높은 알림 시스템 구현 가능
> 

1. **Rate Limiter**
    
    : 너무 많은 요청이 한 번에 들어오면 **속도 제어**
    
2. **Redis Bloom Filter (Deduplication)**
    
    : 동일 알림 **중복 전송 방지**
    
3. **Kafka Producer → Kafka → Kafka Consumer**
    
    : 메시지를 안전하게 **전달·재처리**할 수 있는 비동기 스트림 처리 backbone
    
4. **DLQ (Dead Letter Queue)**
    
    : 계속 실패한 알림을 분리하여 **후속 조사/수동 처리**
    

![안정성과 중복 방지 흐름](9%EC%A3%BC%EC%B0%A8%20%EB%B0%9C%ED%91%9C%EC%9E%90%EB%A3%8C/image%201.png)

안정성과 중복 방지 흐름

# LLD (Low-Level Design)

---

**알림 시스템을 객체 지향적으로 설계하는 기본 골격**

## 주요 아이디어

---

1. **Channel enum**
    - 채널 추상화. 알림 채널을 enum으로 정의하여 **확장성** 확보
2. **Notification class**
    - Notification 인터페이스로 공통 기능 묶기
    - **다형성(polymorphism)** 활용
        
        → 각 알림 채널을 같은 타입으로 다룸
        
        - 데이터 필드가 달라도 공통 API로 처리 가능
3. **Notification Sender**
    - Sender 인터페이스로 실제 발송 책임 분리
        
        → 단일 책임 원칙(SRP) 구현
        
4. **Notification Sender Factory**
    - Factory 패턴으로 채널별 Sender 선택
    - 채널이 늘어나도 기존 코드 수정 최소화 → **OCP(Open Closed Principle)** 준수
    - 외부 의존성 교체도 쉬움 (예: Twilio → 다른 SMS 서비스)
5. **Notification Dispatcher**
    - Dispatcher가 라우팅 역할 수행
    - Notification은 Dispatcher에게 보내기만 하면 됨
    - Dispatcher가 내부적으로 **올바른 Sender를 찾아 전송**
    
    → **추상화 수준**을 높여 유지보수성 향상
    
6. **Usage**
    - 확장성 내장 설계
    - 새로운 채널을 추가해도
        
        → Notification 구현체 + Sender 구현체만 만들면 됨
        
    - 나중에 **비동기 처리 / Retry / DLQ / Observability**도 쉽게 붙일 수 있는 구조

## 큐 기반 비동기 설계 사용 이유

---

| 효과 | 설명 |
| --- | --- |
| **디커플링** | API와 워커가 서로 독립적으로 동작 |
| **수평 확장성** | 트래픽 증가 시 워커만 늘리면 됨 |
| **버퍼링** | 트래픽 폭주 시 메시지를 큐에 적재 |
| **내결함성** | 워커 장애 시 메시지는 큐에 남아 재처리 가능 |

## 확장성 전략 (Scalability)

---

**채널별 수평 확장 (Channel-wise Horizontal Scaling)**

- Email / SMS / Push **각 채널 워커를 독립적으로 확장**
- 예: SMS 트래픽이 늘면 SMS 워커만 확장

**Kafka Topic Partitioning**

- 사용자 ID, 채널, 우선순위 기준으로 **토픽 파티션 분할**
- 동시에 처리해도 충돌 없이 높은 처리량 확보

**재시도 큐 / DLQ 활용**

- 전송 실패 메시지는
    
    → **Retry Queue**로 이동 후 지수 백오프 재시도
    
    → 계속 실패하면 **DLQ**로 이동하여 후속 처리
    

**Rate Limiting & Throttling**

- 채널별 발송 제한 적용
    
    (예: SMS는 통신사 규제로 분당 X건 이상 발송 금지)
    

**예약 알림 처리**

- 예약 정보를 별도 DB/토픽에 저장
- cron 기반 워커가 지정된 시점에 처리

## 도전 과제 및 고려사항

---

| 항목 | 설명 |
| --- | --- |
| **전달 보장(Delivery Guarantees)** | 100% 보장 불가 채널 존재 → 재시도 & 대체 채널 전략 필요 |
| **비용 관리** | SMS/Email 외부 서비스 비용 증가 → 발송 정책/모니터링 필요 |
| **모니터링 & 알림** | 큐 길이, 실패율, 워커 상태 등 Prometheus/Grafana로 관측 및 알림 |

## 일관성 고려사항 (Consistency)

---

**Eventual Consistency (결과적 일관성)**

- 분산 시스템에서는 **가용성과 성능을 위해 선택되는 방식**
- 예시:
    - Kafka에 적재된 알림은 워커가 다운되어 있어도
        
        → 복구 후 **언젠가는** 처리됨
        
    - 전송 상태 업데이트가 **즉시 반영되지 않을 수 있음**

**Idempotency (멱등성)**

- **중복 발송 방지**가 핵심
- 구현 방식:
    - 알림마다 **고유한 requestId** 부여
    - 워커가 **이미 처리한 알림인지 확인** 후 발송

## 전달 보장 (Delivery Guarantees)

---

**최소 한 번(at-least-once) 전달 보장을 목표로 함**

- Kafka/SQS에 저장된 메시지는 **정상 처리(ACK)** 되기 전까지 삭제되지 않음
- 이로 인해 **중복 전송**이 발생할 수 있지만
    
    → 멱등성(idempotency)으로 해결 가능
    
- 대규모 환경에서는 exactly-once 보장보다
    
    → at-least-once + 멱등성 조합이 훨씬 **단순하고 실용적**
    

## Dead Letter Queue(DLQ) 기반 장애 처리

---

> **실패를 관리하는 것도 안정성의 핵심**
> 

**Dead Letter Queue란?**

- **여러 번 재시도해도 실패한 메시지를 저장**하는 특별 큐
- 메시지를 유실시키지 않고 **사후 분석 및 수동 처리** 가능

**DLQ 동작 방식 요약**

| 기능 | 설명 |
| --- | --- |
| 재시도 | 임시 장애 시 **지수 백오프** 기반 재시도 |
| 격리 | 계속 실패하면 DLQ로 이동 |
| 모니터링 | 재발 문제 파악 → 품질 개선 |
| 장애 확산 방지 | 실패 작업이 시스템 전체를 막지 않음 |

# 결론: 견고하고 확장 가능한 알림 시스템 구축

---

→ 수백만 사용자에게 알림을 안정적으로 제공하는 시스템을 만들기 위해

- **메시지 큐 기반 비동기 처리(Decoupling)**
- **채널별 수평 확장(Horizontal Scaling)**
- **재시도 + DLQ + 멱등성(Idempotency)** 을 통한 내결함성 확보
- **SOLID 원칙 + Factory/Strategy 패턴**
    
    → 유지보수성 및 확장성 강화
    
- **모니터링 & 일관성(Observability / Eventual Consistency)** 확보