# 12기 아키텍처 스터디
---
>
>
> 
## Members
---
|                           <a href="https://github.com/gimin0226"><img src="https://github.com/gimin0226.png" width=120/></a>                           |                          <a href="https://github.com/marshmallowing"><img src="https://github.com/marshmallowing.png" width=120/></a>                           |                       <a href="https://github.com/chwwwon"><img src="https://github.com/chwwwon.png" width=120 /></a>                         |
|:-------------------------------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------------------:|
|                                     <a href="https://github.com/gimin0226">김기민</a>                                     |                                 <a href="https://github.com/marshmallowing">정유진</a>                                  |                                  <a href="https://github.com/chwwwon">최형원</a>                                  |                
## 목표
1. **대규모 트래픽을 처리하는 시스템 설계 원리**를 체계적으로 이해하게 됩니다.
2. **클라우드, 데이터베이스, 캐시, 메시지 큐** 등 주요 아키텍처 요소를 실무 관점에서 설명할 수 있게 됩니다.
3. 실제 면접에서 자주 다뤄지는 **시스템 설계 문제 해결 패턴**을 익히고 응용할 수 있게됩니다.
---
## 스터디 일정
- 정기 모임: 매주 수요일 오후 7시
- 진행 방식: 강남역 대면
- 과제 제출: 매주 화요일 23:59까지
---
## 스터디 커리큘럼

| **주차** | **챕터** | **챕터 내용** |
|---------|--------|----------------|
| 1주차 | 1장, 2장 | 사용자 수에 따른 규모 확장성, 개략적인 규모 추정|
| 2주차 | 3장, 4장 | 시스템 설계 면접 공략법, 처리율 제한 장치의 설계 | 
| 3주차 | 5장      | 안정 해시 설계 | 
| 4주차 | 6장     | 키-값 저장소 설계 |
| 5주차 | 7장     | 분산 시스템을 위한 유일 ID 생성기 설계 | 
| 6주차 | 8장 | URL 단축기 설계 |  
| 7주차 | 9장 | 웹 크롤러 설계 | 
| 8주차 | 10장 | 알림 시스템 설계 | 
| 9주차 | 11장 | 뉴스 피드 시스템 설계 |  
| 10주차 | 12장 | 채팅 시스템 설계 | 
| 11주차 | 13장 | 검색어 자동완성 시스템 |  
| 12주차 | 14장 | 유튜브 설계 |  
| 13주차 | 15장 | 구글 드라이브 설계 | 
## 스터디 진행 방식
- 매주 정해진 파트를 읽고 정리한다.
- 각 파트 끝에 있는 참고문헌을 2개씩 공부하고 정리하여 발표한다.
- 파트의 주제와 연관된 기술 블로그를 찾아 발표한다. (선택)
  
## Directory Structure
---
```
│
├─ multithread-java-study
│     │
│     │  
│     │
│     │
│     ├─ gimin0226/ // 본인의 핸들명(Github ID)
│     │     ├─  Week01/
│     │     │    ├─ chapter01.md
│     │     │    ├─ references01.md # 발표 자료 ( 참고문헌 정리한 내용 )
│     │     │    ├─ references02.md 
│     │     │    └─ tech-blog.md
│     │     │
│     │     ├─  Week02/
│     │     │    ├─ chapter01.md # 책 내용 정리
│     │     │    ├─ references01.md # 발표 자료 ( 참고문헌 정리한 내용 )
│     │     │    ├─ references02.md 
│     │     │    └─ tech-blog.md
│     │     │
│     │     │
│     │     └─ ... 이하 동일
│     │   
│     │   
│     ├─ marshmallowing/                 // 다른 구성원도 동일 구조
│
│
```

## 브랜치 전략 가이드

### 원칙
- 참가자는 해당 주차 문서를 **개인 주차 브랜치**에서 작업한다.  
  예: `gimin0226-week-01`
- 모든 PR의 대상(base)은 **develop** 이다. `main`에는 직접 올리지 않는다.
- 주차 마감 시 스터디장(김기민)이 `develop`의 누적 변경을 `main`에 한 번에 반영한다.
  - main: 마감본 확정본, 주차가 끝나야만 갱신됨
  - develop: 이번 주 모두가 함께 쌓아가는 통합본, 개인 PR들이 계속 머지되는 곳
---

### 브랜치 네이밍 규칙
- `<github핸들명>-week-<NN>`
- 예: `gimin0226-week-01`, `marshmallowing-week-03`

---

### 참가자 작업 흐름 (로컬 → PR)

### 1. 기준 브랜치 최신화
```bash
git switch develop
git pull origin develop --ff-only
```

- 원격 저장소(origin)의 `develop` 브랜치를 가져와 fast-forward가 가능할 때만 반영한다.
- fast-forward merge: 로컬 브랜치에 새 커밋이 없고, 원격이 더 앞서 있을 때 단순히 **포인터만 앞으로 이동**하는 병합 방식
- 즉, 내 로컬 브랜치가 원격보다 뒤처졌을 때만, 안전하게 최신화하는 것

### 2. 주차 브랜치 생성
```bash
git switch -c gimin0226-week-01
```

### 3. 산출물 작성 및 커밋 후 원격 푸시
```bash
git add .
git commit -m "docs: Week01 chapter01 수정"
git push -u origin gimin-week-01
```
- 커밋 메시지 prefix 가이드
  - `docs`: → 개인 문서/섹션
- 커밋 메시지 예시
  - `docs: Week01 chapter01 수정`
  - `docs: Week01 chapter01 생성`

### 4. Github에서 PR 생성
- base: `develop`
- compare: `gimin-week-01`
- PR 제목 예시
  - `[김기민] Week01/chapter01.md 제출`

### 5. 머지 후 로컬 정리
- 스터디장이 PR을 승인하고 Merge하면, Github가 자동으로 해당 커밋을 `develop`에 합친다.
```bash
git switch develop
git pull origin develop --ff-only
git branch -d gimin-week-01
```
- PR이 병합됐으니, 로컬도 상태를 최신화하고, 필요 없어진 작업 브랜치 정리
- branch 삭제 작업은 모든 파일을 develop 브랜치에 올린 후 수행할 것
 
---
## 스터디장 할 것 (다른 스터디원들은 x)

### PR 리뷰·머지 기준
- base가 `develop`인지 확인
- 제목,본문 컨벤션 준수 여부 확인

### 주차 마감 직전 점검
- `develop`에 해당 주차 PR들이 모두 머지됐는지 확인
- 필요시 누락자 멘션

### 주차 마감 반영( develop -> main )
- Github에서 Create pull request 선택
- base: `main`, compare: `develop`
- PR 제목: `1주차 마감 PR`
- 본문: 주요 변경 요약 + 참여자 목록 정리
