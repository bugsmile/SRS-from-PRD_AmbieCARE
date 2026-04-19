# MVP 개발 목표 적절성 종합 검토 보고서
## — 난이도 / 구현 가능성 / 시스템·비용 효율성 통합 분석 —

**문서 ID:** REVIEW-SRS-COMPREHENSIVE-001  
**검토 대상:** `SRS_v01(ENG_OPUS_MVP).md` (Revision 2.0, 2026-04-19)  
**보조 참조:** `SRS_v01(Eng_Opus)_Plan_eng.md` (PLAN-SRS-001-MVP)  
**검토 일자:** 2026-04-19  
**검토 관점:**

| # | 관점 | 설명 |
| :--- | :--- | :--- |
| A | 난이도 & 구현 가능성 | 초급자(3개월) + IT기획 3년차 + 바이브코딩 기반 MVP 개발 가능성 |
| B | 개발 속도 | MVP 단계의 과도한 시간 소요 여부 — Time-to-MVP 최적화 |
| C | 외부 연동 & 기술 스택 | 복잡성, 오픈소스 여부, 벤더 락인 리스크 |
| D | 운영 소요 비용 | 클라우드 서비스(Vercel/Supabase/Gemini 등) 기반 월간 비용 산정 |

---

## Part I. 종합 판정 요약 (Executive Summary)

| 평가 축 | 판정 | 핵심 결론 |
| :--- | :--- | :--- |
| **난이도 & 구현 가능성** | ⚠️ 조건부 가능 | 웹 파트 ~60%는 바이브코딩 가능. Edge AI/HW/FW는 전문가 필수. |
| **개발 속도** | ❌ 과도 | 23 FR + 20 NFR, 11 Route Handler, 7 Mermaid 시퀀스 → MVP에 불필요한 엔터프라이즈급 명세. **Time-to-MVP 6~8개월(전체) vs. 웹 파트만 5~7주**로 괴리 큼. |
| **외부 연동 & 기술 스택** | ⭕ 적절 (일부 과잉) | Next.js + Prisma + Supabase + Vercel 조합은 바이브코딩 MVP에 최적. 단, EMR HMAC, PagerDuty, Supabase Realtime 동시 사용은 과잉. |
| **운영 비용** | ⭕ 합리적 | 월 예상 비용 $50~$120 (MVP <500 디바이스 기준). 다만 Gemini API 비용 관리와 Vercel Pro 구독 필요. |

---

## Part II. 관점 A — 난이도 & 구현 가능성 (요약)

> 본 파트는 `SRS_V02_check_difficulty.md`의 핵심 결론을 요약합니다. 상세 분석은 해당 문서를 참조하세요.

### A-1. 기능별 바이브코딩 적합성 매트릭스

| FR | 기능 | MoSCoW | 바이브코딩 | 핵심 판단 근거 |
| :--- | :--- | :--- | :--- | :--- |
| FR-01 | AI 필터링 엔진 (Edge) | Must | ❌ 불가 | 딥러닝 모델 + UWB 신호처리 + 임베디드 추론 |
| FR-02 | 센서 모듈 (HW/FW) | Must | ❌ 불가 | 물리 하드웨어 + 펌웨어 개발 |
| FR-03 | 비영상 프라이버시 (Edge) | Must | ❌ 불가 | 레이더 원시 데이터 비식별화 |
| FR-04 | B2B 대시보드 + EMR | Must | 🔺 부분적 | UI ⭕ / EMR HMAC 보안 ❌ |
| FR-05 | 일일 웰니스 리포트 + AI | Should | ⭕ 가능 | **바이브코딩 최적 영역** |
| FR-06 | 수면 트렌드 차트 | Could | ⭕ 가능 | Recharts 컴포넌트 |
| FR-07 | SMS/카카오톡 폴백 | Should | 🔺 조건부 | API 키 발급 등 사전 절차 필요 |
| FR-08 | 대시보드 필터 | Could | ⭕ 가능 | shadcn/ui DataTable |

### A-2. 핵심 결론

- **Must 등급의 FR-01~03(10개 REQ)이 모두 바이브코딩 불가** → SRS의 가장 큰 비중이 범위 밖
- **웹 파트만 분리하면 약 5~7주 내 데모 MVP 가능**
- IT 기획자의 도메인 지식(비즈니스 로직, 우선순위 판단)은 바이브코딩에서 오히려 강점

---

## Part III. 관점 B — 개발 속도 (Time-to-MVP 최적화)

### B-1. 현재 SRS의 MVP 소요 시간 분석

SRS §10에 명시된 스프린트 추정치를 기준으로 총 개발 기간을 산출합니다.

| Feature Group | SRS 추정치 | 스프린트(2주) | 바이브코딩 고려 시 실제 예상 | 비고 |
| :--- | :--- | :--- | :--- | :--- |
| FR-01: AI 필터링 엔진 | XL (3-4 Sprints) | 6~8주 | **해당 없음** (바이브코딩 불가) | 전문 ML팀 별도 필요 |
| FR-02: 센서 모듈 | L (2-3 Sprints) | 4~6주 | **해당 없음** | 임베디드팀 별도 필요 |
| FR-03: 프라이버시 Edge | L (2-3 Sprints) | 4~6주 | **해당 없음** | Edge 팀 별도 필요 |
| FR-04: B2B 대시보드 + EMR | M (1-2 Sprints) | 2~4주 | 3~5주 ⬆️ | 초급자 디버깅 시간 추가 |
| FR-05: 웰니스 리포트 | M (1-2 Sprints) | 2~4주 | 2~3주 | AI SDK 문서 잘 정리됨 |
| FR-06: 수면 차트 | S (1 Sprint) | 2주 | 1~2주 | 바이브코딩 적합 |
| FR-07: SMS/카카오 | S (1 Sprint) | 2주 | 2~3주 ⬆️ | API 키 발급 절차 |
| FR-08: 대시보드 필터 | S (0.5-1 Sprint) | 1~2주 | 1주 | 바이브코딩 적합 |
| **NEW: PWA Guardian Portal** | M (1-2 Sprints) | 2~4주 | 3~4주 ⬆️ | Service Worker 학습 필요 |
| **NEW: AI Wellness Summary** | S (0.5-1 Sprint) | 1~2주 | 1~2주 | Vercel AI SDK |
| **인프라 셋업** | S (0.5-1 Sprint) | 1~2주 | 1~2주 | create-next-app + Vercel |

**SRS가 명시한 총 추정: 약 12~18 스프린트 (24~36주, 전체)**  
**바이브코딩 웹 파트만: 약 14~22주 (풀타임 기준)**

### B-2. 개발 속도 병목 분석

> ⚠️ 현재 SRS는 MVP 단계에서 **과도한 시간 소요를 유발하는 5가지 구조적 문제**를 내포합니다.

| # | 병목 요인 | SRS 참조 | Time-to-MVP 영향 | 권고 조치 |
| :--- | :--- | :--- | :--- | :--- |
| **B-2-1** | **11개 Route Handler 동시 관리** | §3.3, §6.1 | +3~4주 | MVP에서 필수 6~7개로 축소. Archive, Filter, Trend는 Phase 2로 분리. |
| **B-2-2** | **4종 인증 메커니즘 혼재** | §3.3 Auth 컬럼 | +2~3주 | API Key / JWT / RBAC / HMAC → **JWT 단일**로 통일. EMR HMAC은 실제 연동 시 추가. |
| **B-2-3** | **7개 Mermaid 시퀀스 다이어그램 → 코드 변환** | §3.4, §6.3 | 개발 범위 혼란 | MVP 실행 범위를 2~3개 핵심 시퀀스로 한정. 나머지는 참조용으로 유지. |
| **B-2-4** | **Prisma 7개 모델 + M:N Join Table** | §3.6, §6.2 | +1~2주 | DeadLetterEvent 제거. UserDevice Join Table은 유지하되, Facility 모델은 하드코딩으로 시작. |
| **B-2-5** | **PWA + Service Worker + Web Push 동시 구현** | §3.2, GAP-02 | +2~3주 | Phase 0에서는 일반 웹앱으로 시작. Service Worker/Web Push는 Phase 2에서 추가. |

### B-3. 권고: MVP 스코프 축소안 (Time-to-Demo: 5~7주)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MVP 스코프 축소 제안                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Phase 0 (1주):  프로젝트 셋업 + Prisma 5 모델 + Vercel 배포 확인    │
│                                                                     │
│  Phase 1 (2주):  핵심 UI 3개 페이지                                  │
│    ├─ Guardian Dashboard (일일 리포트 카드)                           │
│    ├─ B2B Dashboard (Traffic Light 그리드)                           │
│    └─ Daily Report 상세 조회                                         │
│                                                                     │
│  Phase 2 (2주):  데이터 파이프라인                                    │
│    ├─ 모의 이벤트 수신 API (Edge 시뮬레이터 대체)                     │
│    ├─ Daily Report 생성 (Cron + Server Action)                      │
│    ├─ Gemini AI 요약 생성                                            │
│    └─ 기본 로그인 (NextAuth.js)                                      │
│                                                                     │
│  Phase 3 (2주):  보조 기능                                           │
│    ├─ 오보 신고 피드백                                                │
│    ├─ Sleep Trend 차트 (Recharts)                                   │
│    └─ PWA manifest + Service Worker (기본)                          │
│                                                                     │
│  ──── Phase 4+ (전문 개발자 합류 후) ────────────────────────────     │
│    ├─ EMR Webhook (HMAC 보안)                                       │
│    ├─ Supabase Realtime 실시간 구독                                  │
│    ├─ SMS/카카오톡 폴백                                               │
│    ├─ 90일 아카이브 + Hash Chain                                     │
│    └─ Edge AI/HW/FW 통합                                            │
└─────────────────────────────────────────────────────────────────────┘
```

### B-4. SRS 내 불필요한 복잡도 — "MVP킬러" 항목

SRS에 정의되어 있지만 MVP 단계에서는 **시간만 소모하고 사용자 가치를 즉시 전달하지 못하는** 항목입니다.

| # | 항목 | SRS 참조 | 이유 | 권고 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **DeadLetterEvent 모델** | §3.6, §6.2.7 | EMR 연동 자체가 Wave 2 수준인데, 실패 이벤트 관리 테이블까지 설계하는 것은 과잉 | 제거. 로그 기반 에러 추적으로 대체 |
| 2 | **Hash Chain 무결성 증명** | REQ-FUNC-015, §3.3 #7 | 법적 증거력은 실제 소송 리스크 발생 시에 필요. MVP에서는 단순 SHA-256 해시 필드 유지로 충분 | integrityHash 필드만 유지, 체인 검증 로직 생략 |
| 3 | **PagerDuty 연동** | REQ-NF-007, §6.3.2 | 운영팀이 구성되지 않은 MVP 단계에서 유료 PagerDuty 연동은 불필요 | Slack Webhook으로 대체 (무료) |
| 4 | **Amplitude/Mixpanel 이중 옵션** | §3.1 Analytics | MVP에서는 하나의 분석 도구만 필요 | Amplitude 또는 Vercel Analytics 하나만 선택 |
| 5 | **Rate Limit (100 req/min/facility)** | §3.3 #2 | MVP <500 디바이스에서 Rate Limit 로직은 과잉 엔지니어링 | MVP에서 생략. Wave 2에서 적용 |
| 6 | **Cold Archival to Supabase Storage** | REQ-NF-017, GAP-03 | 90일 이상 데이터 아카이빙은 MVP 런칭 후 3개월 뒤에야 필요 | 런칭 후 90일 시점에 구현해도 충분 |
| 7 | **다중 zone 지원 (BEDROOM, BATHROOM, LIVING_ROOM)** | §6.2.1 locationZone | MVP에서는 단일 공간(침실) 검증만으로 충분 | zone을 1개(BEDROOM)로 고정. 확장은 Wave 2 |

---

## Part IV. 관점 C — 외부 연동 & 기술 스택 분석

### C-1. 기술 스택 총괄 평가

| 기술 | 역할 | 오픈소스 | 무료 티어 | 벤더 락인 리스크 | 바이브코딩 친화도 | 종합 판정 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Next.js 15 (App Router)** | 풀스택 프레임워크 | ✅ MIT | ✅ 셀프호스팅 가능 | 🟢 낮음 (self-host 가능) | ⭐⭐⭐⭐⭐ | ✅ 적합 |
| **Prisma ORM** | DB 추상화 | ✅ Apache 2.0 | ✅ 완전 무료 | 🟢 낮음 (SQL fallback 가능) | ⭐⭐⭐⭐ | ✅ 적합 |
| **Supabase** | PostgreSQL + Auth + Storage + Realtime | ✅ Apache 2.0 | ✅ 무료 티어 | 🟡 중간 (Realtime API 독자 규격) | ⭐⭐⭐ | ✅ 적합 (주의 필요) |
| **Vercel** | 배포 + Edge Functions + Cron | ❌ 프로프라이어터리 | 🔺 Hobby 제한 | 🔴 높음 (Cron, Edge 의존) | ⭐⭐⭐⭐⭐ | ⚠️ 적합 (락인 인지 필요) |
| **shadcn/ui + Tailwind CSS** | UI 컴포넌트 | ✅ MIT | ✅ 완전 무료 | 🟢 없음 (코드 소유) | ⭐⭐⭐⭐⭐ | ✅ 최적 |
| **NextAuth.js (Auth.js)** | 인증 | ✅ ISC | ✅ 완전 무료 | 🟢 낮음 | ⭐⭐⭐ | ✅ 적합 |
| **Vercel AI SDK** | LLM 오케스트레이션 | ✅ Apache 2.0 | ✅ 완전 무료 | 🟡 중간 (Vercel 엔진 최적화) | ⭐⭐⭐⭐⭐ | ✅ 적합 |
| **Google Gemini API** | LLM 모델 | ❌ 프로프라이어터리 | ✅ 무료 할당량 | 🟡 중간 (AI_MODEL env 스왑 가능) | ⭐⭐⭐⭐ | ✅ 적합 |
| **FCM (Firebase)** | 푸시 알림 | ❌ 프로프라이어터리 | ✅ 무료 | 🟡 중간 (Google 의존) | ⭐⭐⭐ | ✅ 적합 |
| **Web Push API (VAPID)** | 브라우저 푸시 | ✅ W3C 표준 | ✅ 완전 무료 | 🟢 없음 (표준) | ⭐⭐ | 🔺 복잡 |

### C-2. 벤더 락인 리스크 매트릭스

```
┌───────────────────────────────────────────────────────────────────┐
│               벤더 락인 리스크 vs. 대체 난이도 매트릭스              │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  높음 ┤                          🔴 Vercel Cron                   │
│       │                          🔴 Vercel Edge Runtime           │
│ 락인  │                                                           │
│ 리스크│         🟡 Supabase Realtime                              │
│       │         🟡 Gemini API                                     │
│       │         🟡 FCM                                            │
│  낮음 ┤  🟢 Next.js   🟢 Prisma   🟢 shadcn/ui                  │
│       │  🟢 Tailwind  🟢 NextAuth                                │
│       │                                                           │
│       └──┬─────────────┬──────────────┬─────────────┬───────      │
│          쉬움          보통           어려움          매우 어려움    │
│                      (대체 난이도 →)                               │
└───────────────────────────────────────────────────────────────────┘
```

### C-3. 외부 연동 복잡도 상세 분석

#### C-3-1. Vercel 플랫폼 의존성 ⚠️

| 의존 서비스 | SRS 참조 | 락인 수준 | 대안 | 이탈 비용 |
| :--- | :--- | :--- | :--- | :--- |
| **Vercel Cron Jobs** | §3.1, CRON | 🔴 높음 | node-cron + 자체 서버, GitHub Actions 스케줄 | 코드 수정 필요 (Cron Route → 스케줄러) |
| **Vercel Edge Runtime** | REQ-NF-001 | 🔴 높음 | Cloudflare Workers, Deno Deploy | 런타임 API 차이로 부분 재작성 필요 |
| **Vercel Analytics** | REQ-NF-001 | 🟡 중간 | Amplitude, PostHog (오픈소스) | SDK 교체만으로 가능 |
| **Git Push 자동 배포** | CON-12 | 🟢 낮음 | Netlify, Railway, Fly.io, Docker + 자체 서버 | Next.js는 어디서든 배포 가능 |

> **판정:** MVP 단계에서 Vercel 의존은 **허용 가능**합니다. 개발 속도 이점이 락인 리스크를 상회합니다. 다만 **Wave 2에서 자체 호스팅(Docker + VPS) 전환 가능성을 SRS에 명시**해두어야 합니다.

#### C-3-2. Supabase 의존성 ⭕

| 의존 서비스 | SRS 참조 | 락인 수준 | 대안 |
| :--- | :--- | :--- | :--- |
| **PostgreSQL** | CON-08 | 🟢 낮음 | 표준 PostgreSQL이므로 AWS RDS, Neon, PlanetScale 등으로 이전 가능 |
| **Supabase Realtime** | FR-04, GAP-01 | 🟡 중간 | Socket.IO, Pusher, Ably. 독자 API이므로 코드 수정 필요 |
| **Supabase Storage** | REQ-NF-017 | 🟡 중간 | S3 호환 API이므로 AWS S3, Cloudflare R2로 쉽게 이전 가능 |
| **Supabase Auth** | 미사용 (NextAuth.js 대체) | 🟢 없음 | 이미 NextAuth.js 사용 중 |

> **판정:** Supabase는 **오픈소스(셀프호스팅 가능)**이므로 락인 리스크가 관리 가능합니다. Realtime만 독자 규격이며, MVP 이후 필요 시 Socket.IO로 대체 가능합니다.

#### C-3-3. Gemini API 의존성 ⭕

| 항목 | 분석 |
| :--- | :--- |
| **락인 리스크** | 🟡 중간 — 단, SRS §7.4에서 `AI_MODEL` 환경변수로 모델 스왑을 설계했으므로 **잘 관리됨** |
| **대안 모델** | Claude (Anthropic), GPT-4o (OpenAI), Llama 3 (Meta, 오픈소스). Vercel AI SDK가 다중 프로바이더를 지원하므로 코드 변경 최소 |
| **비용 리스크** | 하루 1회/디바이스 생성이므로 MVP <500 디바이스 기준 월 $5~$15 수준 |
| **권고** | ✅ 현재 설계가 적절. ENV 기반 모델 스왑은 바이브코딩 MVP에서 특히 유용 (비용 문제시 즉시 Flash 모델로 전환 가능) |

#### C-3-4. EMR Webhook 연동 ⚠️ 과잉

| 항목 | 분석 |
| :--- | :--- |
| **현재 설계** | HMAC-SHA256 서명 + Exponential Backoff 3회 재시도 + DeadLetterEvent 모델 + Rate Limit 100 req/min |
| **MVP 필요 수준** | EMR 벤더(Carefor)와의 기술 파트너십이 **아직 체결되지 않은 상태**(DEP-01). MVP에서 실제 연동이 일어날 가능성 낮음. |
| **과잉 판정** | SRS의 EMR 명세는 **프로덕션급**. MVP에서는 "EMR 연동 시뮬레이터"(Webhook URL로 JSON 전송만 확인)로 충분. |
| **권고** | HMAC-SHA256 + DeadLetterEvent + Rate Limit → MVP에서 **제거**. 단순 HTTP POST 전송 확인용 모의 Webhook으로 대체. |

#### C-3-5. FCM + Web Push 이중 구조 ⚠️ 조정 필요

| 항목 | 분석 |
| :--- | :--- |
| **현재 설계** | FCM + Web Push API 동시 사용. Emergency = 즉시, Report = 07:30 스케줄. |
| **복잡도** | FCM 서버 키 + VAPID 키 + Service Worker + Push 구독 관리 → 초급자에게 **4개 개념을 동시에 학습**하도록 요구 |
| **권고** | MVP Phase 1~2에서는 **Web Push만** 사용 (VAPID). FCM은 Phase 3 이후 추가. 또는 **FCM만** 사용하고 Web Push는 Wave 2로 이관. 둘 다 동시에 구현하면 디버깅 복잡도가 2배. |

### C-4. 기술 스택 종합 판정

| 판정 항목 | 결과 |
| :--- | :--- |
| **전체 스택의 바이브코딩 적합성** | ⭕ **우수** — Next.js + Prisma + shadcn/ui는 AI 코딩 어시스턴트가 가장 잘 지원하는 스택 |
| **오픈소스 비율** | **8/10 (80%)** — Next.js, Prisma, shadcn/ui, Tailwind, NextAuth.js, Supabase, Vercel AI SDK, Web Push API |
| **프로프라이어터리 서비스** | Vercel (배포), Gemini API (LLM), FCM (푸시) — 모두 대안 존재 |
| **MVP 필수 서비스 수** | SRS 기준 10개 → **권고: 6~7개로 축소** |
| **핵심 리스크** | Vercel Cron/Edge 락인 (but MVP에서는 허용 가능) |

---

## Part V. 관점 D — 운영 소요 비용 분석

### D-1. 서비스별 월간 비용 산정 (MVP <500 디바이스 기준)

| 서비스 | 플랜 | 월 비용 (USD) | 포함 내역 | 초과 시 비용 | SRS 참조 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Vercel** | Pro | **$20/월** | 무제한 배포, Cron Jobs, Edge Functions, Analytics, 100GB 대역폭, Serverless 1000 GB-hours | 대역폭 $0.15/GB, Serverless $0.18/GB-hr | CON-12 |
| **Supabase** | Free → Pro | **$0~$25/월** | Free: 500MB DB, 1GB Storage, 50K MAU. Pro: 8GB DB, 100GB Storage, 무제한 MAU, Realtime 200동시연결 | Storage $0.021/GB, DB $0.125/GB | CON-08 |
| **Google Gemini API** | Pay-as-you-go | **$5~$15/월** | gemini-1.5-pro: 입력 $1.25/1M tokens, 출력 $5/1M tokens. MVP 500디바이스 × 1회/일 × 30일 = 15,000 호출 (프롬프트 ~300 tokens/call) | Flash 모델로 스왑 시 ~70% 절감 | CON-10, CON-11 |
| **FCM** | 무료 | **$0** | 무제한 알림 전송 | 없음 | DEP-03 |
| **Amplitude** | Free Starter | **$0** | 10M 이벤트/월 | $0.00045/이벤트 | §3.1 |
| **GitHub** | Free | **$0** | 무제한 퍼블릭 리포, 무제한 Actions (퍼블릭) | Private: $4/인/월 | - |
| **도메인 (선택)** | .com | **$1/월** (연 $12) | 커스텀 도메인 | - | - |
| **PagerDuty** (SRS 명시) | Free / Pro | **$0~$21/월** | Free: 5 사용자까지. Pro: $21/사용자/월 | - | REQ-NF-007 |

### D-2. 시나리오별 월간 총 비용

| 시나리오 | 디바이스 수 | 월 비용 | 연간 비용 | 비고 |
| :--- | :--- | :--- | :--- | :--- |
| **최소 MVP (개발/테스트)** | <10 | **~$20** | ~$240 | Vercel Pro만. Supabase Free + Gemini 무료 할당량 내 |
| **초기 Beta (폐쇄 베타)** | 50~100 | **$45~$60** | $540~$720 | Vercel Pro + Supabase Free(또는 Pro 전환) + Gemini Flash |
| **확장 Beta / Wave 1** | 200~500 | **$70~$120** | $840~$1,440 | Vercel Pro + Supabase Pro + Gemini Pro + Amplitude Free |
| **Wave 2 진입 (5K 목표)** | 1,000~5,000 | **$200~$500+** | $2,400~$6,000+ | Vercel Enterprise 검토 필요. Supabase Scale 전환. Gemini 배치 최적화 필수. |

### D-3. SRS 비용 목표 대비 검증

SRS REQ-NF-012에서 **≤ 500 KRW/Unit/Month** (약 $0.37/디바이스/월)을 비용 목표로 명시합니다.

| 시나리오 | 총 비용 | 디바이스 수 | 단위 비용 | SRS 목표 충족 |
| :--- | :--- | :--- | :--- | :--- |
| Beta 50대 | $50/월 | 50 | $1.00/대 (₩1,350) | ❌ 미달 (초기 고정 비용 분산) |
| Beta 200대 | $80/월 | 200 | $0.40/대 (₩540) | ❌ 근접하나 미달 |
| MVP 500대 | $120/월 | 500 | $0.24/대 (₩324) | ✅ **충족** |
| Wave 2 5,000대 | $400/월 | 5,000 | $0.08/대 (₩108) | ✅ **충족 (규모의 경제)** |

> **결론:** SRS의 ≤500원/대/월 목표는 **500대 이상에서만 달성 가능**합니다. Beta 단계(<200대)에서는 고정 비용(Vercel Pro $20, Supabase Pro $25)의 비중이 커서 단위 비용이 높습니다. 이는 구조적인 문제가 아니라 초기 규모의 한계이므로 SRS 수정은 불필요하나, **비용 목표 달성 시점을 "500대 이상" 조건부로 명시**하는 것을 권고합니다.

### D-4. 비용 최적화 권고

| # | 최적화 항목 | 절감 예상 | 실행 난이도 |
| :--- | :--- | :--- | :--- |
| 1 | **Gemini 1.5 Flash** 모델 사용 (개발/Beta) | AI 비용 ~70% 절감 | ⭐ (env 변수 변경) |
| 2 | **PagerDuty → Slack Webhook 대체** | $0~$21/월 절감 | ⭐ (무료 대안) |
| 3 | **Amplitude → Vercel Analytics 통합** | $0 (이미 Vercel Pro에 포함) | ⭐⭐ |
| 4 | **Supabase Free 티어 유지** (DB 500MB 이내) | $25/월 절감 | ⭐ (초기 데이터 적음) |
| 5 | **AI 서머리 캐싱 철저히 적용** (하루 1회/디바이스) | API 호출 50%+ 절감 | ⭐⭐ (이미 SRS에 포함) |
| 6 | **Cold Archival은 MVP 런칭 90일 후 구현** | 초기 Storage 비용 $0 | ⭐ (지연 실행) |

### D-5. 비용 관점 숨겨진 리스크

| # | 리스크 | 확률 | 영향 | 대응 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **Vercel Serverless 사용량 초과** | 2/5 | 3/5 | MVP <500 디바이스에서는 안전. 대시보드에서 사용량 모니터링 필수. |
| 2 | **Gemini API 요금 폭증** (프롬프트 최적화 실패) | 2/5 | 2/5 | 프롬프트를 100 토큰 이내로 제한. Flash 모델 사용. |
| 3 | **Supabase Realtime 연결 제한 초과** | 2/5 | 3/5 | Free 기본 200 동시연결. Pro 업그레이드 시 500~무제한. |
| 4 | **SMS/카카오톡 API 비용** (FR-07 실행 시) | 3/5 | 3/5 | SMS: 건당 ₩20~50. 500디바이스 × 30일 × 1건 = ₩300K~750K/월 추가. **MVP에서는 Web Push만** 사용 권고. |

---

## Part VI. 3개 관점 통합 — 교차 분석

### VI-1. "할 수 있는가" × "빨리 할 수 있는가" × "저렴하게 할 수 있는가"

| 기능/요소 | 구현 가능성 (A) | 개발 속도 (B) | 비용 효율 (D) | 종합 MVP 적합도 |
| :--- | :--- | :--- | :--- | :--- |
| **Next.js + Prisma 셋업** | ⭕ | ⭕ 1~2일 | ⭕ $0 | ✅ 최적 |
| **shadcn/ui 기반 Dashboard UI** | ⭕ | ⭕ 1~2주 | ⭕ $0 | ✅ 최적 |
| **Gemini AI 웰니스 요약** | ⭕ | ⭕ 2~3일 | ⭕ $5~15/월 | ✅ 최적 — **MVP 킬러 피처** |
| **Vercel 배포 + Cron** | ⭕ | ⭕ 1일 | ⭕ $20/월 | ✅ 최적 |
| **NextAuth.js 기본 로그인** | 🔺 | ⚠️ 2~3일 | ⭕ $0 | ✅ 무난 |
| **Recharts 수면 차트** | ⭕ | ⭕ 2~3일 | ⭕ $0 | ✅ 최적 |
| **Supabase Realtime 구독** | 🔺 | ⚠️ 1~2주 | 🔺 $0~25/월 | ⚠️ Phase 2로 이관 권고 |
| **PWA Service Worker + Web Push** | 🔺 | ❌ 2~3주 | ⭕ $0 | ⚠️ Phase 3으로 이관 권고 |
| **EMR Webhook (HMAC)** | 🔺 | ❌ 2~3주 | ⭕ $0 | ❌ Wave 2로 이관 |
| **SMS/카카오톡 폴백** | 🔺 | ⚠️ 2~3주 | ❌ ₩300K+/월 | ❌ Wave 2로 이관 (비용) |
| **90일 아카이브 + Hash Chain** | 🔺 | ❌ 2~3주 | ⭕ $0 | ❌ 런칭 후 90일 시점 구현 |
| **PagerDuty 연동** | ⭕ | ⚠️ 3~5일 | ❌ $21/월 | ❌ Slack 대체 |
| **Edge AI / HW / FW** | ❌ | ❌ | ❌ (별도 비용) | ❌ 별도 팀 |

### VI-2. 최종 MVP 스코프 재정의 제안

위 교차 분석에 기반하여, **3개 관점 모두에서 ✅인 항목만** MVP 1차 스코프로 확정합니다.

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  ✅ MVP Core Scope (5~7주)                                           │
│  ─────────────────────────                                           │
│  • Next.js + Prisma 5 모델 + Vercel 배포                             │
│  • B2C Guardian Portal (웹앱, PWA는 Phase 3)                         │
│  • B2B Dashboard UI (Traffic Light, 정적 데이터)                      │
│  • Gemini AI 웰니스 요약 (Vercel AI SDK)                              │
│  • Daily Report 생성 (Vercel Cron + Server Action)                   │
│  • 모의 데이터 수신 API (Edge 시뮬레이터 대체)                        │
│  • NextAuth.js 기본 로그인 (JWT 단일)                                 │
│  • Sleep Trend 차트 (Recharts)                                       │
│  • 오보 신고 피드백 (Prisma update)                                   │
│                                                                      │
│  ⚠️ MVP Phase 2 (전문가 리뷰 후 추가, +3~4주)                        │
│  ─────────────────────────────────────                                │
│  • Supabase Realtime 대시보드 실시간 업데이트                        │
│  • PWA manifest + Service Worker + Web Push                          │
│  • Amplitude 이벤트 트래킹                                            │
│  • NextAuth.js RBAC (관리자/보호자 역할 분리)                         │
│                                                                      │
│  ❌ Wave 2 이관 (전문 개발자 + 별도 예산)                              │
│  ─────────────────────────────────────                                │
│  • EMR Webhook (HMAC-SHA256 + DeadLetterEvent)                       │
│  • SMS/카카오톡 폴백 (운영 비용 + 사업자 등록)                        │
│  • 90일 아카이브 + Hash Chain 무결성                                  │
│  • PagerDuty 연동                                                    │
│  • Cold Archival (Supabase Storage)                                  │
│  • Edge AI / HW / FW 통합                                            │
│  • iOS/Android 네이티브 앱                                            │
│  • Rate Limiting                                                     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Part VII. SRS 문서 수정 권고 사항 종합

### VII-1. SRS 구조 수정 권고

| # | 권고 사항 | 영향 영역 | 우선순위 |
| :--- | :--- | :--- | :--- |
| 1 | **"Web-Only MVP"와 "Edge/HW" 영역을 명시적으로 분리** | §1.2 Scope | 🔴 필수 |
| 2 | **Route Handler를 MVP Core(6~7개)와 Phase 2+(4~5개)로 명확히 구분** | §3.3, §6.1 | 🔴 필수 |
| 3 | **인증 메커니즘을 MVP단계 JWT 단일로 단순화, HMAC/RBAC는 Wave 2로 표기** | §3.3 Auth, §4.2 | 🔴 필수 |
| 4 | **모의 데이터 생성기(Mock Data Generator / Seed Script) 사양 추가** | §6.2 또는 새 섹션 | 🔴 필수 |
| 5 | **UI 와이어프레임 또는 레퍼런스 디자인 링크 추가** | 새 섹션 | 🟡 권고 |
| 6 | **비용 목표 ≤500원/대/월의 달성 조건 "500대 이상" 명시** | REQ-NF-012 | 🟡 권고 |
| 7 | **DeadLetterEvent 모델을 Wave 2로 이관 표기** | §3.6, §6.2.7 | 🟡 권고 |
| 8 | **PagerDuty를 Slack Webhook으로 대체 가능 표기** | REQ-NF-007, §6.3.2 | 🟢 선택 |
| 9 | **FCM/Web Push 이중 구조를 MVP 단일(Web Push만 또는 FCM만)로 단순화 표기** | §3.3 #3 | 🟡 권고 |
| 10 | **Sprint Estimation(§10)에 "바이브코딩 기준 추정치" 열 추가** | §10 | 🟢 선택 |

### VII-2. 새로 추가해야 할 섹션

| 섹션명 | 내용 | 이유 |
| :--- | :--- | :--- |
| **Mock Data Specification** | Edge 시뮬레이터 대체를 위한 모의 이벤트 데이터 구조, Seed Script 사양 | 센서 없이 웹 파트를 개발/테스트하기 위한 필수 조건 |
| **MVP Phase 정의** | Phase 0~3 vs. Wave 2의 명확한 범위 정의 | 바이브코딩 실행자가 "지금 뭘 만들어야 하는지" 즉시 판단 가능 |
| **바이브코딩 실행 가이드** (선택) | 핵심 프롬프트 템플릿, AI 코딩 어시스턴트에 전달할 컨텍스트 구조 | IT 기획자가 AI에게 SRS를 효과적으로 전달하기 위한 가이드 |

---

## Part VIII. 최종 결론

### ⚖️ 종합 판정표

| 검토 관점 | 현재 SRS 판정 | 조정 후 예상 판정 | 핵심 조정 사항 |
| :--- | :--- | :--- | :--- |
| **A. 난이도/구현 가능성** | ⚠️ 웹 파트 50~60% 가능 | ✅ 웹 파트 80%+ 가능 | Edge/HW 분리, 인증 단순화, 모의 데이터 추가 |
| **B. 개발 속도** | ❌ 14~22주 (웹 파트) | ✅ 5~7주 (Core MVP) | 11→6~7 Route Handler, 7→5 모델, Phase 분리 |
| **C. 외부 연동/기술 스택** | ⚠️ 10개 서비스 동시 연동 | ✅ 6~7개로 축소 | EMR/PagerDuty/SMS 이관, 푸시 단일화 |
| **D. 운영 비용** | ⭕ $70~120/월 (500대) | ⭕ $40~70/월 (최적화) | Gemini Flash, PagerDuty→Slack, Supabase Free |

### 🎯 최우선 실행 항목 (Top 5)

| 순위 | 항목 | 이유 |
| :--- | :--- | :--- |
| **1** | SRS에서 Web-Only MVP 범위를 명확히 분리 | 모든 후속 작업의 전제 조건 |
| **2** | 모의 데이터 사양(Mock Data Spec) 정의 | 센서 없이 개발을 시작하기 위한 필수 조건 |
| **3** | 인증을 JWT 단일로 단순화 | 개발 속도를 2~3주 단축하는 가장 효과적인 조치 |
| **4** | Route Handler를 Core 6~7개로 축소 | 관리 복잡도 50% 감소 |
| **5** | Gemini Flash 모델을 기본값으로 설정 | 비용 70% 절감 + 응답 속도 향상 |

---

**— 검토 완료 —**
