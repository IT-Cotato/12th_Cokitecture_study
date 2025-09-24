# 1주차 발표 자료 1

[https://blog.teamtreehouse.com/should-you-go-beyond-relational-databases](https://blog.teamtreehouse.com/should-you-go-beyond-relational-databases)

2번

데이터 유형에 따라 RDBMS가 아닌, 다른 DBMS가 필요

## 1. RDBMS의 한계

### 구조적

- **Sparse Tables**: 행 당 열 수가 많은데, 행과 관계없는 열이 존재하는가?
- **Attribute Tables**: 복잡한 조인 테이블 유발
    
    Ex) FK + name + value
    
- **Serialized Data**: 스키마에 구조화된 데이터(자주 변하는) 직렬화하여 저장
    
    Ex) JSON, XML, YAML 등
    
- **Complex Relationships**: 많은 조인이 필요한 관계
    
    Ex) many-to-many, 계층구조(recursive FK) 등
    
- **Frequent Schema Changes**: 스키마 변화가 빈번함

### 확장적

- **Write Bottlenecks:** 단일 DB에서 write 용량 한계에 도달 (read 한계는 복제/캐싱 사용)
- **Data Volume:** 데이터 양이 많음 → 단일 DB 서버에 보관하기 어려움
- **Performance Issues:** batch/analytical process가 트랜잭션을 방해 (성능 저하 유발)

---

## 2. Non-Relational DB

: 기존 관게형 DB의 한계를 해결하기 위해 설계된 비관계형 DB

- 사용 사례

### **Key-Value Stores**

- 특징
    - 키 기반 조회 수행
    - 해시 테이블과 유사한 동작
    - 지연 시간↓ (실시간 처리 가능)
    - write/read 처리량↑
    - 수평적 확장 가능
- 최적 용도: 캐싱. 세션 저장. 사용자 환경 등 간단한 조회 작업
- 예시: Redis, DynamoDB, Aerospike

### **Document Databases**

- 특징
    - 반구조화된 데이터 처리하도록 설계
        
        Ex) JSON, BSON 등
        
    - 유연한 스키마 허용 → 반정형 데이터 모델에 적합
    - 문서 내 필드에 대한 쿼리&인덱싱 지원
        
        → 별도 로직 없이 필터링, 집계 작업 가능
        
- 최적 용도: 콘텐츠 관리 시스템, 제품 카탈로그, 사용자 프로필 등
- 예시: MongoDB, Couchbase

### **Column-Family Stores**

- 대규모 분산, 빅데이터 분석용으로 사용
- 최적 용도: write 처리량이 높은 작업
    
    Ex) 대규모 분석 워크로드, 로그, time-series 데이터 처리 등
    
- 예시: Apache Cassandra, HBase

### Graph Databases

- 특징:
    - DB간 강한 연결을 가진 애플리케이션에 사용
    - 가변 길이 관계 체인을 효율적으로 탐색 처리
    - **전이적 관계** 등 **복잡한 쿼리** 처리 가능
        
        (ex. friend-of-a-friend 쿼리)
        
    - **다대다 관계, 계층적 데이터** 효율적 관리
    - **그래프 탐색**을 위한 쿼리 언어 사용
        
        Ex) Cypher, Gremlin 등
        
- 최적 용도: 소셜 네트워크, 추천 엔진, 사기 탐지, 지식 그래프
- 예시: Neo4j, Amazon Neptune, ArangoDB

### **Bigtable-Inspired Databases**

- Google Bigtable이 희소 테이블 및 와이드 칼럼 데이터 저장을 위해 도입한 데이터 모델
- 특징
    - 각 행은 임의의 수의 컬럼 가짐
    - 비어있지 않은 값만 저장함 (오버헤드↓)
    - 확장 가능
    - 유연한 컬럼 구조 (스키마X)
        - 동적 데이터 모델 가능
    - 대규모 데이터 관리
- 최적 용도: IoT 데이터, 시계열 데이터, 분석 처리
- 예시: Google Bigtable, Apache Cassandra, HBase

### **Distributed Key-Value Stores**

- 특징:
    - 키-값 저장소의 단순성을 확장하여 방대한 양의 데이터 처리
    - 여러 대의 머신 클러스터 사용
    - 수평적 확장성&내결함성 제공
        - 수요가 높은 애플리케이션에 이상적
    - 자동 데이터 분할 및 복제
    - 데이터 - 작업 부하 분산 샤딩
    - 특정 작업에 대한 선택적 엄격 일관성 지원
    - 지연 시간(요청-응답 주기)과 높은 처리량(배치 처리) 사이 균형 맞추기
    - 
- 최적 용도: 전자상거래, 게임, IoT 등 대규모 데이터셋에 대한 저지연 접근이 필요한 대규모 애플리케이션
- 예시: Amazon DynamoDB, Azure Cosmos DB, ScyllaDB
- Brewer’s CAP Theorem: 일관성, 가용성, 분할 내구성 중 두 가지만 우선순위 둘 수 있음

---

## 3. MapReduce and Batch Processing

- 대규모 데이터 처리 시 인프라 걱정 없이 병렬 처리 가능
- 특징
    - 데이터 처리에 대해 Divide-and-conquer 접근
    - 분산 저장 시스템과 원활하게 통합
        
        Ex) HDFS, Amazon S3
        
    - document DB들이 내부적으로 유사한 기능(local aggregation, filtering) 제공하는 경우도 있음
- 최적 용도: Apache Hadoop, Apache Spark
- 예시: 로그 분석, ETL 파이프라인, 머신러닝 워크플로우와 같은 백그라운드 데이터 처리

---

## 4. 올바른 DB를 선택하는 방법

DB 선택시 고려사항

1. **데이터 구조 (Data Structure)**
    
    : 데이터가 관계형으로 딱 맞는지, 계층적/반정형/연결성이 많은지 여부
    
2. **확장성 요구 (Scalability Requirements)**
    
    : 수직 확장(vertical), 수평 확장(horizontal)의 필요성, 대량의 데이터나 높은 처리량 여부
    
3. **쿼리 패턴 (Query Patterns)**
    
    : 단순 조회인지, 복잡한 조인/관계 탐색인지, 필터링/집계/변환이 많이 필요한지
    
4. **운영 복잡성 (Operational Complexity)**
    
    : 유지보수, 장애 처리, 복제/샤딩 등 관리 어려움 여부
    
5. **개발자 경험 (Developer Experience)**
    
    : 팀이 익숙한 기술인지, 학습 곡선, 생산성 고려 등
    

---

## 5. 데이터베이스 기술 트렌드

- **멀티모델(Multi-Model Databases)**:
    - 한 DB 시스템에서 여러 모델(관계, 문서, 그래프 등)을 지원하는 추세
        
        → 유연성 증가
        
    
    Ex) PostgreSQL, ArangoDB
    
- **Serverless DB**:
    - 인프라 자동 확장 + 관리 부담↓
    
    Ex) AWS, Azure
    
- **AI 기반 쿼리 최적화**
    - 쿼리 실행 계획, 인덱스 최적화, 리소스 할당 등에 AI 활용 증가

---

## 6. 결론

- 관계형 DB는 여전히 유용하고 견고하며 많은 경우에서 적합함
- 하지만 모든 경우에 관계형이 최선은 아님
- 애플리케이션 요구가 변화하면 데이터 모델/DB 기술도 고려
    1. 비관계형 데이터베이스 활용
    2. 분산처리 프레임워크 등 데이터 처리 기법 활용
- 실제 요구사항 기반으로 판단해야 함