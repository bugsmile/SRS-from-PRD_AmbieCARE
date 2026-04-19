# SRS v02 → v03 수정 계획서
## — 바이브코딩 MVP 최적화: 난이도 조정 + 기술적 모순 해소 —

**문서 ID:** PLAN-SRS-002-VIBE  
**Revision:** 1.0  
**Date:** 2026-04-20  
**Base Document:** `SRS_v01(ENG_OPUS_MVP).md` (Revision 2.0)  
**Input Reviews:**
- `SRS_V02_check_difficulty.md` — 개발 난이도 및 구현 가능성 검토
- `MVP-개발목표-적절성-종합-검토(난이도-가능성-효율성)-보고서.md` — 시스템/비용 효율성 종합 검토  

**Deliverable:** `SRS_v03(ENG_OPUS_MVP_VIBE).md`  
**Standard:** ISO/IEC/IEEE 29148:2018

---

## 0. 계획 목적 및 배경

### 0.1 목적

본 계획서는 SRS v02(ENG_OPUS_MVP)를 아래 두 가지 핵심 문제를 해결하여 **바이브코딩 MVP에 실제로 실행 가능한 SRS v03**으로 수정하기 위한 작업 계획입니다.

| # | 핵심 문제 | 출처 | 해결 방향 |
| :--- | :--- | :--- | :--- |
| **P-01** | **개발 난이도 과잉** — Must 등급 FR-01~03(10개 REQ)이 모두 바이브코딩 불가. 11개 Route Handler, 4종 인증 혼재, 7개 Prisma 모델이 초급자에게 과도. | `SRS_V02_check_difficulty.md` §3~6 | Edge/HW와 Web-Only MVP를 명확히 분리하고, Route Handler·인증·모델을 축소. |
| **P-02** | **인프라 제약 vs. NFR 목표의 기술적 모순** — "완전 무료" 인프라 목표와 SLA 99.9%, p95 ≤ 2,000ms, PagerDuty 유료 연동, SMS 유료 발송 비용 등이 상충. | `종합-검토-보고서` Part III~V | NFR 임계치를 무료 인프라 현실에 맞게 재조정하고, 유료 서비스 의존 항목을 무료 대안으로 교체. |

### 0.2 대상 프로필 재확인

| 항목 | 프로필 |
| :--- | :--- |
| 개발 경험 | SW 개발 학습 3개월 (초급자) |
| 직무 배경 | IT 기획 3년차 |
| 개발 방식 | **완전 바이브코딩** (AI 코딩 어시스턴트 기반) |
| 인프라 예산 | **완전 무료** (Free Tier Only) |
| 도구 비용 | AI 코딩 어시스턴트 구독비만 허용 |

### 0.3 참조 문서

| 문서 | 역할 |
| :--- | :--- |
| `SRS_v01(ENG_OPUS_MVP).md` (v02) | 수정 대상 원본 SRS |
| `SRS_V02_check_difficulty.md` | 난이도/구현 가능성 검토 (Input #1) |
| `MVP-개발목표-적절성-종합-검토(난이도-가능성-효율성)-보고서.md` | 시스템/비용 효율성 검토 (Input #2) |
| `SRS_v01(Eng_Opus)_Plan_eng.md` | v01→v02 수정 시 사용된 이전 계획서 (구조 참고) |
| PRD v0.3 | 비즈니스 요구사항 원본 |

---

## 1. 핵심 모순 식별 및 해소 전략

### 1.1 기술적 모순 매트릭스

현재 SRS v02에 존재하는 **인프라 제약(완전 무료)과 NFR 목표 사이의 충돌**을 체계적으로 식별합니다.

| 모순 ID | SRS v02의 NFR 목표 | 무료 인프라 현실 | 충돌 수준 | 해소 전략 |
| :--- | :--- | :--- | :--- | :--- |
| **CTR-01** | **SLA ≥ 99.9%** (REQ-NF-005) — Vercel Pro(99.99%) + Supabase Pro(99.9%) 기반 산출 | Vercel Hobby: SLA 보장 없음. Supabase Free: SLA 보장 없음. | 🔴 심각 | NFR을 "Best Effort, 목표 99%"로 하향. MVP는 데모/Beta 목적이므로 SLA 강제 불필요. |
| **CTR-02** | **p95 ≤ 2,000ms** E2E 긴급 알림 (REQ-NF-001) — Vercel Edge Runtime으로 cold start 제거 가정 | Vercel Hobby: Edge Runtime 미지원. Serverless cold start 3~5초 발생 가능. | 🔴 심각 | 긴급 알림 자체가 Edge 센서에서 발생해야 하며, Web 파트에서는 "알림 수신/조회"만 담당. p95 목표를 "5,000ms 이내 알림 조회 가능"으로 완화. |
| **CTR-03** | **PagerDuty Sev1 자동 에스컬레이션** (REQ-NF-007) — PagerDuty Pro $21/사용자/월 | 완전 무료 목표와 직접 충돌 | 🔴 심각 | PagerDuty 제거. **Slack Incoming Webhook**(무료) 또는 **Discord Webhook**(무료)으로 대체. |
| **CTR-04** | **≤ 500 KRW/Unit/Month** 비용 목표 (REQ-NF-012) — Vercel Pro $20/월 + Supabase Pro $25/월 기반 | 완전 무료에서는 단위 비용 자체가 $0이지만, Free Tier 제약(Vercel 100GB-hr, Supabase 500MB DB)이 존재 | 🟡 중간 | 비용 목표를 "Free Tier 제약 내 운영"으로 재정의. 디바이스 <50대 기준. |
| **CTR-05** | **SMS/KakaoTalk 폴백** (FR-07, Should) — SMS 건당 ₩20~50, 카카오 알림톡 건당 ₩7~15 | 완전 무료 불가. 500대 × 30일 = 월 ₩300K~750K | 🔴 심각 | FR-07을 **Won't (MVP)**로 하향. 무료 대안: Email 알림(Resend 무료 100통/일), Discord/Slack Webhook. |
| **CTR-06** | **Amplitude/Mixpanel 이벤트 트래킹** (§3.1) — Amplitude Free 10M 이벤트/월은 충분하나 유료 전환 압박 | Amplitude Free는 무료이나, 외부 서비스 의존 | 🟢 낮음 | Amplitude Free 유지하되, 대안으로 **Vercel Analytics(Hobby 포함)** 또는 **Umami(오픈소스 셀프호스팅)** 명시. |
| **CTR-07** | **Supabase Realtime 200 동시 연결** (ASM-05) — Supabase Free 기본 제공 | Supabase Free: Realtime 동시 연결 200개 지원이지만, 프로젝트 일시정지(7일 미사용 시) 리스크 | 🟡 중간 | 동시 연결 목표를 <50으로 하향. 프로젝트 활성 유지를 위한 Cron ping 설정 (하지만 Vercel Hobby Cron은 일 1회 제한). |
| **CTR-08** | **Vercel Cron Jobs** — 매 1분 하트비트 모니터링 (CRON_HEARTBEAT), 자정/07:30 리포트 생성 | Vercel Hobby Cron: **일 1회** 실행만 가능 (Pro 이상에서만 분단위 지원) | 🔴 심각 | 1분 하트비트 Cron 삭제. 리포트 생성 Cron 1개만 유지(일 1회). 하트비트는 디바이스 → API 풀 방식으로 전환. 또는 **GitHub Actions cron**(무료, 5분 간격 가능)을 대안으로 명시. |
| **CTR-09** | **Vercel Hobby Serverless 한도** — 100 GB-hours/월, 함수 실행 10초 타임아웃 | 11개 Route Handler 동시 운영 시 한도 초과 가능 | 🟡 중간 | Route Handler를 6~7개로 축소. 함수 코드 최소화. |
| **CTR-10** | **Supabase Free DB 500MB 제한** — 7개 Prisma 모델, 이벤트 데이터 지속 축적 | 500 디바이스 × 288 events/일(5분 배치) × 365일 = 수 GB → 초과 | 🟡 중간 | 모델을 5개로 축소. 이벤트 보존 기간을 30일로 단축(90일 → 30일). 디바이스 목표를 <50대로 현실화. |

### 1.2 모순 해소 원칙

> **핵심 원칙: "무료 인프라에서 실현 가능한 NFR만 SRS에 명시한다."**

| 원칙 ID | 원칙 | 적용 방법 |
| :--- | :--- | :--- |
| **RP-01** | **Free Tier First** | 모든 인프라 서비스는 무료 티어 기준으로 설계. 유료 전환은 "Wave 2 확장 옵션"으로만 언급. |
| **RP-02** | **Best Effort SLA** | MVP 단계에서 SLA 수치 보장을 요구하지 않음. "Best Effort, 목표 ~99%"로 완화. |
| **RP-03** | **단일 인증** | JWT 단일 방식. RBAC, HMAC, API Key 분리는 Wave 2로 이관. |
| **RP-04** | **무료 대안 의무 탐색** | PagerDuty → Slack/Discord Webhook, SMS → Email(Resend Free), Amplitude → Vercel Analytics 또는 Umami. |
| **RP-05** | **디바이스 목표 현실화** | SRS v02의 "500대"는 Pro 플랜 기준. 무료 인프라에서는 **<50대(Beta/Demo)**로 현실화. |
| **RP-06** | **Phase 명시** | 모든 기능에 구현 Phase를 명시. 바이브코딩 실행자가 "지금 뭘 만들어야 하는지" 즉시 판단 가능하도록. |

---

## 2. 변경 범위 요약 (Impact Matrix)

### 2.1 SRS 섹션별 변경 영향도

| SRS 섹션 | v02 현재 상태 | v03 변경 유형 | 변경 규모 | 핵심 변경 사유 |
| :--- | :--- | :--- | :--- | :--- |
| §1.1 Purpose | MVP 대상 | **Partial Edit** | 🟢 Minor | 바이브코딩 실행 컨텍스트 추가 |
| §1.2 Scope | 500 디바이스 목표 | **Major Edit** | 🔴 Major | 디바이스 목표 <50대 + MVP Core/Phase 2/Wave 2 3단계 분리 |
| §1.3 Definitions | 97개 용어 | **Addition** | 🟢 Minor | "바이브코딩", "Free Tier" 등 신규 용어 추가 |
| §1.4 References | 12개 참조 | **Addition** | 🟢 Minor | Resend, Umami, Supabase Free Tier 제약 문서 추가 |
| §1.5.1 Constraints | CON-01~12 | **Major Edit** | 🔴 Major | **CON-13~16 추가** (Free Tier 제약: Vercel Hobby, Supabase Free, Cron 일 1회, DB 500MB) |
| §1.5.2 Assumptions | ASM-01~06 | **Major Edit** | 🟡 Medium | ASM-04~06 삭제/수정 (Pro SLA 가정 제거). **ASM-07~09 추가** (Free Tier 가정) |
| §1.5.3 Dependencies | DEP-01~06 | **Partial Edit** | 🟡 Medium | DEP-01 (EMR) Wave 2로 이관. DEP-05 Vercel Cron 제약 명시. |
| **§2 Stakeholders** | 5 페르소나 | **No Change** | ⚪ None | 변경 없음 |
| **§3.1 External Systems** | 7개 시스템 | **Major Edit** | 🔴 Major | PagerDuty 제거, SMS/Kakao 제거, Amplitude → Vercel Analytics/Umami 대안 명시 |
| **§3.2 Client Applications** | 3개 클라이언트 | **Partial Edit** | 🟡 Medium | PWA → Phase 2. MVP Core에서는 일반 웹앱으로 시작. |
| **§3.3 Route Handlers** | 11개 | **Major Edit** | 🔴 Major | **11개 → 6개(MVP Core)** + 5개(Phase 2/Wave 2) 분리 |
| **§3.3.1 Server Actions** | 6개 | **Partial Edit** | 🟡 Medium | 6개 → **4개(MVP Core)** + 2개(Phase 2) 분리 |
| **§3.4 Interaction Sequences** | 4개 시퀀스 | **Major Edit** | 🔴 Major | 7개 → **2개(MVP Core)** + 나머지는 참조용으로 유지 |
| **§3.5 Use Case Diagram** | 10 UC | **Partial Edit** | 🟡 Medium | MVP Core UC 표시(Phase 태그 추가) |
| **§3.6 ERD** | 7개 모델 | **Major Edit** | 🔴 Major | **7개 → 5개** (DeadLetterEvent, Facility 제거/단순화) |
| **§3.7 Class Diagram** | 8개 클래스 | **Major Edit** | 🔴 Major | 불필요 클래스 제거/축소 (EMRWebhookHandler, OTAService 등) |
| **§3.8 Component Diagram** | 전체 아키텍처 | **Major Edit** | 🔴 Major | Free Tier 기반으로 재작성. 제거된 서비스 반영. |
| **§4.1 Functional Requirements** | 23 FR (REQ-FUNC-001~023) | **Major Edit** | 🔴 Major | Phase 태그 추가. FR-07 Won't로 하향. EMR/HMAC 관련 AC 단순화. |
| **§4.2 Non-Functional Requirements** | 20 NFR (REQ-NF-001~020) | **Major Edit** | 🔴 Major | **10개 NFR 임계치 재조정** (CTR-01~10 반영). 유료 서비스 의존 NFR 삭제/수정. |
| **§5 Traceability Matrix** | 43 행 | **Partial Edit** | 🟡 Medium | Phase 태그 추가. 테스트 방법론 업데이트 (수동 테스트 중심). |
| **§6.1 API List** | 11개 | **Major Edit** | 🔴 Major | §3.3과 동기화. 6개 MVP Core + 5개 Phase 2/Wave 2 분리. |
| **§6.2 Data Model** | 7개 모델 | **Major Edit** | 🔴 Major | §3.6과 동기화. 5개 모델로 축소. |
| **§6.3 Detailed Sequences** | 3개 시퀀스 | **Major Edit** | 🔴 Major | MVP Core 시퀀스만 유지. 나머지 참조용. |
| **§6.4 Validation Plan** | 3개 실험 | **Partial Edit** | 🟡 Medium | 디바이스 수 현실화 (<50대). |
| **§7 AI Integration** | Gemini 사양 | **Partial Edit** | 🟡 Medium | Free Tier API 할당량 명시. Flash 모델 기본값. |
| **§8 Project Structure** | 디렉토리 트리 | **Major Edit** | 🔴 Major | MVP Core에 필요한 파일만 남기고 축소. |
| **§9 Environment Variables** | 12개 변수 | **Partial Edit** | 🟡 Medium | 제거된 서비스의 변수 삭제. 8~9개로 축소. |
| **§10 Sprint Estimation** | 11 Feature Groups | **Major Edit** | 🔴 Major | 바이브코딩 기준 재추정. Phase별 분리. |
| **§11 Risk Assessment** | 6 Risks | **Major Edit** | 🔴 Major | Free Tier 관련 리스크 재정의. |
| **§12 Gap Analysis** | 8 Gaps | **Major Edit** | 🔴 Major | Free Tier 모순 해소 반영. 기존 Gap 업데이트. |
| **🆕 §13 MVP Phase 정의** | 없음 | **New Creation** | 🆕 New | Phase 0~3 + Wave 2 범위 정의. **v03의 핵심 신규 섹션.** |
| **🆕 §14 Mock Data Spec** | 없음 | **New Creation** | 🆕 New | Edge 시뮬레이터 대체용 모의 데이터 구조 정의. |
| **🆕 §15 Free Tier 제약 명세** | 없음 | **New Creation** | 🆕 New | 각 서비스별 Free Tier 한도와 초과 시 대응 전략. |

---

## 3. 상세 변경 계획

### Phase 1 — 기반 구조 수정 (Foundations)

#### 3.1 §1.2 Scope — MVP 목표 재정의

**현재 (v02):**
```
In-Scope: 500 디바이스 목표, Vercel Pro + Supabase Pro 기반
```

**변경 (v03):**

| 항목 | v02 | v03 | 변경 사유 |
| :--- | :--- | :--- | :--- |
| 디바이스 목표 | <500 (Vercel Pro) | **<50 (Free Tier Beta/Demo)** | CTR-04, CTR-10 |
| 인프라 tier | Vercel Pro + Supabase Pro | **Vercel Hobby + Supabase Free** | 완전 무료 제약 |
| MVP Core 범위 | 전체 FR/NFR 동시 | **Phase 0~3 (5~7주)로 한정** | 난이도 조정 |
| Wave 2 범위 | 명시하되 분리 불명확 | **명시적 분리 + 전문 개발자 필요 태그** | P-01 |
| B2C Portal | PWA (Service Worker + Web Push) | **일반 웹앱 → Phase 3에서 PWA 전환** | 난이도 하향 |

**추가할 In-Scope 항목:**

| 항목 | 설명 | Phase |
| :--- | :--- | :--- |
| Mock Data Generator | Edge 센서 시뮬레이터 대체. Seed Script + 모의 API | **Phase 0** |
| Email 알림 (Resend Free) | SMS/KakaoTalk 대신 무료 이메일 알림 | **Phase 2** |

**추가할 Out-of-Scope 항목 (v02에서 이관):**

| 항목 | v02 위치 | 이관 사유 |
| :--- | :--- | :--- |
| EMR Webhook (HMAC-SHA256) | FR-04 Must | 벤더 파트너십(DEP-01) 미체결 + HMAC 보안은 전문가 필요 |
| SMS/KakaoTalk 폴백 (FR-07) | Should | 유료 서비스. 완전 무료 제약 위반 |
| PagerDuty 연동 | REQ-NF-007 | 유료 서비스 ($21/인/월) |
| Cold Archival (>90일) | REQ-NF-017 | MVP 런칭 후 90일 뒤에야 필요 |
| Hash Chain 무결성 검증 | REQ-FUNC-015 일부 | 법적 증거력은 프로덕션 단계에서 필요 |
| Rate Limiting (100 req/min) | §3.3 #2 | <50 디바이스에서 불필요 |

---

#### 3.2 §1.5.1 Constraints — Free Tier 제약 조건 추가

**신규 추가할 제약 조건:**

| Constraint ID | 제약 | 영향 범위 | 대응 |
| :--- | :--- | :--- | :--- |
| **CON-13** | **Vercel Hobby 제한** — Serverless 100 GB-hr/월, 함수 10초 타임아웃, Edge Runtime 미지원, Cron 일 1회, 대역폭 100GB | §3.3 Route Handlers, Cron, NFR 전체 | Route Handler 6개 축소, Cron 일 1회, cold start 허용 |
| **CON-14** | **Supabase Free 제한** — DB 500MB, Storage 1GB, Realtime 200 동시연결, 7일 미사용 시 프로젝트 일시정지 | §3.6 ERD, §4.2 NFR | 모델 5개 축소, 데이터 30일 보존, 일시정지 방지용 ping 설정 |
| **CON-15** | **Gemini API Free 할당량** — 무료: 15 RPM(분당 요청), 1M TPM(분당 토큰). 하루 1,500 요청 제한 | §7 AI Integration | 하루 50대 × 1회 = 50 요청 → 충분. 단, burst 방지를 위해 순차 생성 |
| **CON-16** | **완전 무료 인프라 원칙** — 모든 운영 비용을 $0으로 유지. AI 코딩 도구 구독비만 예외. | 전체 | 유료 서비스(PagerDuty, SMS, Pro 플랜) 일체 배제 |

---

#### 3.3 §1.5.2 Assumptions — 수정 및 추가

**삭제/수정할 가정:**

| 기존 ID | 현재 내용 | 변경 | 사유 |
| :--- | :--- | :--- | :--- |
| ASM-04 | Vercel Pro SLA 99.99% 유지 | **삭제** | Hobby에는 SLA 보장 없음 |
| ASM-05 | Supabase Realtime 200 동시연결 충분 (500 디바이스) | **수정** → "50 디바이스 기준 충분" | 디바이스 목표 하향 |
| ASM-06 | iOS Safari 16.4+ Web Push 대다수 지원 | **유지**, 단 "Phase 3 PWA 전환 시 검증" 추가 | PWA는 Phase 3 |

**신규 추가할 가정:**

| Assumption ID | 가정 | 검증 시점 |
| :--- | :--- | :--- |
| **ASM-07** | Vercel Hobby의 100 GB-hr/월 Serverless 한도가 6개 Route Handler + <50 디바이스 운영에 충분 | Beta 2주차 사용량 모니터링 |
| **ASM-08** | Supabase Free DB 500MB가 50 디바이스 × 30일 이벤트 데이터 보존에 충분 (30일 자동 정리 기준) | 데이터 축적량 월간 점검 |
| **ASM-09** | Gemini 1.5 Flash 무료 할당량(15 RPM, 일 1,500 req)이 50 디바이스 일일 리포트 생성에 충분 | Phase 2 AI 연동 시 검증 |

---

### Phase 2 — 기능 요구사항 재조정

#### 3.4 §3.3 Route Handlers — 11개 → 6개 축소

**MVP Core (v03에 유지, 6개):**

| # | Route | Method | 설명 | Auth | Phase |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `app/api/events/ingest/route.ts` | POST | 모의 이벤트 수신 (Edge 시뮬레이터 대체) | API Key (env) | **Phase 0** |
| 2 | `app/api/reports/daily/[deviceId]/[date]/route.ts` | GET | 일일 리포트 조회 (AI 요약 포함) | JWT (NextAuth) | **Phase 1** |
| 3 | `app/api/events/[eventId]/false-alarm/route.ts` | POST | 오보 신고 피드백 | JWT (NextAuth) | **Phase 2** |
| 4 | `app/api/ai/wellness-summary/route.ts` | POST | Gemini AI 웰니스 요약 생성 | JWT (NextAuth) | **Phase 2** |
| 5 | `app/api/dashboard/status/route.ts` | GET | 대시보드 상태 조회 (정적, Realtime 없음) | JWT (NextAuth) | **Phase 1** |
| 6 | `app/api/devices/[deviceId]/heartbeat/route.ts` | POST | 디바이스 하트비트 수신 (디바이스→서버 push) | API Key (env) | **Phase 1** |

**Phase 2로 이관 (3개):**

| # | Route | 이관 사유 |
| :--- | :--- | :--- |
| 7 | `app/api/reports/trend/[deviceId]/route.ts` | Sleep Trend 차트 데이터. 기본 리포트 조회 API에서 날짜 범위 쿼리로 대체 가능. |
| 8 | `app/api/notifications/push/route.ts` | Web Push는 Phase 3 PWA 전환 시 추가. MVP에서는 Email 알림 사용. |
| 9 | `app/api/dashboard/filters/route.ts` | Dashboard 필터 저장. <50 디바이스에서는 필터 불필요. |

**Wave 2로 이관 (2개):**

| # | Route | 이관 사유 |
| :--- | :--- | :--- |
| 10 | `app/api/webhooks/emr/route.ts` | EMR 벤더 파트너십 미체결(DEP-01). HMAC-SHA256 보안은 전문가 필수. |
| 11 | `app/api/events/archive/route.ts` | 90일 아카이브 + RBAC. 무료 인프라에서 30일 보존으로 단축하면 별도 아카이브 불필요. |

---

#### 3.5 §3.3.1 Server Actions — 6개 → 4개 축소

**MVP Core (4개):**

| # | Action | Phase | 비고 |
| :--- | :--- | :--- | :--- |
| 1 | `createWellnessEvent` | Phase 0 | 모의 이벤트 저장 |
| 2 | `updateFalseAlarmFlag` | Phase 2 | 오보 신고 |
| 3 | `generateDailyReport` | Phase 2 | 일일 리포트 + AI 요약 |
| 4 | `updateDeviceStatus` | Phase 1 | 하트비트 상태 업데이트 |

**Phase 2 이관 (2개):**

| # | Action | 이관 사유 |
| :--- | :--- | :--- |
| 5 | `saveDashboardFilter` | <50 디바이스에서 불필요 |
| 6 | `createUser` | NextAuth.js 기본 가입 흐름으로 대체 가능 |

---

#### 3.6 §3.6 ERD — 7개 모델 → 5개 모델

**MVP Core (5개 모델):**

| 모델 | 변경 | 비고 |
| :--- | :--- | :--- |
| **SensorDevice** | 유지 (일부 필드 단순화) | `facilityId` 제거 (Facility 모델 제거). `locationZone`을 "BEDROOM" 고정. |
| **WellnessEvent** | 유지 (일부 필드 단순화) | `integrityHash` 필드 유지하되 체인 검증 로직 생략. `zone`을 "BEDROOM" 고정. |
| **UserAccount** | 유지 (단순화) | `facilityId` 제거. `role`은 유지 (GUARDIAN/ADMIN). |
| **UserDevice** | 유지 | M:N 조인 테이블. 변경 없음. |
| **DailyReport** | 유지 | `aiSummary` 필드 유지. |

**제거 (2개 모델):**

| 모델 | 제거 사유 |
| :--- | :--- |
| **Facility** | B2B 시설 관리는 Wave 2. MVP에서는 시설 정보를 환경변수 또는 하드코딩으로 대체. |
| **DeadLetterEvent** | EMR Webhook 자체가 Wave 2로 이관. 실패 이벤트 관리 테이블 불필요. |

---

#### 3.7 §4.1 Functional Requirements — Phase 태그 및 우선순위 재조정

**변경 대상 FR 및 REQ:**

| FR | v02 Priority | v03 Priority | Phase | 주요 변경 |
| :--- | :--- | :--- | :--- | :--- |
| FR-01 (AI Engine) | Must | Must **(Edge/HW — 바이브코딩 범위 외)** | Wave 2 | 태그 추가: "전문 개발자 필수. 바이브코딩 범위 외." |
| FR-02 (Sensor) | Must | Must **(Edge/HW — 바이브코딩 범위 외)** | Wave 2 | 동일 |
| FR-03 (Privacy Edge) | Must | Must **(Edge/HW — 바이브코딩 범위 외)** | Wave 2 | 동일 |
| FR-04 (Dashboard + EMR) | Must | **Must (Dashboard Only)** / Could (EMR) | Phase 1 / Wave 2 | Dashboard UI는 MVP Core. EMR HMAC은 Wave 2로 분리. |
| FR-05 (Wellness Report) | Should | **Must** ⬆️ | Phase 2 | **MVP 킬러 피처로 격상.** 바이브코딩 최적 영역. |
| FR-06 (Trend Charts) | Could | Could | Phase 3 | 변경 없으나 Phase 태그 추가 |
| **FR-07 (SMS/Kakao)** | **Should** | **Won't (MVP)** ⬇️ | Wave 2 | 유료 서비스. 무료 대안(Email)으로 대체. |
| FR-08 (Dashboard Filter) | Could | Could | Phase 3 | <50 디바이스에서 불필요 |

**REQ별 Acceptance Criteria 수정 대상:**

| REQ ID | 현재 AC | v03 변경 | 사유 |
| :--- | :--- | :--- | :--- |
| REQ-FUNC-004 | "guardian push arrives under 60 seconds via FCM/Web Push" | "guardian receives **email notification** within 5 minutes" | MVP에서 FCM/Web Push 미구현. Email 알림으로 대체 (Phase 2). |
| REQ-FUNC-008 | "push message is fired one time to guardians/admins via FCM/Web Push" | "email notification sent to guardians. Dashboard status updated to INACTIVE" | FCM/Web Push → Email 전환. |
| REQ-FUNC-011 | "receiving a status change via **Supabase Realtime** subscription" | "UI displays status from **periodic API polling (30s interval)**" | Supabase Realtime 구독은 Phase 2. MVP Core에서는 폴링. |
| REQ-FUNC-013 | "Route Handler transfers data via HTTP POST with HMAC-SHA256 signature" | **Wave 2로 전체 이관** | EMR HMAC 보안은 전문가 필수 |
| REQ-FUNC-014 | "Route Handler retries 3 times with exponential backoff... creates DeadLetterEvent" | **Wave 2로 전체 이관** | DeadLetterEvent 모델 제거 |
| REQ-FUNC-015 | "Events >90 days are archived to Supabase Storage... integrity hashes preserved" | "Events older than **30 days** are **automatically deleted** via daily cleanup query. integrityHash field is preserved for future verification." | 90일→30일. Cold Archival 제거. 단순 삭제로 대체. |
| REQ-FUNC-020 | "Vercel Cron triggers at 07:30 KST... dispatches report notifications via FCM/Web Push" | "Vercel Cron triggers once daily... report is generated and stored. **Email notification sent via Resend Free** (100 emails/day)." | Cron 일 1회 + Email 전환 |
| REQ-FUNC-022 | "user acquires Web Push + SMS/KakaoTalk as fallback" | **Won't (MVP). Wave 2.** | FR-07 삭제 |

---

### Phase 3 — 비기능 요구사항 재조정

#### 3.8 §4.2 Non-Functional Requirements — 임계치 재조정

| NFR ID | v02 임계치 | v03 임계치 | 변경 사유 (CTR 매핑) |
| :--- | :--- | :--- | :--- |
| **REQ-NF-001** | p95 ≤ 2,000ms E2E (Edge Runtime) | **p95 ≤ 5,000ms** (Serverless cold start 허용). "긴급 알림 수신이 아닌 리포트/대시보드 조회 기준" | CTR-02. Hobby Edge Runtime 미지원. |
| **REQ-NF-002** | ≤ 0.3 events/month/home | ≤ 0.3 events/month/home **(변경 없음 — Edge AI 영역)** | Edge 독립. |
| **REQ-NF-004** | p95 ≤ 500ms at 1,000 active nodes | **p95 ≤ 1,000ms at 50 active nodes.** "Supabase Free DB 기준" | CTR-09, CTR-10. 디바이스 목표 하향. |
| **REQ-NF-005** | SLA ≥ 99.9% (Vercel Pro + Supabase Pro) | **Best Effort, 목표 ~99%.** "무료 티어 기반. SLA 보장 없음. Supabase Free 7일 일시정지 리스크 존재." | CTR-01 |
| **REQ-NF-007** | PagerDuty Sev1 자동 에스컬레이션 | **삭제. Slack/Discord Incoming Webhook(무료)으로 대체.** 조건: 오프라인 디바이스 비율 ≥ 10% 시 경고. | CTR-03 |
| **REQ-NF-008** | 100% TLS 1.3 (Vercel 기본 제공) | **유지 (Vercel Hobby도 TLS 1.3 기본 적용)** | 변경 없음 |
| **REQ-NF-011** | RBAC via NextAuth.js role middleware | **JWT 단일 인증. RBAC는 Phase 2.** "MVP Core에서는 role 필드는 DB에 저장하되, 미들웨어 강제는 Phase 2에서 적용." | 난이도 조정 |
| **REQ-NF-012** | ≤ 500 KRW/Unit/Month | **$0/Unit/Month (Free Tier 운영).** "Free Tier 제약 내에서 운영 가능한 <50 디바이스 기준. 유료 전환 시 기존 ≤500원 목표 복원." | CTR-04 |
| **REQ-NF-013** | OTA Deploy Success ≥ 99% | **Wave 2 이관.** "Edge/FW 영역. 바이브코딩 범위 외." | 난이도 조정 |
| **REQ-NF-017** | Hot 90일 + Cold >3년 (Supabase Storage Cron) | **Hot 30일. 자동 삭제.** "Supabase Free 500MB 제약. Cold Archival은 Wave 2에서 유료 전환 후 구현." | CTR-10 |
| **REQ-NF-018** | <500 devices (Vercel Pro) | **<50 devices (Vercel Hobby + Supabase Free)** | CTR-09 |

---

### Phase 4 — 신규 섹션 추가

#### 3.9 🆕 §13 MVP Phase 정의 (신규)

SRS v03의 **핵심 신규 섹션**. 모든 FR/NFR에 Phase 태그를 부여하여 바이브코딩 실행자가 "지금 뭘 만들어야 하는지" 즉시 판단 가능하도록 합니다.

**Phase 구조:**

| Phase | 기간 | 범위 | 인프라 | 개발 방식 |
| :--- | :--- | :--- | :--- | :--- |
| **Phase 0: 기반** | 1주 | Next.js + Prisma 5모델 + Vercel 배포 + Seed Data | Vercel Hobby + Supabase Free | 바이브코딩 |
| **Phase 1: 핵심 UI** | 2주 | Guardian Dashboard + B2B Dashboard + 로그인(NextAuth.js) + Daily Report 조회 | 동일 | 바이브코딩 |
| **Phase 2: 파이프라인** | 2주 | 모의 이벤트 수신 API + Daily Report 생성(Cron) + Gemini AI 요약 + Email 알림 + 오보 피드백 | 동일 | 바이브코딩 |
| **Phase 3: 부가** | 1~2주 | Sleep Trend 차트 + PWA 전환(Service Worker) + Dashboard 필터 + Amplitude 트래킹 | 동일 | 바이브코딩 + 전문가 리뷰 |
| **Wave 2: 확장** | 별도 | Edge AI/HW 통합 + EMR HMAC + SMS/카카오 + PagerDuty + Cold Archival + RBAC 강화 | Vercel Pro + Supabase Pro | 전문 개발자 필수 |

---

#### 3.10 🆕 §14 Mock Data Specification (신규)

Edge 센서 없이 웹 파트를 개발/테스트하기 위한 **모의 데이터 사양**.

**내용 정의:**

| 항목 | 사양 |
| :--- | :--- |
| **Seed Script 위치** | `prisma/seed.ts` |
| **모의 디바이스** | 3~5대 (BEDROOM 고정) |
| **모의 유저** | Guardian 2명, Admin 1명 |
| **모의 WellnessEvent** | 7일분, 5분 간격, eventType 분포: ACTIVITY_ALERT 95%, WELLNESS_SCORE 4%, EMERGENCY 1% |
| **모의 DailyReport** | 7일분, sleepScore 60~95 랜덤, bathroomVisitCount 1~5 랜덤 |
| **모의 AI Summary** | 하드코딩된 3~5개 예시 문장 (Gemini API 호출 없이 테스트 가능) |
| **이벤트 생성 API** | `app/api/events/ingest/route.ts`에 `mock=true` 쿼리 파라미터 시 자동 생성 모드 |

---

#### 3.11 🆕 §15 Free Tier 제약 명세 (신규)

| 서비스 | 무료 플랜 | 핵심 제약 | MVP 사용량 (50대) | 여유도 | 초과 시 대응 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Vercel Hobby** | 무료 | 100 GB-hr Serverless, 함수 10초 타임아웃, Cron 일 1회, Edge Runtime 미지원 | ~20 GB-hr 예상 | 🟢 충분 | Pro 업그레이드 ($20/월) |
| **Supabase Free** | 무료 | 500MB DB, 1GB Storage, 50K MAU, Realtime 200 동시연결, **7일 미사용 시 일시정지** | ~100MB DB 예상 | 🟢 충분 | Pro 업그레이드 ($25/월). 일시정지 방지: 일일 ping. |
| **Gemini 1.5 Flash** | 무료 | 15 RPM, 1M TPM, 일 1,500 요청 | 50 요청/일 | 🟢 충분 | gemini-1.5-flash로 비용 최소화 |
| **NextAuth.js** | 무료 (OSS) | 제약 없음 | - | 🟢 - | - |
| **Resend (Email)** | 무료 | 100 이메일/일, 3,000/월 | 50 이메일/일 | 🟢 충분 | 유료 $20/월 (50K/월) |
| **GitHub** | 무료 | 무제한 public repo, Actions 2,000분/월 | - | 🟢 - | - |

---

### Phase 5 — 다이어그램 및 구조 업데이트

#### 3.12 §3.4 Interaction Sequences — 7개 → 2개 (MVP Core)

**MVP Core 유지 (2개):**

| 시퀀스 | 변경 사항 |
| :--- | :--- |
| **§3.4.1 Daily Wellness Report** | Vercel Cron(자정, 일 1회) → Server Action → Prisma → Gemini Flash → DailyReport 저장 → **Email 알림(Resend)**. FCM/Web Push 참조 제거. |
| **§3.4.3 PMF Diagnostic** | 변경 없음 (Amplitude 트래킹 유지) |

**참조용 유지 (Wave 2 명시, 5개):**

| 시퀀스 | 상태 |
| :--- | :--- |
| §3.4.2 Zero False Alarm AI Validator | "Wave 2 — Edge/HW 영역" 태그 |
| §3.4.4 EMR System Sync | "Wave 2 — EMR 파트너십 후" 태그 |
| §6.3.1 Fall Detection E2E | "Wave 2" 태그 |
| §6.3.2 Device Offline → PagerDuty | PagerDuty → Slack Webhook으로 수정 + "Phase 3" 태그 |
| §6.3.3 OTA Firmware | "Wave 2 — Edge/FW 영역" 태그 |

---

#### 3.13 §3.7 Class Diagram — 축소

**제거할 클래스:**

| 클래스 | 제거 사유 |
| :--- | :--- |
| `EMRWebhookHandler` | EMR HMAC Wave 2 이관 |
| `OTAService` | Edge/FW 영역. Wave 2. |
| `HeartbeatCronJob` | Cron 일 1회 제한. 하트비트는 디바이스 push → Route Handler 방식으로 변경. 별도 클래스 불필요. |

**단순화할 클래스:**

| 클래스 | 변경 |
| :--- | :--- |
| `PushNotificationService` | FCM/Web Push 제거 → `EmailNotificationService` (Resend API)로 교체 |
| `TriageEngine` | 유지하되 "Phase 3" 태그. MVP Core에서는 단순 정렬. |

---

#### 3.14 §3.8 Component Diagram — Free Tier 기반 재작성

**제거할 컴포넌트:**

| 컴포넌트 | 제거 사유 |
| :--- | :--- |
| PagerDuty | 유료. Slack/Discord Webhook으로 대체. |
| FCM (Firebase Cloud Messaging) | Phase 2 이관. MVP Core에서는 Email. |
| Web Push API (VAPID) | Phase 3 PWA 전환 시 추가. |
| EMR System 연결선 | Wave 2 이관. |
| Cold Archival (Supabase Storage) | Wave 2 이관. |

**추가할 컴포넌트:**

| 컴포넌트 | 역할 |
| :--- | :--- |
| **Resend (Email API)** | 무료 이메일 알림 (100/일) |
| **Slack/Discord Webhook** | 운영 알림 (PagerDuty 대체) |
| **Mock Data Generator** | Edge 시뮬레이터 대체 |

---

#### 3.15 §8 Project Structure — MVP Core 축소

**제거할 파일/디렉토리:**

| 경로 | 제거 사유 |
| :--- | :--- |
| `app/api/webhooks/emr/` | EMR Wave 2 |
| `app/api/notifications/push/` | FCM/Web Push Phase 2 이관 |
| `app/api/events/archive/` | Archive Wave 2 |
| `app/api/dashboard/filters/` | Filter Phase 3 |
| `app/api/reports/trend/` | Trend Phase 2 (daily API에서 범위 쿼리로 대체) |
| `app/actions/dashboard.ts` | Filter Phase 3 |
| `app/actions/users.ts` | NextAuth.js 기본 흐름으로 대체 |
| `components/dashboard/emr-status-banner.tsx` | EMR Wave 2 |
| `lib/emr-webhook.ts` | EMR Wave 2 |
| `lib/push.ts` | FCM/Web Push Phase 2 |

**추가할 파일:**

| 경로 | 역할 |
| :--- | :--- |
| `lib/email.ts` | Resend API 유틸리티 |
| `lib/slack.ts` | Slack/Discord Webhook 유틸리티 |
| `prisma/seed.ts` | 모의 데이터 생성 스크립트 (§14에서 정의) |
| `app/api/mock/generate/route.ts` | 모의 이벤트 자동 생성 API (개발/데모용) |

---

#### 3.16 §9 Environment Variables — 12개 → 8개

**제거할 변수:**

| 변수 | 제거 사유 |
| :--- | :--- |
| `FCM_SERVER_KEY` | FCM Phase 2 이관 |
| `NEXT_PUBLIC_VAPID_PUBLIC_KEY` | Web Push Phase 3 |
| `VAPID_PRIVATE_KEY` | Web Push Phase 3 |
| `EMR_WEBHOOK_API_KEY` | EMR Wave 2 |
| `EMR_WEBHOOK_HMAC_SECRET` | EMR Wave 2 |

**추가할 변수:**

| 변수 | 용도 |
| :--- | :--- |
| `RESEND_API_KEY` | Resend 무료 이메일 API 키 |
| `SLACK_WEBHOOK_URL` | Slack/Discord 운영 알림 Webhook URL (선택) |

**최종 변수 목록 (8개):**

```env
# Database (Supabase Free)
DATABASE_URL="postgresql://..."

# Auth (NextAuth.js)
NEXTAUTH_SECRET="..."
NEXTAUTH_URL="http://localhost:3000"

# AI (Gemini Free)
GOOGLE_GENERATIVE_AI_API_KEY="..."
AI_MODEL="gemini-1.5-flash"           # Flash 기본 (무료 할당량 최적화)

# Email Notification (Resend Free)
RESEND_API_KEY="..."

# Analytics (선택)
NEXT_PUBLIC_AMPLITUDE_KEY="..."       # 또는 Vercel Analytics 사용 시 제거

# Ops Alert (선택)
SLACK_WEBHOOK_URL="..."               # 운영 경고용 Slack/Discord
```

---

### Phase 6 — Sprint 추정 / Risk / Gap 업데이트

#### 3.17 §10 Sprint Estimation — 바이브코딩 기준 재추정

| Feature Group | v02 추정 | v03 추정 (바이브코딩) | Phase | 변경 사유 |
| :--- | :--- | :--- | :--- | :--- |
| FR-01: AI Engine | XL (3-4) | **N/A** (바이브코딩 범위 외) | Wave 2 | Edge/HW |
| FR-02: Sensor | L (2-3) | **N/A** | Wave 2 | Edge/HW |
| FR-03: Privacy Edge | L (2-3) | **N/A** | Wave 2 | Edge/HW |
| FR-04: Dashboard (UI only) | M (1-2) | **S (0.5~1 Sprint, 1~2주)** | Phase 1 | EMR 분리로 축소 |
| FR-05: Wellness Report + AI | M (1-2) | **M (1 Sprint, 2주)** | Phase 2 | MVP 핵심. AI SDK 적합. |
| FR-06: Sleep Charts | S (1) | **S (0.5 Sprint, 1주)** | Phase 3 | Recharts 단순 |
| FR-07: SMS/Kakao | S (1) | **N/A** (Won't) | Wave 2 | 유료 서비스 |
| FR-08: Dashboard Filter | S (0.5-1) | **XS (0.5 Sprint, 3~5일)** | Phase 3 | <50대 불필요하지만 Could |
| PWA Guardian Portal | M (1-2) | **S (0.5 Sprint, 1주)** | Phase 3 | manifest + SW 기본만 |
| AI Wellness Summary | S (0.5-1) | **포함** (FR-05에 통합) | Phase 2 | 별도 추정 불필요 |
| Infra Setup | S (0.5-1) | **XS (2~3일)** | Phase 0 | create-next-app + Vercel 배포 |
| **Mock Data + Seed** | N/A (신규) | **XS (1~2일)** | Phase 0 | 신규 |
| **Email Notification** | N/A (신규) | **XS (1~2일)** | Phase 2 | SMS/FCM 대체 |
| **총 바이브코딩 소요** | - | **약 5~7주 (풀타임)** | Phase 0~3 | |

---

#### 3.18 §11 Risk Assessment — Free Tier 리스크 반영

**제거할 리스크:**

| Risk ID | 내용 | 제거 사유 |
| :--- | :--- | :--- |
| RISK-01 | Vercel cold starts → p95 latency | Edge Runtime 미지원. 임계치 완화로 해소. |
| RISK-03 | Supabase Realtime 연결 제한 | <50 디바이스로 하향. |
| RISK-04 | 5K 디바이스 확장성 | MVP <50대. Wave 2 과제. |

**수정할 리스크:**

| Risk ID | v02 | v03 |
| :--- | :--- | :--- |
| RISK-02 | iOS Safari Web Push 커버리지 | "PWA는 Phase 3. MVP Core에서는 Email 알림 사용" |
| RISK-05 | SQLite → PostgreSQL 마이그레이션 | "MVP는 Supabase Free PostgreSQL 직접 사용. SQLite는 로컬 테스트 전용." |
| RISK-06 | Gemini API 비용/속도 | "Flash 모델 기본. 무료 할당량(일 1,500 req) 내 운영. <50 디바이스에서 충분." |

**신규 추가할 리스크:**

| Risk ID | 리스크 | 확률 | 영향 | 대응 |
| :--- | :--- | :--- | :--- | :--- |
| **RISK-07** | Supabase Free 프로젝트 7일 미사용 시 **일시정지** → 서비스 중단 | 3/5 | 4/5 | GitHub Actions cron 또는 외부 uptime monitor (UptimeRobot 무료)로 일일 ping 설정 |
| **RISK-08** | Vercel Hobby **Serverless 100 GB-hr** 초과 → 429 에러 | 2/5 | 3/5 | Route Handler 최소화 (6개), 함수 코드 최소화, 불필요한 API 호출 REDUCE |
| **RISK-09** | Resend Free **100 이메일/일** 초과 → 알림 미전송 | 2/5 | 2/5 | <50 디바이스에서는 안전. 긴급 알림만 이메일 전송, 일반 리포트는 앱 내 조회 |
| **RISK-10** | 바이브코딩 AI가 **NextAuth.js 설정 오류** 생성 → 인증 우회 보안 허점 | 3/5 | 4/5 | MVP core에서는 demo credentials 사용. 실제 배포 전 보안 체크리스트 수행. |

---

#### 3.19 §12 Gap Analysis — 업데이트

**수정할 Gap:**

| Gap ID | v02 | v03 변경 |
| :--- | :--- | :--- |
| GAP-01 | WebSocket → Supabase Realtime | "**MVP Core: API 폴링 (30초)**. Phase 2에서 Supabase Realtime 전환" |
| GAP-02 | iOS Native → PWA | "**MVP Core: 일반 웹앱**. Phase 3에서 PWA 전환(manifest + SW)" |
| GAP-03 | Cold Archival → Supabase Storage Cron | "**MVP: 30일 보존, 자동 삭제.** Wave 2에서 Cold Archival 구현" |
| GAP-05 | PagerDuty 실시간 모니터링 | "**MVP: Slack/Discord Webhook.** Wave 2에서 PagerDuty 검토" |
| GAP-08 | Dead Letter Queue | "**제거.** DeadLetterEvent 모델 삭제. EMR Wave 2에서 필요 시 재도입" |

**신규 Gap:**

| Gap ID | 설명 | 심각도 | 대응 |
| :--- | :--- | :--- | :--- |
| **GAP-09** | **긴급 알림 지연** — Edge Runtime 미지원으로 cold start 3~5초 + Email 전송 지연 → 실시간 긴급 알림 불가 | 🔴 High | MVP는 "리포트/대시보드 조회" 중심. 실시간 긴급 알림은 Wave 2 (Edge AI + FCM/Web Push). 이를 SRS에 명확히 명시. |
| **GAP-10** | **모의 데이터 의존** — 실제 센서 없이 모의 데이터로만 운영. 데이터 품질/패턴이 비현실적일 수 있음. | 🟡 Medium | Seed Script에 현실적인 데이터 분포 반영. Beta 시 실제 센서 일부 연동 테스트. |

---

## 4. 실행 순서 및 일정

### 4.1 SRS 수정 작업 순서

```
Phase 1: 기반 구조 (§1 수정)
  ├─ §1.2 Scope 재정의 (MVP Core / Phase 2 / Wave 2 분리)
  ├─ §1.5.1 Free Tier 제약 조건 추가 (CON-13~16)
  ├─ §1.5.2 가정 수정/추가 (ASM 수정)
  └─ §1.5.3 의존성 조정

Phase 2: 기능 요구사항 조정 (§3, §4.1 수정)
  ├─ §3.3 Route Handler 11→6개 축소
  ├─ §3.3.1 Server Action 6→4개 축소
  ├─ §3.6 ERD 7→5개 모델 축소
  ├─ §4.1 FR Phase 태그 + 우선순위 재조정
  └─ §4.1 개별 REQ AC 수정 (FCM/Web Push → Email 등)

Phase 3: 비기능 요구사항 조정 (§4.2 수정)
  ├─ NFR 임계치 재조정 (10개 항목)
  └─ 유료 서비스 의존 NFR 무료 대안으로 교체

Phase 4: 신규 섹션 추가
  ├─ 🆕 §13 MVP Phase 정의
  ├─ 🆕 §14 Mock Data Specification
  └─ 🆕 §15 Free Tier 제약 명세

Phase 5: 다이어그램 / 구조 업데이트
  ├─ §3.4 Interaction Sequences (7→2 Core + 5 참조)
  ├─ §3.7 Class Diagram 축소
  ├─ §3.8 Component Diagram 재작성
  ├─ §8 Project Structure 축소
  └─ §9 Environment Variables 12→8개

Phase 6: 보조 문서 업데이트
  ├─ §5 Traceability Matrix Phase 태그 추가
  ├─ §10 Sprint Estimation 재추정
  ├─ §11 Risk Assessment 재정의
  └─ §12 Gap Analysis 업데이트

Phase 7: 검증
  ├─ Traceability Matrix 무결성 확인
  ├─ Free Tier 제약 vs. NFR 정합성 확인
  ├─ ISO/IEC/IEEE 29148:2018 구조 준수 확인
  └─ 모든 REQ-ID가 Phase / Wave 태그를 보유하는지 확인
```

### 4.2 작업 체크리스트

- [ ] **Phase 1: 기반 구조**
  - [ ] §1.2 Scope: 디바이스 목표 <50대, 3단계 범위 분리
  - [ ] §1.2 In-Scope: Mock Data Generator, Email 알림 추가
  - [ ] §1.2 Out-of-Scope: EMR HMAC, SMS/Kakao, PagerDuty, Cold Archival 추가
  - [ ] §1.5.1: CON-13~16 (Free Tier 제약) 추가
  - [ ] §1.5.2: ASM-04 삭제, ASM-05 수정, ASM-07~09 추가
  - [ ] §1.5.3: DEP-01 Wave 2 표기, DEP-05 Cron 일 1회 제약 표기

- [ ] **Phase 2: 기능 요구사항**
  - [ ] §3.3: Route Handler 11→6개 (5개는 Phase 2/Wave 2 태그)
  - [ ] §3.3.1: Server Action 6→4개 (2개는 Phase 2 태그)
  - [ ] §3.6: Prisma 모델 7→5개 (Facility, DeadLetterEvent 제거)
  - [ ] §4.1 FR-04: Dashboard(Must)/EMR(Wave 2) 분리
  - [ ] §4.1 FR-05: Should→Must 격상
  - [ ] §4.1 FR-07: Should→Won't 하향
  - [ ] §4.1 REQ-FUNC-004, 008, 011, 013~015, 020, 022: AC 수정

- [ ] **Phase 3: 비기능 요구사항**
  - [ ] REQ-NF-001: p95 2,000ms → 5,000ms
  - [ ] REQ-NF-004: 1,000 nodes → 50 nodes
  - [ ] REQ-NF-005: SLA 99.9% → Best Effort ~99%
  - [ ] REQ-NF-007: PagerDuty → Slack/Discord Webhook
  - [ ] REQ-NF-011: RBAC → JWT 단일 (RBAC Phase 2)
  - [ ] REQ-NF-012: ≤500원 → $0 (Free Tier)
  - [ ] REQ-NF-017: Hot 90일 → 30일, Cold Archival 제거
  - [ ] REQ-NF-018: <500 → <50 디바이스

- [ ] **Phase 4: 신규 섹션**
  - [ ] 🆕 §13 MVP Phase 정의 (Phase 0~3 + Wave 2 상세 표)
  - [ ] 🆕 §14 Mock Data Specification (Seed Script 사양)
  - [ ] 🆕 §15 Free Tier 제약 명세

- [ ] **Phase 5: 다이어그램 / 구조**
  - [ ] §3.4: 시퀀스 다이어그램 2개 Core + 5개 참조 태그
  - [ ] §3.7: Class Diagram 3개 클래스 제거, 1개 교체
  - [ ] §3.8: Component Diagram Free Tier 기반 재작성
  - [ ] §8: Project Structure 축소 (제거/추가 파일 반영)
  - [ ] §9: env 변수 12→8개

- [ ] **Phase 6: 보조 문서**
  - [ ] §5: 각 행에 Phase/Wave 태그 추가
  - [ ] §10: 바이브코딩 기준 Sprint 재추정
  - [ ] §11: RISK-01,03,04 제거. RISK-07~10 추가
  - [ ] §12: GAP-01,02,03,05,08 수정. GAP-09,10 추가.

- [ ] **Phase 7: 검증**
  - [ ] Traceability Matrix 무결성 (모든 REQ-ID 연결 확인)
  - [ ] Free Tier 제약 vs. NFR 목표 정합성 확인
  - [ ] ISO 29148 구조 준수 (필수 섹션 누락 없음)
  - [ ] Phase/Wave 태그 100% 부여 확인
  - [ ] 제거된 기능의 잔여 참조 정리 (dangling reference 없음)

---

## 5. 산출물 정의

### 5.1 최종 산출물

| # | 산출물 | 파일명 | 설명 |
| :--- | :--- | :--- | :--- |
| 1 | **바이브코딩 MVP SRS v03** | `SRS_v03(ENG_OPUS_MVP_VIBE).md` | Free Tier + 바이브코딩 최적화된 최종 SRS |
| 2 | **Revision History** | SRS 내 Revision History | v02→v03 변경 요약 |

### 5.2 품질 기준

| # | 기준 | 검증 방법 |
| :--- | :--- | :--- |
| 1 | ISO/IEC/IEEE 29148:2018 구조 준수 | 섹션 매핑 체크리스트 |
| 2 | **모든 REQ-ID에 Phase/Wave 태그 부여** | 트레이서빌리티 매트릭스 전수 검사 |
| 3 | **Free Tier 제약과 NFR 목표 간 모순 0건** | CTR-01~10 해소 확인 |
| 4 | **유료 서비스 의존 0건** (MVP Core 범위) | 외부 시스템/환경변수 전수 점검 |
| 5 | Mermaid 다이어그램 정상 렌더링 | Markdown 프리뷰 확인 |
| 6 | 바이브코딩 실행자가 "Phase 0부터 즉시 착수 가능" | §13 MVP Phase 정의가 자기충족적인지 확인 |

---

## 6. 결론 및 승인 요청

### 6.1 핵심 변경 요약

| 카테고리 | v02 (현재) | v03 (계획) |
| :--- | :--- | :--- |
| **인프라** | Vercel Pro + Supabase Pro ($45~120/월) | **Vercel Hobby + Supabase Free ($0/월)** |
| **디바이스 목표** | <500대 | **<50대 (Beta/Demo)** |
| **Route Handler** | 11개 | **6개 (MVP Core)** |
| **Prisma 모델** | 7개 | **5개** |
| **인증** | JWT + RBAC + HMAC + API Key (4종) | **JWT 단일** |
| **알림** | FCM + Web Push + SMS/Kakao | **Email (Resend Free)** |
| **모니터링** | PagerDuty ($21/월) | **Slack/Discord Webhook ($0)** |
| **SLA** | ≥ 99.9% (Pro SLA 기반) | **Best Effort ~99%** |
| **p95 레이턴시** | ≤ 2,000ms (Edge Runtime) | **≤ 5,000ms (Serverless cold start 허용)** |
| **데이터 보존** | Hot 90일 + Cold >3년 | **Hot 30일 + 자동 삭제** |
| **AI 모델** | Gemini 1.5 Pro | **Gemini 1.5 Flash (무료 할당량 최적화)** |
| **LLM 비용** | $5~15/월 | **$0 (무료 할당량 내)** |
| **전체 운영 비용** | $70~120/월 | **$0/월** |
| **개발 기간** | 12~18 스프린트 (전체) | **5~7주 (바이브코딩, 웹 파트)** |
| **새로운 섹션** | - | §13 Phase 정의, §14 Mock Data, §15 Free Tier 명세 |

### 6.2 승인 요청 항목

본 계획의 실행 전 아래 사항에 대한 확인이 필요합니다:

> **1. MVP Core에서 "긴급 알림(실시간 푸시)" 기능의 제외를 수용할 수 있는가?**
> - v03에서는 FCM/Web Push를 Phase 2 이후로 이관합니다.
> - MVP Core의 알림은 **Email(Resend Free)** 기반이므로 실시간성이 없습니다.
> - 이는 SRS의 핵심 가치 "#1 Zero False Alarm"의 전달 수단(즉시 알림)에 영향을 줍니다.
> - Edge AI 자체는 변경 없으나, 웹 알림 전달이 지연됩니다.

> **2. SLA를 "Best Effort ~99%"로 완화하는 것을 수용할 수 있는가?**
> - Supabase Free는 7일 미사용 시 프로젝트 일시정지 리스크가 있습니다.
> - 이를 ping으로 우회하더라도 SLA 보장이 불가합니다.

> **3. 데이터 보존 기간을 90일→30일로 단축하는 것을 수용할 수 있는가?**
> - Supabase Free DB 500MB 제약 때문입니다.
> - 법적 증거력(장영희 극단 사례)과의 충돌이 발생합니다.

> **4. EMR 연동을 Wave 2로 완전히 이관하는 것을 수용할 수 있는가?**
> - B2B 가치 제안의 핵심인 "이중 입력 제거"가 MVP에서 시연 불가합니다.

---

**— 계획서 끝 —**
