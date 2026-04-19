# SRS v01(ENG_OPUS_MVP) — 개발 난이도 및 구현 가능성 검토 보고서

**문서 ID:** REVIEW-SRS-DIFFICULTY-001  
**검토 대상:** `SRS_v01(ENG_OPUS_MVP).md` (Revision 2.0, 2026-04-19)  
**검토 일자:** 2026-04-19  
**검토 관점:**

| 항목 | 프로필 |
| :--- | :--- |
| 개발 경험 수준 | SW 개발 학습 3개월 (초급자) |
| 직무 배경 | IT 기획 3년차 (요구사항 정의, 문서화, 이해관계자 소통 역량 보유) |
| 개발 방식 | **완전 바이브코딩** (AI 코딩 어시스턴트 기반, 직접 코드 작성 최소화) |

---

## 1. 검토 요약 (Executive Summary)

> **종합 판정: 현재 SRS는 바이브코딩 MVP에 과(過)설계 상태이며, 범위 축소 없이는 구현 실패 위험이 높습니다.**

| 평가 항목 | 결과 |
| :--- | :--- |
| 전체 기능 수 | 23개 Functional Requirements + 20개 Non-Functional Requirements |
| 바이브코딩으로 구현 가능한 FR | **약 10~12개** (전체의 ~50%) |
| 전문 개발자 필수 영역 | **Edge AI (FR-01), HW/FW (FR-02, FR-03), EMR HMAC 보안 (FR-04 일부)** |
| 예상 총 개발 기간 (바이브코딩 기준) | 웹 파트만 4~6개월 (풀타임 기준) |
| SRS 문서 자체의 품질 | ISO/IEC/IEEE 29148 준수, 기술 사양 매우 상세 — **그러나 바이브코딩 실행자에게는 오히려 과도하게 복잡** |

---

## 2. 대상 프로필 역량 분석

### 2.1 강점 (IT 기획 3년차의 이점)

| 강점 | SRS 내 활용 가능 영역 |
| :--- | :--- |
| **요구사항 이해 능력** | PRD→SRS 트레이서빌리티 매트릭스(§5) 이해 및 검증 가능 |
| **비즈니스 로직 설계** | Triage 우선순위 로직, 리포트 생성 흐름, 알림 분기 조건 정의 |
| **이해관계자 소통** | 페르소나 기반 UC 다이어그램(§3.5), B2B/B2C 분리 구조 설계 의도 파악 |
| **문서 구조화** | MoSCoW 기반 우선순위 관리, 스프린트 추정치(§10) 활용한 로드맵 조정 |

### 2.2 약점 (3개월 초급 개발자의 한계)

| 약점 | SRS 내 해당 영역 | 위험도 |
| :--- | :--- | :--- |
| TypeScript/React 기초 부족 | Next.js App Router 전체 구조 (page.tsx, layout.tsx, Server Components vs Client Components) | 🔴 높음 |
| 비동기 프로그래밍 이해 부족 | Prisma async/await, Supabase Realtime 구독, Promise 체이닝 | 🔴 높음 |
| 인증/보안 구현 경험 부재 | NextAuth.js + JWT + RBAC + HMAC-SHA256 + VAPID Key 관리 | 🔴 높음 |
| 데이터베이스 설계 미숙 | 7개 모델 관계 (1:N, M:N Join Table), 인덱싱, 마이그레이션 | 🟡 중간 |
| CI/CD 파이프라인 경험 부재 | Vercel 배포, Cron Job 설정, vercel.json 구성 | 🟡 중간 |
| Edge/IoT 개발 경험 없음 | UWB 센서, Edge AI Validator, 심층학습 모델, OTA | 🔴 치명적 |

---

## 3. 기능 영역별 난이도 분석

> **난이도 등급 기준:**
> - ⭐ (1): 바이브코딩으로 즉시 구현 가능. 프롬프트 1~3회면 충분.
> - ⭐⭐ (2): 바이브코딩으로 구현 가능하나, 다소 시행착오 예상. 프롬프트 5~10회.
> - ⭐⭐⭐ (3): 바이브코딩 가능하나, 개념 학습 + 디버깅에 상당 시간 소요. 1~2주.
> - ⭐⭐⭐⭐ (4): 바이브코딩만으로는 어려움. 기초 지식 학습 + 전문가 리뷰 필요.
> - ⭐⭐⭐⭐⭐ (5): 바이브코딩 불가. 전문 개발자 필수.

---

### 3.1 FR-01: Zero False Alarm AI Filtering Engine (Must, XL) — Edge AI

| 항목 | 분석 |
| :--- | :--- |
| **난이도** | ⭐⭐⭐⭐⭐ (5/5) |
| **바이브코딩 가능성** | ❌ 불가 |
| **핵심 기술 요소** | Deep Learning 추론 모델, UWB 레이더 신호 처리, Edge 임베디드 환경, 반려동물 구분(≥99% 정확도), confidence_score 기반 threshold 최적화 |
| **왜 불가능한가** | ① 딥러닝 모델 학습에는 실제 UWB 센서 데이터셋이 필요(합성 데이터로 부족). ② 임베디드 환경(C/C++/Rust)에서의 추론 최적화는 AI 코딩 어시스턴트가 커버하지 못하는 하드웨어-소프트웨어 인터페이스 영역. ③ REQ-FUNC-003의 "반려동물 ≥99% 정확도"는 수개월간의 데이터 수집과 모델 튜닝이 필요한 ML Engineering 과제. |

> ⚠️ **CAUTION:** FR-01은 제품의 핵심 가치 제안(Zero False Alarm)을 담당하는 영역입니다. 이 부분은 ML/Edge 전문 개발자의 참여가 절대적으로 필요합니다. 바이브코딩으로 대체할 수 없습니다.

---

### 3.2 FR-02: Zero-Friction Contactless Sensor Module (Must, L) — HW/FW

| 항목 | 분석 |
| :--- | :--- |
| **난이도** | ⭐⭐⭐⭐⭐ (5/5) |
| **바이브코딩 가능성** | ❌ 불가 |
| **핵심 기술 요소** | UWB 센서 하드웨어, 펌웨어 개발, 자동 캘리브레이션, Wi-Fi 하트비트, OTA 업데이트 |
| **왜 불가능한가** | 물리적 하드웨어 개발 및 펌웨어 프로그래밍은 바이브코딩의 범위 밖. NXP/Infineon 칩셋 제어, 레이더 신호 수집, 에지 추론 파이프라인 모두 임베디드 엔지니어 영역. |

> ⚠️ **WARNING:** 하트비트 모니터링 API(Route Handler)는 바이브코딩으로 구현 가능하나, 센서 측 펌웨어는 반드시 별도 팀이 개발해야 합니다.

---

### 3.3 FR-03: Privacy-Preserving Non-Video Tracking (Must, L) — Edge

| 항목 | 분석 |
| :--- | :--- |
| **난이도** | ⭐⭐⭐⭐⭐ (5/5) |
| **바이브코딩 가능성** | ❌ 불가 |
| **핵심 기술 요소** | 레이더 원시 데이터 → 수치 메타데이터 변환(비식별화) 로직, Edge 로컬 CPU 처리 |
| **왜 불가능한가** | FR-01, FR-02와 동일한 임베디드/Edge 영역. PII 제거 로직의 코드 구현 자체는 가능하지만, UWB 신호 처리 파이프라인과 긴밀하게 결합되어 있음. |

---

### 3.4 FR-04: B2B Multi-bed Dashboard + EMR Webhook (Must, M) — 🟢 부분적 구현 가능

| 세부 요소 | 난이도 | 바이브코딩 가능성 | 비고 |
| :--- | :--- | :--- | :--- |
| **Traffic Light UI (shadcn/ui Badge)** | ⭐⭐ | ⭕ 가능 | shadcn/ui 컴포넌트 조합은 바이브코딩에 매우 적합. 프롬프트: "Create a dashboard with red/yellow/green status cards using shadcn/ui" |
| **Supabase Realtime 구독** | ⭐⭐⭐ | 🔺 조건부 가능 | Supabase 클라이언트 SDK 사용법을 AI가 잘 알고 있지만, `useEffect` 내 구독/해제 라이프사이클 관리에서 버그 발생 가능 |
| **Triage 우선순위 정렬** | ⭐⭐ | ⭕ 가능 | 단순 sort 로직이므로 함수로 분리하면 바이브코딩 용이 |
| **EMR Webhook (HMAC-SHA256)** | ⭐⭐⭐⭐ | 🔺 위험 | HMAC-SHA256 서명 생성/검증 로직은 보안에 민감. 코드 자체는 AI가 생성하지만, 검증 실패 시 EMR 연동 전체가 깨짐. 전문가 리뷰 필수 |
| **Exponential Backoff Retry** | ⭐⭐⭐ | 🔺 조건부 가능 | 패턴 자체는 잘 알려져 있으나, Route Handler 내 비동기 처리와 결합 시 복잡도 증가 |
| **DeadLetterEvent 생성** | ⭐⭐ | ⭕ 가능 | Prisma create 호출이므로 단순 |
| **90일 아카이브 + Hash Chain** | ⭐⭐⭐⭐ | 🔺 위험 | SHA-256 해시 체인 무결성 증명은 구현 자체보다 "올바른 구현" 검증이 어려움 |

> **FR-04 종합 판정:** 🔺 부분적 구현 가능. UI 부분은 바이브코딩 적합. EMR/보안 로직은 전문가 리뷰 필요.

---

### 3.5 FR-05: B2C Daily Wellness Notification Pipeline (Should, M) — 🟢 대부분 구현 가능

| 세부 요소 | 난이도 | 바이브코딩 가능성 | 비고 |
| :--- | :--- | :--- | :--- |
| **Vercel Cron → Server Action 호출** | ⭐⭐ | ⭕ 가능 | `vercel.json`의 cron 설정 + Route Handler 연결은 AI에게 명확한 패턴 |
| **Prisma 데이터 집계 (sleep score, bathroom count)** | ⭐⭐ | ⭕ 가능 | `findMany` + `aggregate` 쿼리 패턴 |
| **이상치 탐지 (bathroom dwell > avg +50%)** | ⭐⭐ | ⭕ 가능 | 단순 통계 비교 로직 |
| **Gemini AI 요약 생성 (Vercel AI SDK)** | ⭐⭐ | ⭕ 가능 | `generateText()` 호출은 매우 직관적이며 Vercel AI SDK 문서가 잘 정리됨. 바이브코딩에 가장 적합한 영역 |
| **FCM/Web Push 발송** | ⭐⭐⭐ | 🔺 조건부 가능 | FCM 서버 키 설정, 토큰 관리, Service Worker 등록은 처음에 혼란 가능 |
| **"INSUFFICIENT_DATA" 상태 처리** | ⭐ | ⭕ 가능 | 단순 분기 조건 |

> **FR-05 종합 판정:** ⭕ 대부분 구현 가능. **바이브코딩 MVP의 핵심 데모 기능으로 가장 적합.**

---

### 3.6 FR-06: Sleep Trend Charts (Could, S)

| 항목 | 분석 |
| :--- | :--- |
| **난이도** | ⭐⭐ (2/5) |
| **바이브코딩 가능성** | ⭕ 가능 |
| **비고** | Recharts/Chart.js + Next.js 페이지 렌더링은 바이브코딩 프롬프트로 충분. "Create a line chart showing sleep score trends over 7 days using Recharts" 수준. |

---

### 3.7 FR-07: SMS/KakaoTalk Fallback (Should, S)

| 항목 | 분석 |
| :--- | :--- |
| **난이도** | ⭐⭐⭐ (3/5) |
| **바이브코딩 가능성** | 🔺 조건부 가능 |
| **비고** | SMS API(NCP, Twilio 등)와 KakaoTalk 알림톡 API는 각각 별도의 인증 키와 템플릿 등록이 필요. 코드 구현은 AI가 가능하나, API 키 발급 및 사업자 등록 등 운영 절차가 선행되어야 함. |

---

### 3.8 FR-08: Configurable Dashboard (Could, S)

| 항목 | 분석 |
| :--- | :--- |
| **난이도** | ⭐⭐ (2/5) |
| **바이브코딩 가능성** | ⭕ 가능 |
| **비고** | shadcn/ui DataTable + 필터 컴포넌트 + Prisma 저장. UI 프레임워크에 의존하는 작업이라 바이브코딩 적합. |

---

## 4. 인프라 및 횡단 관심사(Cross-Cutting Concerns) 난이도

| 영역 | SRS 참조 | 난이도 | 바이브코딩 가능성 | 상세 분석 |
| :--- | :--- | :--- | :--- | :--- |
| **Next.js 프로젝트 초기 세팅** | §8 Project Structure | ⭐⭐ | ⭕ | `npx create-next-app` + shadcn/ui init. AI 프롬프트로 5분 내 완성 가능. |
| **Prisma 스키마 정의 (7개 모델)** | §3.6, §6.2 | ⭐⭐⭐ | 🔺 | 모델 정의 자체는 가능하나, 관계 설정(1:N, M:N Join Table), `@@index`, `@unique` 등의 디테일에서 오류 가능. SRS의 ERD를 그대로 프롬프트에 붙이면 AI가 잘 생성. |
| **NextAuth.js 인증 (JWT + RBAC)** | §6.1, REQ-NF-011 | ⭐⭐⭐⭐ | 🔺 위험 | NextAuth.js v5(Auth.js) 설정은 잘못 구성하면 인증 우회 허점이 생김. RBAC 미들웨어 구현은 세부적인 이해 필요. |
| **Supabase 연동 (DB + Realtime + Storage)** | §3.1, CON-08 | ⭐⭐⭐ | 🔺 | Supabase 프로젝트 생성 및 기본 연동은 문서가 잘 되어 있으나, Realtime 구독 + Prisma 병행 사용 시 이중 클라이언트 관리 이슈 발생 가능. |
| **Vercel 배포 + Cron Job** | §3.1 CRON, CON-12 | ⭐⭐ | ⭕ | Git Push → 자동 배포는 가장 단순한 CI/CD. Cron은 `vercel.json`에 schedule 한 줄 추가. |
| **PWA 설정 (Service Worker + manifest.json)** | §3.2, GAP-02 | ⭐⭐⭐ | 🔺 | Service Worker 라이프사이클, Web Push 구독 관리는 복잡. `next-pwa` 라이브러리 사용 시 단순화 가능하나, iOS Safari 호환성 테스트 필요. |
| **환경 변수 관리 (12개)** | §9 | ⭐⭐ | ⭕ | `.env.local` 파일 관리는 단순. 다만 Vercel 대시보드 환경 변수 설정과의 동기화 주의 필요. |
| **TLS 1.3 보안** | REQ-NF-008 | ⭐ | ⭕ | Vercel이 기본 제공. 개발자가 별도 구현할 사안 아님. |
| **Amplitude/Mixpanel 연동** | §3.1 Analytics | ⭐⭐ | ⭕ | SDK import + event tracking 호출은 매우 단순. |

---

## 5. 종합 난이도 매트릭스

```
┌──────────────────────────────────────────────────────────────────┐
│           바이브코딩 구현 가능성 vs. 비즈니스 우선순위 매트릭스        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Must ┤  ❌ FR-01 (AI Engine)    ⚠️ FR-04 (Dashboard+EMR)       │
│       │  ❌ FR-02 (HW/Sensor)                                    │
│       │  ❌ FR-03 (Privacy Edge)                                 │
│       │                                                          │
│ Should┤  ✅ FR-05 (Wellness Report + AI) ⚠️ FR-07 (SMS/Kakao)   │
│       │                                                          │
│ Could ┤  ✅ FR-06 (Charts)      ✅ FR-08 (Filters)              │
│       │                                                          │
│       └──┬─────────────┬─────────────┬──────────────┬──────────  │
│          ✅ 쉬움       ⚠️ 보통      🔺 어려움       ❌ 불가       │
│     (바이브코딩 난이도 →)                                         │
└──────────────────────────────────────────────────────────────────┘
```

> ⚠️ **핵심 문제:** Must 등급인 FR-01, FR-02, FR-03이 모두 바이브코딩 불가 영역(Edge/HW/ML)입니다. 이는 SRS의 Must 요구사항 중 가장 큰 비중(REQ-FUNC-001~010, 10개)이 바이브코딩 범위 밖에 있다는 것을 의미합니다.

---

## 6. 바이브코딩 기반 MVP에서 실제 구현 가능한 범위

### 6.1 ✅ 바이브코딩으로 구현 가능한 영역 (약 60% of Web Part)

| # | 구현 가능 항목 | SRS 참조 | 예상 소요 | 바이브코딩 프롬프트 난이도 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Next.js 프로젝트 셋업 + Prisma 스키마 | §8, §6.2 | 1~2일 | ⭐⭐ |
| 2 | B2C Guardian Portal UI (PWA) | §3.2, FR-05 | 1~2주 | ⭐⭐ |
| 3 | B2B Dashboard UI (Traffic Light) | §3.2, FR-04 | 1~2주 | ⭐⭐ |
| 4 | Daily Report 조회 API | §3.3 #4, #5 | 2~3일 | ⭐⭐ |
| 5 | False Alarm 피드백 API | §3.3 #6 | 1일 | ⭐ |
| 6 | Gemini AI 요약 생성 | §7, FR-05 | 2~3일 | ⭐⭐ |
| 7 | Sleep Trend 차트 | FR-06 | 2~3일 | ⭐⭐ |
| 8 | Dashboard 필터 저장 | FR-08 | 1~2일 | ⭐⭐ |
| 9 | Vercel 배포 + Cron 설정 | §3.1, CON-12 | 1일 | ⭐ |
| 10 | 환경 변수 설정 | §9 | 0.5일 | ⭐ |
| 11 | 기본 NextAuth.js 로그인 | REQ-NF-011 | 2~3일 | ⭐⭐⭐ |
| 12 | Amplitude 이벤트 트래킹 | §3.1 Analytics | 1일 | ⭐ |

**예상 총 기간: 약 5~7주** (풀타임 기준, 바이브코딩 + 디버깅 포함)

### 6.2 🔺 바이브코딩 + 전문가 리뷰 필요 영역

| # | 항목 | SRS 참조 | 위험 요소 |
| :--- | :--- | :--- | :--- |
| 1 | EMR Webhook (HMAC-SHA256) | FR-04, §3.3 #2 | 보안 구현 정확성 |
| 2 | Supabase Realtime 실시간 구독 | FR-04, GAP-01 | WebSocket 라이프사이클 관리 |
| 3 | PWA Service Worker + Web Push | GAP-02 | iOS Safari 호환성, 구독 관리 |
| 4 | NextAuth.js RBAC 미들웨어 | REQ-NF-011 | 인증 우회 방지 |
| 5 | Cold Archival (Supabase Storage) | REQ-NF-017, GAP-03 | Cron 기반 마이그레이션 안정성 |
| 6 | FCM Push Notification | §3.3 #3 | 토큰 관리, 서비스 계정 설정 |

### 6.3 ❌ 바이브코딩 불가 영역 (전문 개발자 필수)

| # | 항목 | SRS 참조 | 필요 전문성 |
| :--- | :--- | :--- | :--- |
| 1 | Edge AI Validator (Deep Learning) | FR-01 전체 | ML Engineer + 데이터 사이언티스트 |
| 2 | UWB 센서 하드웨어/펌웨어 | FR-02 전체 | 임베디드 엔지니어 |
| 3 | 레이더 원시 데이터 비식별화 | FR-03 | 신호처리 엔지니어 |
| 4 | OTA 펌웨어 업데이트 시스템 | §6.3.3 | 펌웨어 엔지니어 |
| 5 | 반려동물 구분 AI 모델 (≥99% 정확도) | REQ-FUNC-003 | ML Engineer + 대규모 데이터셋 |

---

## 7. SRS 문서 자체에 대한 수정 권고

> 아래는 원본 SRS를 수정하지 않으며, 바이브코딩 MVP를 위한 SRS 조정 방향을 제안합니다.

### 7.1 과(過)설계로 판단되는 항목

| SRS 항목 | 과설계 사유 | 권고 조치 |
| :--- | :--- | :--- |
| **11개 Route Handler** | 초급자가 한 번에 생성/관리하기에 과도한 수. 인증 방식도 API Key / JWT / RBAC / HMAC 4종 혼재. | MVP에서는 5~6개로 축소. 인증은 JWT 단일 방식으로 통일 후, EMR HMAC은 Wave 2로 지연. |
| **6개 Server Action** | 각 Action별 입출력이 상세히 정의되어 있으나, 바이브코딩에서는 "하나의 파일에 여러 함수"로 관리하는 것이 효율적. | Server Action 목록은 유지하되, 구현 시 단순화 허용. |
| **Mermaid 시퀀스 다이어그램 7개** | 기획자에게는 이해에 도움이 되나, 바이브코딩에서는 이 복잡한 플로우를 한 번에 구현할 수 없음. | 핵심 시퀀스 2개(일일 리포트 생성, 긴급 알림)로 축소하여 MVP 범위 명확화. |
| **DeadLetterEvent 모델** | EMR Webhook 실패 시 별도 테이블에 저장하는 패턴은 엔터프라이즈 수준. MVP에서는 단순 로그로 충분. | MVP에서 제거. 로그 기반 에러 추적으로 대체. |
| **Hash Chain 무결성 증명** | SHA-256 해시 체인은 법적 증거력 확보를 위한 기능이나, MVP에서는 우선순위 낮음. | Wave 2로 지연. MVP에서는 `integrityHash` 필드만 유지하되, 체인 검증 로직은 생략. |
| **PagerDuty 연동** | 운영 밀착 모니터링은 Beta 이후에 필요. MVP에서는 Slack 알림으로 충분. | PagerDuty → Slack Webhook으로 단순화. |

### 7.2 누락/불명확 항목

| 항목 | 문제점 | 권고 |
| :--- | :--- | :--- |
| **Edge→Server 통신 시뮬레이터** | SRS는 실제 UWB 센서가 데이터를 보내는 것을 전제하나, 바이브코딩 MVP에서는 센서가 없음. 시뮬레이터/모의 데이터에 대한 정의가 없음. | **모의 데이터 생성기(Seed Script)** 사양을 SRS에 추가 필요. `prisma/seed.ts`에 예시 데이터 포함. |
| **에러 핸들링 공통 패턴** | Route Handler별 에러 응답 포맷이 통일되어 있지 않음. | 공통 에러 응답 스키마 (`{ error: string, code: string, details?: any }`) 정의 필요. |
| **UI/UX 와이어프레임** | SRS에 UI 디자인 명세가 없음. 바이브코딩은 UI 프롬프트가 매우 중요. | 최소한 Guardian Portal 3개 페이지, Dashboard 2개 페이지의 와이어프레임 또는 레퍼런스 이미지 추가 권고. |
| **테스트 전략** | Traceability Matrix에 테스트 케이스 ID가 있으나, 실제 테스트 수행 방법(수동/자동)이 불명확. | 바이브코딩 MVP에서는 수동 테스트 체크리스트를 별도 문서로 분리하여 관리 권고. |

---

## 8. 바이브코딩 MVP 실행 전략 제안

### Phase 0: 준비 (1주)

```
✅ Next.js + Prisma + shadcn/ui 프로젝트 초기화
✅ Supabase 프로젝트 생성 + 연동
✅ Vercel 배포 확인 (Hello World)
✅ 환경 변수 설정 (§9)
✅ Prisma 스키마 생성 + seed 데이터 투입
```

### Phase 1: 핵심 화면 (2주)

```
✅ 로그인/회원가입 (NextAuth.js 기본)
✅ Guardian Dashboard 페이지 (일일 리포트 카드)
✅ B2B Dashboard 페이지 (Traffic Light 그리드)
✅ Daily Report 상세 조회 페이지
```

### Phase 2: 데이터 파이프라인 (2주)

```
✅ 모의 이벤트 수신 API (Edge 시뮬레이터 대체)
✅ Daily Report 생성 Server Action + Vercel Cron
✅ Gemini AI 요약 생성 연동
✅ FCM/Web Push 기본 연동
```

### Phase 3: 부가 기능 (1~2주)

```
✅ False Alarm 피드백 기능
✅ Sleep Trend 차트 (Recharts)
✅ Dashboard 필터 저장
✅ PWA manifest + Service Worker 기본 설정
```

### Phase 4: Edge/HW 통합 (전문 개발자 협업 필수)

```
⛔ Edge AI Validator 연동
⛔ 실제 UWB 센서 데이터 연동
⛔ EMR Webhook (HMAC 보안)
⛔ OTA 펌웨어 업데이트
```

> 💡 **TIP:** Phase 0~3은 바이브코딩으로 약 5~7주 내 구현 가능합니다. Phase 4는 전문 개발자 합류 후 별도 진행하며, Phase 0~3의 결과물이 "웹 파트 데모" 역할을 하여 투자/팀 빌딩에 활용할 수 있습니다.

---

## 9. 최종 결론

### ⚖️ 판정 요약

| 질문 | 답변 |
| :--- | :--- |
| 현재 SRS가 바이브코딩 MVP에 적절한가? | **❌ 과설계.** 그러나 범위를 축소하면 웹 파트의 데모 MVP는 구현 가능. |
| 초급자(3개월)가 혼자 전체를 만들 수 있는가? | **❌ 불가.** Edge AI + HW/FW는 반드시 전문가가 필요. |
| 웹 파트만 따로 떼어 바이브코딩으로 만들 수 있는가? | **⭕ 가능.** FR-04(UI 부분), FR-05, FR-06, FR-08 + 인프라 셋업은 5~7주 내 데모 가능. |
| IT 기획 3년차의 도메인 지식이 도움이 되는가? | **⭕ 매우 유리.** 비즈니스 로직, 사용자 흐름, 우선순위 판단에서 순수 개발자보다 오히려 바이브코딩에 유리할 수 있음. |

### 🎯 핵심 권고사항

1. **SRS를 "Web-Only MVP"와 "Edge/HW"로 분리하여 별도 문서화하세요.** 현재 SRS는 두 영역이 혼합되어 있어 바이브코딩 범위 판단이 어렵습니다.

2. **FR-05(Daily Wellness Report + AI Summary)를 바이브코딩 MVP의 1순위 데모 기능으로 삼으세요.** Gemini AI 요약은 제품의 차별화 포인트이면서 바이브코딩에 가장 적합합니다.

3. **모의 데이터 생성기(Mock Data Generator)를 SRS에 정의하세요.** 실제 센서가 없는 상태에서 웹 파트를 개발/테스트하려면 현실적인 모의 데이터가 필수입니다.

4. **인증 체계를 MVP에서 단순화하세요.** API Key + JWT + RBAC + HMAC 4종 혼재 → JWT 단일 인증으로 통일. EMR HMAC 보안은 실제 EMR 연동 시 추가.

5. **UI 와이어프레임 또는 레퍼런스 디자인을 SRS에 추가하세요.** 바이브코딩에서 UI 프롬프트의 품질이 결과물 품질을 결정합니다.

---

**— 검토 완료 —**
