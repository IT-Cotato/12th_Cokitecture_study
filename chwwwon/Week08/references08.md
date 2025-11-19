# 8주차 발표자료

## 1.  Dynamic Rendering이란?

- **정의**: 검색엔진 크롤러(예: Googlebot)에게는 **미리 렌더링된 정적 HTML**을 제공하고, 일반 사용자에게는 클라이언트 렌더링 SPA(Single Page Application)를 그대로 제공하는 방식.
- **목적**
    1. SPA 특유의 빠르고 부드러운 UX 유지
    2. 검색 엔진에는 완성된 HTML 제공 → **JavaScript 인덱싱 문제 해결**
- **성격**: JS 렌더링 문제를 우회하기 위한 **임시 해결책(Workaround)**

---

## 2. 현재 권고 사항

- Google은 **Dynamic Rendering을 더 이상 권장하지 않음**.
    - 유지보수 비용 높고, 사용자/봇 사이의 콘텐츠 불일치 위험이 있기 때문.
- **권장 대안**
    - **SSR (Server-Side Rendering)** — Next.js, Nuxt
    - **SSG (Static Site Generation)**
    - **Hydration / Islands Architecture**
- **가능하면 SSR/SSG로 전환**
- 단, 기존 SPA/PWA 구조에서 전환이 어렵다면 **한시적으로 Dynamic Rendering 적용 가능**

---

## 3. 아키텍처 개념도 & 동작 흐름

![image.png](8%EC%A3%BC%EC%B0%A8%20%EB%B0%9C%ED%91%9C%EC%9E%90%EB%A3%8C/image.png)

1. **요청 식별**: 미들웨어가 **User-Agent**를 확인해 Bot인지 사용자 인지 판단
2. **스위칭**:
    - **사용자** → 원래 SPA(클라이언트 렌더링) 응답
    - **봇**(JS 취약) → **프리렌더러(Rendertron/Prerender 등)** 로 페이지 렌더링 요청
3. **캐싱**: 프리렌더 결과를 **캐시**하여 반복 크롤링 시 속도/리소스 절감
4. **전달**: 봇에게는 **JS 실행 완료된 HTML**을 반환 → 인덱싱 안정화

---

## 4. 다이나믹 렌더링을 구현하는 경우

- SPA/PWA 이며 **콘텐츠가 자주 변경되는 사이트**
- 크롤러가 JS 실행 전에 **로딩 화면만 가져가는 경우**
- SSR/SSG 도입까지 **임시 단계(브리지 전략)** 가 필요할 때
- 실무 사례 (앱 AffiliCats)
    - SPA가 로딩 화면만 노출되어 모바일 친화성 검사/크롤링 실패
    - Rendertron + Express 미들웨어로 Googlebot 등 특정 UA에 프리렌더 HTML을 제공
    - 캐시 선웜업(서버 시작 시 SPA 선호출)과 주기 갱신 구현

---

## 5. 구현 체크리스트

> 과거 다이나믹 렌더링 구현시 권장되던 실무 흐름
> 

### A. 프리렌더러 준비

- 예: **Rendertron (Deprecated)**, Prerender.io 또는 Puppeteer 기반 자체 렌더러

### B. 서버 미들웨어 구성 (Express 예시)

- User-Agent 리스트에 봇 추가
- 봇 요청 시 프리렌더된 HTML 반환
- 사용자 요청은 SPA 리소스 (index.html, JS bundle) 그대로 제공

### C. 캐시 전략

- 서버 기동 시 주요 URL **사전 렌더링 (pre-warm)**
- 주기적으로 새로 렌더링하여 **캐시 갱신**

### D. 검증 & 모니터링

- Google Search Console의 URL 검사 도구 테스트
- 크롤링/커버리지 상태 모니터링
- 로깅: 봇/사용자 요청 비율, 프리렌더 렌더링 시간, 실패율

---

## 6. 주의 사항 & 리스크

| 항목 | 설명 |
| --- | --- |
| 콘텐츠 일관성 | 사용자와 봇이 **동일한 정보**를 보도록 유지해야 함 (차이가 크면 클로킹 위험) |
| 운영 비용 증가 | 프리렌더 서버 운영, 캐시 동기화, UA 탐지 등 유지보수 부담 |
| Rendertron 종료 | Rendertron은 Deprecated 상태 → **장기적 전략으로 사용X** |
| 보안 고려 | 프록시 라우팅, headless browser 운영에 따른 공격면 증가 가능 |

---

## 7. 권장 대안 설계(실무 관점)

- **SSR** — 서버에서 HTML을 생성하여 SEO + UX 모두 확보
- **SSG + ISR** — 트래픽 많은 페이지에 적합 (정적 + 필요 시 자동 갱신)
- **Hydration / Islands** — 정적으로 렌더 후 필요한 컴포넌트만 JS 동작

***“목표: 사용자와 봇 모두에게 “같은 HTML”을 효율적으로 제공하는 것”***