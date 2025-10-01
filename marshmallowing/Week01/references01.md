## Replication

> **Replication**은 동일한 데이터를 여러 서버에 동기화해 두어 DB의 부하를 분산 시키는 기술
>
- **읽기 성능 확장**
    - 읽기 작업과 쓰기 작업의 부하를 분산시켜 성능 향상
- **장애 복구 지원**
    - **Failover -** 장애가 발생해도 서비스 지속 가능한 기능
    - Master의 내용을 복제하기 때문에 데이터베이스가 중지된다해도 Slave 중 하나를 Master로 승격시켜 **SPOF**를 피할 수 있다.
- 백업 및 데이터 보안 강화, 지역 분산
- 핵심 이슈: **Consistency(일관성) ↔ Availability(가용성) ↔ Partition tolerance(네트워크 분리 허용)** 간의 균형 (CAP 정리)

### Replication 방식

애플리케이션 서버는 **Master 서버에 Insert, Update, Delete 작업을** 요청하고 **Master 서버는 Slave(Replica) 서버에 변경된 이벤트를 전송하여 데이터를 동기화한다.** **Select 작업은 Slave 서버로 요청하여** 읽기 작업과 쓰기 작업의 부하를 분산시킨다. 예시 자료에서는 한 대의 Slave 서버를 두었지만, 1대의 Master 서버에 여러 Slave 서버를 두어서 대용량 트래픽에 적합한 Multi Slave 구성도 가능하다.

### MySQL에서의 Replication 동작 방식

<aside>
💡

MySQL 복제는 **Binary Log(바이너리 로그)** 기반으로 동작한다

</aside>

> MySQL은 Replication 환경에서 3개의 쓰레드를 생성한다
>
- **Binary Log Dump Thread**
    - Replica 서버로부터 Replication 요청이 발생했을때 **Master 서버에 생성되는 쓰레드**
    - 모든 변경(INSERT, UPDATE, DELETE 등)을 **Binary Log(binlog)**에 기록
    - Binary Log 를 읽어서 Slave 쪽으로 데이터를 전송
    - 바이너리 로그(Binary Log)

      MySQL 서버에서 Create, Alter, Drop과 같은 작업을 수행하면 MySQL은 그 변화된 이벤트를 기록하는데 이러한 **변경사항들에 대한 정보를 담고 있는 이진 파일을 바이너리 로그 파일**

- **I/O receiver Thread**
    - Replication 시작 명령어인 **START REPLICA** 명령이 실행되면 **Replica 서버에 생성되는 쓰레드**
    - Master 서버의 Binary Log Dump Thread 와 연결하여 Binary Log를 읽고 읽어온 데이터를 **Relay Log**에 저장
- **SQL applier Thread**
    - **Replica 서버에 생성되는 쓰레드**
    - Relay Log 내용을 읽어서 실제 쿼리를 실행 → 데이터 반영

<aside>
💡

**Primary(binlog 작성) → Replica(IO Thread → Relay Log → SQL Thread)** 흐름

</aside>


1. 클라이언트(Application)에서 Commit을 수행한다.
2. Connection Thead는 스토리지 엔진에게 해당 트랜잭션에 대한 Prepare(Commit 준비)를 수행한다.
3. Commit을 수행하기 전에 먼저 **Binary Log**에 변경사항을 기록한다.
4. 스토리지 엔진에게 트랜잭션 Commit을 수행한다.
5. Master Thread는 시간에 구애받지 않고(비동기적으로) Binary Log를 읽어서 Slave로 전송한다.
6. Slave의 I/O Thread는 Master로부터 수신한 변경 데이터를 **Relay Log**에 기록한다. (기록하는 방식은 Master의 Binary Log와 동일하다)
7. Slave의 SQL Thread는 Relay Log에 기록된 변경 데이터를 읽어서 스토리지 엔진에 적용한다

### 복제 방식

- **Statement-based Replication (SBR)**
    - Master에서 **실행한** **SQL 문장 자체**를 Replica로 전달
    - 문제: `NOW()` 같은 비결정적 함수 사용 시 데이터 불일치 가능
- **Row-based Replication (RBR)**
    - **변경된 행(row) 단위**로 데이터를 복제
    - 안정적이고 일관성 ↑, 하지만 binlog 크기가 커질 수 있음
- **Mixed Replication**
    - 기본적으로 Statement 사용, 필요 시 Row로 전환
    - 두 방식의 장점 혼합

### 복제 동기화 방식

- **Asynchronous Replication** **비동기 복제** (기본 동작)
    - Master가 binlog만 기록하면 성공으로 처리 Replica 서버에 변경 이벤트가 정상적으로 전달되었는지 여부와 상관없이 COMMIT
    - 성능은 높지만, 장애 시 **데이터 유실 위험, 데이터 정합성 측면에서 불리**
- **Semi-synchronous Replication 반동기 복제**
    - 최소 1개의 Replica가 binlog를 수신해 Relay Log에 기록 후 Ack 응답을 보내면 COMMIT 완료
    - 데이터 유실 가능성 ↓, 성능은 Asynchronous보다는 낮음

### 고려해야 할 이슈 & 트레이드오프

- **복제 지연(latency)**
    - 슬레이브가 마스터보다 늦게 업데이트되는 경우, 읽기 요청 시 오래된 데이터를 줄 가능성
- **일관성 vs 가용성**
    - CAP 정리처럼, 복제 시스템은 모든 조건을 만족할 수 없기 때문에 일관성, 가용성, 파티션 내성 간에 균형을 잡아야 함
- **충돌 해결(Conflict Resolution)**
    - 멀티 마스터 복제에서는 동일한 레코드를 여러 마스터가 동시에 수정할 수 있으므로, 충돌을 어떻게 해결할지를 정의해야 함 (타임스탬프 기반, 버전 기반 등)
- **장애 복구 (Failover / Failback)**
    - 마스터 장애 시 슬레이브를 승격해서 마스터로 만드는 구조 필요
    - 승격된 마스터와 나머지 슬레이브 간 동기화 및 재조정 필요

<aside>
💡

결과적으로 Replication 을 통해 데이터베이스 리소스를 이중화 할 수 있고 이는 백업, 확장, 분석, 분산 등 여러 장점을 가질 수 있다. 하지만 기본적으로 Replication은 비동기 통신을 통해 동기화되므로 **데이터 정합성 문제가 발생할 수 있다.** 

</aside>

참고자료

[https://en.wikipedia.org/wiki/Replication_(computing)](https://en.wikipedia.org/wiki/Replication_(computing))

[https://rachel0115.tistory.com/entry/MySQL-가용성과-확장성을-위한-Database-Replication](https://rachel0115.tistory.com/entry/%08MySQL-%EA%B0%80%EC%9A%A9%EC%84%B1%EA%B3%BC-%ED%99%95%EC%9E%A5%EC%84%B1%EC%9D%84-%EC%9C%84%ED%95%9C-Database-Replication)

[https://velog.io/@zpswl45/DB-Replication-개념-정리](https://velog.io/@zpswl45/DB-Replication-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC)