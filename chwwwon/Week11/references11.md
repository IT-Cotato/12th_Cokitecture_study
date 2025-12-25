# 11주차 발표자료

[How to Design an Autocomplete System](https://dzone.com/articles/how-to-design-a-autocomplete-system)

# How to Design an Autocomplete System

# Introduction

### 자동완성이란?

> 사용자가 검색할 때 입력창에 표시되는 검색어 제안 기능
> 
- 텍스트 입력속도 향상
- 타이핑 중 검색어 예측으로 빠른 검색 경험 제공

### 좋은 자동완성 시스템의 요구사항

- **빠른 응답 속도**
    - 사용자가 **한 글자를 입력할 때마다 즉시** 추천 결과가 갱신되어야 함
- **실시간 업데이트**
    - 지연이 발생하면 UX가 급격히 나빠짐
- **정확한 예측**
    - 사용자가 입력 중인 검색 의도를 최대한 정확히 추론

### Vocabulary(어휘 집합)의 개념

- Autocomplete는 **시스템이 알고 있는 단어만 추천 가능**
- 추천 후보는 미리 정의된 **Vocabulary**에서 나옴

**예시 Vocabulary**

- Wikipedia에 존재하는 모든 고유 단어
- 검색 로그에서 수집된 인기 검색어
- 다중 단어 구문(multi-word phrases), 엔티티 이름(사람/장소/브랜드 등)

📌 **문제점**

- Vocabulary의 크기가 매우 큼
    - Wikipedia 기준: 수백만 개 단어
    - 구문/엔티티까지 포함하면 규모는 더 커짐

### 시스템의 핵심: Autocomplete API

- 입력값: **prefix(접두사)**
    
    예: `aut`, `sys`, `des`
    
- 처리:
    - prefix로 시작하는 vocabulary 단어 검색
- 출력:
    - 가능한 자동완성 후보 중 **상위 k개만 반환**

📌 즉, 시스템의 본질은

> “주어진 prefix로 시작하는 단어들을 빠르게 찾아서 상위 k개를 반환하는 문제”
> 

# Suffix Tree Algorithm

### Suffix Tree란?

- 하나의 문자열에 대해 **모든 접미사(suffix)** 를 저장한 **압축된 Trie(Compressed Trie)** 구조
- 즉, **Trie의 변형이자 개선된 형태**

### Suffix Tree의 활용 분야

Suffix Tree는 다양한 문자열 처리 문제에 활용됨:

- 문자열 패턴 매칭 (Pattern Matching)
- 서로 다른 부분 문자열(distinct substrings) 찾기
- 가장 긴 팰린드롬(Longest Palindrome) 탐색 등

### Trie 대비 Suffix Tree의 개선점

- 기존 Trie:
    - 분기(branch)가 없는 **긴 단일 경로(long path)** 가 자주 발생
- Suffix Tree는:
    - 긴 경로를 **하나의 경로로 압축**
    - 공통 부분은 공유하면서 불필요한 노드 제거

→ Trie에 비해 **트리 크기 감소, 메모리 사용이 효율적**

## Suffix Tree의 개념적 정의

- 문자열 s에 대해
    - s의 **모든 접미사 집합**을 기반으로 Trie를 구성
    - 단, **분기 없는 경로는 압축하여 표현**

→ 문자열 s의 부분 문자열 집합 위에 정의된 Trie라고 볼 수 있음

**예시**

- 문자열 s: abakan
- s가 가지는 모든 접미사 집합: { abakan, bakan, akan, kan, an, n }

![image.png](11%EC%A3%BC%EC%B0%A8%20%EB%B0%9C%ED%91%9C%EC%9E%90%EB%A3%8C/image.png)

## Suffix Tree의 성능

### Suffix Tree 생성 알고리즘

- 문자열 **s의 길이에 대해 선형 시간(O(|s|))** 으로 접미사 트리(Suffix Tree)를 생성하는 알고리즘이 존재
- 대표적인 알고리즘: **Ukkonen 알고리즘**

📌 즉, Suffix Tree는

> 이론적으로도 매우 효율적으로 구축 가능한 자료구조
> 

### Suffix Tree의 문제 해결 능력

문자열에 대한 많은 정보를 포함하여 다양한 **복잡한 문자열 문제**를 해결 가능

- **패턴 등장 횟수 계산**
    - 패턴 P를 트리에서 찾은 뒤
    - 해당 노드의 **서브트리 크기**를 계산
- **서로 다른 부분 문자열 개수 계산**
    - Suffix Tree를 사용하면 쉽게 해결 가능

### Prefix Completion (자동완성 원리)

접두사(prefix)에 대한 자동완성은 다음 과정으로 수행됨.

1. 접두사의 문자들을 따라
    
    **Trie에서 경로를 순차적으로 탐색**
    
2. 탐색은 **어떤 내부 노드(inner node)** 에서 종료됨
3. 해당 내부 노드에서 시작하여
    
    **도달 가능한 모든 리프 노드까지 탐색**
    
4. 이 리프 노드들이
    
    → 접두사에 대한 **완성 후보(completions)** 가 됨
    

📌 즉,

> 자동완성은 “접두사까지 내려간 뒤, 그 아래를 전부 훑는 과정”
> 

### Prefix Tree(Trie)의 검색 성능

- 접두사 검색에 필요한 비교 횟수
    - **접두사의 문자 수와 동일**
- 검색 시간
    - **어휘(vocabulary) 크기와 무관**

→ 대규모 어휘 집합에서도 성능 유지 가능

### Trie vs 정렬 리스트

- Trie는 **정렬된 리스트(over-ordered lists)** 대비 **훨씬 큰 속도 개선**을 제공
- 이유:
    - 각 문자 비교 단계에서 **검색 공간의 큰 부분을 한 번에 제거** 가능

# **Minimal DFA**

## Prefix Tree의 한계

- Prefix Tree(Trie)는 **공통 접두사(prefix)** 는 효율적으로 저장
- 하지만:
    - `ing`, `ion` 같은 **공통 접미사(suffix)** 는 각 분기(branch)에 **중복 저장됨**

→ 어휘(vocabulary)가 커질수록 메모리 사용량이 빠르게 증가

### Prefix Tree와 DFA의 관계

- Prefix Tree는 **비순환 결정적 유한 오토마타(Acyclic DFA)** 의 한 형태
    
    **Prefix Tree ⊂ DFA**
    
- DFA 관점에서 보면:
    - **동일한 동작을 하는 상태(state)를** 하나로 합칠 수 있음

## DFA 최소화 (Minimization)

### **DFA 최소화란?**

- DFA를 입력으로 받아 **동일한 언어를 인식하며, 더 적은 노드를 가지는 DFA**로 변환
- Prefix Tree를 DFA로 보고 최소화하면:
    - 데이터 구조 크기 감소
    - 중복된 단어 부분(특히 suffix) 공유 가능

📌 결과:

- **Minimal DFA**는 vocabulary가 매우 커도 메모리에 적재 가능
- 디스크 접근을 피할 수 있어 **초고속 Autocomplete 구현 가능**

## Myhill–Nerode 정리

> **최소 DFA를 이론적으로 정의하는 기반**
> 
- 핵심 개념:
    - **문자열 동치 클래스(string equivalence class)**

### 상태의 구분 가능성 (Distinguishability)

- 두 상태가 **구분 불가능(indistinguishable)** 하다는 의미:
    - 모든 문자열에 대해
        - 둘 다 **최종 상태(final state)** 로 가거나
        - 둘 다 **비최종 상태(non-final state)** 로 감

→ 실제로는 모든 문자열을 테스트할 수 없음

### 점진적 동치 판별 아이디어

- **구분 불가능성(indistinguishability)** 을 점진적으로 계산
- 정의
    - 상태 p와 q가 길이 ≤ k 인 문자열로 구분 가능하면 **k-distinguishable** 하다고 한다.
- 짧은 문자열부터 시작해 점점 긴 문자열로 상태를 구분
- 결국 **동치 상태들을 하나로 병합**

## Partition Refinement

> Myhill–Nerode 이론을 실제로 계산 가능하게 만든 절차
> 

### 0단계 (0-indistinguishable)

- 상태를 먼저 두 그룹으로 나눈다:
    - **final 상태**
    - **non-final 상태**

**→ “최종적으로 가는지 아닌지”의 가장 기본적인 구분**

### 1단계 이상 (k-distinguishable)

같은 그룹 안의 상태 p, q에 대해:

- 어떤 입력 기호 σ가 존재하여
- δ(p, σ) 와 δ(q, σ)가 이전 단계에서 **다른 그룹**으로 간다면
    - p와 q는 **구분 가능**
    - 같은 그룹에 둘 수 없음

→ 이렇게 **partition을 계속 쪼갠다**

### 종료 조건

- 더 이상 어떤 입력으로도
    
    그룹을 나눌 수 없을 때 종료
    
- 이때의 partition들이 바로:

> ✅ 최소 DFA의 상태 (indistinguishability equivalence classes)
> 

### Trie / Autocomplete와의 연결

- Trie는 DFA의 한 형태
- DFA를 최소화하면:
    - 공통 suffix까지 공유
    - 메모리 사용량 감소
- Minimal DFA는:
    - 큰 vocabulary도 메모리에 적재 가능
    - 디스크 접근 없이 **초고속 autocomplete 가능**

# Autocomplete System Design

- Autocomplete는 단순 Trie 문제가 아님
- **속도 + 확장성**을 모두 만족해야 하는 시스템 설계 문제
- 구현 방식은 하나로 정해져 있지 않음

## 요청 처리 흐름

User

→ Load Balancer

→ Autocomplete API Node

- 사용자는 prefix 입력
- Load Balancer가 요청을 여러 노드로 분산

![image.png](11%EC%A3%BC%EC%B0%A8%20%EB%B0%9C%ED%91%9C%EC%9E%90%EB%A3%8C/image%201.png)

### Autocomplete 노드 역할

- 각 노드는 **마이크로서비스**
- 처리 순서:
    1. Cache 확인
    2. Cache hit → 즉시 응답
    3. Cache miss → Zookeeper 조회

### Zookeeper의 역할

> **어떤 suffix-tree 서버가 어떤 범위를 담당하는지**를 관리
> 
- 예:
    - `a$ → s1`
    - a로 시작하는 단어는 s1 서버가 처리

### Background Processor의 역할

- 실시간 처리와 분리된 **비동기 처리**
- 대량의 문자열 데이터를 수집 및 집계
- Suffix Tree 서버에 반영

### 데이터 & 가중치 처리

- **입력 데이터**: phrases, weights
- **출처**: 사전, 용어집, 데이터 마이닝 결과
- **weights는 추천 중요도를 나타냄**

### Aggregation 전략

- phrase + weight를 해시
- Aggregator에서 집계:
    - 유사 term
    - 생성 시간
    - 가중치 합
- DB에 저장 후 추천에 활용

## Autocomplete 시스템 확장 개선

### 기존 시스템의 한계

- 수천 개 이상의 동시 요청 처리에 한계
- 단일 서버 구조 → 장애에 취약

![image.png](11%EC%A3%BC%EC%B0%A8%20%EB%B0%9C%ED%91%9C%EC%9E%90%EB%A3%8C/image%202.png)

### 수평 확장 (Horizontal Scaling)

- 여러 서버를 추가하여 트래픽 분산
- **Round-robin** 방식으로 요청 균등 분배
- 단일 장애 지점 제거

### 캐시 서버 확장 문제

- 캐시 서버를 여러 대로 확장 가능
- 핵심 문제:
    - 데이터를 **어떻게 균등하게 분산할 것인가**
    - 서버 장애 시 데이터 재분배 문제

### Consistent Hashing

- 서버 수 변화와 무관하게 동작하는 해싱 기법
- 데이터와 서버를 **해시 링(hash ring)** 에 배치
- 장점:
    - 서버 추가/제거 시 영향 최소화
    - 안정적인 데이터 분산
    - 확장성과 가용성 향상

### Zookeeper 설정 개선

- Suffix-tree 서버 증가에 따라 범위 재정의 필요
- 노드 서버가 올바른 suffix-tree 서버를 찾을 수 있도록 지원

# Conclusion

### 핵심 요약

- Autocomplete 시스템은 다양한 자료구조(Trie, Suffix Tree, DFA 등)를 활용해 설계 가능
- 성능 향상을 위해 **여러 확장 방식**을 적용할 수 있음
- 실제로는 한 prefix에 대해 **표시 가능한 개수보다 훨씬 많은 후보**가 존재

### 구현에 대한 현실적인 선택

- 자동완성 자료구조를 **직접 구현할 필요는 없음**
- 이미 검증된 **오픈소스 솔루션** 활용 가능

### 대표적인 도구

- **Elasticsearch**
- **Solr**
- **Lucene 기반 검색 엔진**

**→ 효율적이고 안정적인 자동완성 기능을 기본 제공**