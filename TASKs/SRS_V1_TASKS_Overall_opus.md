# Rooted MVP — SRS V1.0 기반 전체 개발 태스크 리스트

> **출처 문서:** `SRS_V1.0(KOR_OPUS).md`  
> **추출 기준일:** 2026-04-20  
> **추출 모델:** Claude Opus  
> **원칙:** SRS에 명시되지 않은 기능은 임의 추가하지 않음. UI/UX 디자인과 백엔드·프론트엔드·인프라 관점을 분리.

---

## 범례 (Legend)

| 약어 | 의미 |
|---|---|
| **Phase** | Phase 0 (1주) → Phase 1 (2주) → Phase 2 (2주) → Phase 3 (1~2주) → Wave 2 (별도) |
| **복잡도** | H = High, M = Medium, L = Low |
| **Epic 접두어** | INF = 인프라, DB = 데이터베이스, AUTH = 인증, API = Route Handler, SA = Server Action, CRON = 스케줄러, AI = AI연동, EMAIL = 이메일알림, OPS = 운영/모니터링, SEC = 보안/컴플라이언스, FE = 프론트엔드 개발, UI = UI/UX 디자인, QA = 품질검증, WAVE2 = Wave 2 |

---

## Part 1 — 인프라 & 환경 (Infrastructure)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| INF-001 | Infra | `create-next-app` + Tailwind CSS + TypeScript 프로젝트 초기화 | §8, §13.2, CON-06, CON-09 | None | L | 0 |
| INF-002 | Infra | shadcn/ui CLI 설치 및 `components.json` 기본 구성 | §8, CON-09 | INF-001 | L | 0 |
| INF-003 | Infra | 환경 변수 8종 구성 (`.env.local`, `.env.example`) | §9 | INF-001 | L | 0 |
| INF-004 | Infra | Vercel Hobby 플랜 프로젝트 생성 및 Git Push 자동 배포 설정 | §3.1, §13.2, CON-12, CON-13 | INF-001 | L | 0 |
| INF-005 | Infra | `vercel.json` — Cron Job 스케줄 설정 (일 1회) | §13.2, CON-13, DEP-05 | INF-004 | L | 0 |
| INF-006 | Infra | `next.config.ts` 기본 설정 (App Router, 이미지 도메인 등) | §8 | INF-001 | L | 0 |

---

## Part 2 — 데이터베이스 (Database)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| DB-001 | DB | Supabase Free 프로젝트 생성 및 PostgreSQL 연결 문자열 확보 | §3.1, CON-08, CON-14 | INF-003 | L | 0 |
| DB-002 | DB | Prisma ORM 초기화 및 `prisma/schema.prisma` 파일 생성 | §3.6, §6.2, CON-08 | DB-001 | L | 0 |
| DB-003 | DB | `SensorDevice` 모델 정의 (9개 필드, cuid PK) | §6.2.1, §3.6 | DB-002 | L | 0 |
| DB-004 | DB | `WellnessEvent` 모델 정의 (10개 필드, timestamp 인덱스) | §6.2.2, §3.6 | DB-002 | M | 0 |
| DB-005 | DB | `UserAccount` 모델 정의 (7개 필드, email unique) | §6.2.3, §3.6 | DB-002 | L | 0 |
| DB-006 | DB | `UserDevice` 조인 테이블 정의 (복합 PK) | §6.2.4, §3.6 | DB-003, DB-005 | L | 0 |
| DB-007 | DB | `DailyReport` 모델 정의 (9개 필드, date 인덱스, aiSummary nullable) | §6.2.5, §3.6 | DB-002 | M | 0 |
| DB-008 | DB | Prisma 마이그레이션 실행 및 Supabase 스키마 반영 | §13.2 | DB-003 ~ DB-007 | L | 0 |
| DB-009 | DB | Prisma Client 싱글턴 유틸리티 (`lib/prisma.ts`) | §8, §3.7 | DB-002 | L | 0 |
| DB-010 | DB | 모의 Seed 스크립트 (`prisma/seed.ts`) — 디바이스 3~5대 + 유저 3명 + 7일치 이벤트/리포트 | §14.1, §14.2, §13.2 | DB-008, DB-009 | M | 0 |

---

## Part 3 — 인증 (Authentication)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| AUTH-001 | Auth | NextAuth.js 기본 설정 (`lib/auth.ts`) — JWT 단일 세션 방식 | §4.2.3 REQ-NF-011, §13.2 | INF-003, DB-009 | M | 0 |
| AUTH-002 | Auth | Credentials Provider (데모용 이메일/비밀번호 로그인) | §13.2, REQ-NF-011 | AUTH-001 | L | 0 |
| AUTH-003 | Auth | JWT 콜백 — role 필드(GUARDIAN/FACILITY_ADMIN) 토큰 내장 | §6.2.3, REQ-NF-011 | AUTH-001 | L | 1 |
| AUTH-004 | Auth | 미들웨어 — 보호 라우트 진입 시 JWT 유효성 검증 | §4.2.3 REQ-NF-011 | AUTH-003 | M | 1 |

---

## Part 4 — 백엔드 API Route Handlers (MVP Core 6종)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| API-001 | API | `POST /api/events/ingest` — Mock/Edge 이벤트 데이터 주입. API Key 인증. | §3.3 Route #1, §6.1 #1, §13.2 | DB-009, INF-003 | M | 0 |
| API-002 | API | `POST /api/mock/generate` — 데모 시연용 즉석 목(Mock) 이벤트 실시간 생성 엔드포인트 | §8 mock/generate, §14.1 | API-001 | L | 0 |
| API-003 | API | `GET /api/reports/daily/[deviceId]/[date]` — 특정 기기+날짜 일간 리포트 조회. JWT 인증. | §3.3 Route #2, §6.1 #2, §13.3 | DB-007, DB-009, AUTH-004 | M | 1 |
| API-004 | API | `GET /api/dashboard/status` — 다중 병상 신호등 상태(Red/Yellow/Green) JSON 반환. JWT 인증. | §3.3 Route #5, §6.1 #5, §13.3 | DB-003, DB-009, AUTH-004 | M | 1 |
| API-005 | API | `POST /api/devices/[deviceId]/heartbeat` — 기기 생존 신고 핑 수신. API Key 인증. lastHeartbeatAt 갱신. | §3.3 Route #6, §6.1 #6, §13.3 | DB-003, DB-009, INF-003 | L | 1 |
| API-006 | API | `POST /api/events/[eventId]/false-alarm` — 보호자 오작동 신고 접수. isFalseAlarm=true 마킹. JWT 인증. | §3.3 Route #3, §6.1 #3, §13.4 | DB-004, DB-009, AUTH-004 | L | 2 |
| API-007 | API | `POST /api/ai/wellness-summary` — Gemini 1.5 Flash 호출 → AI 내러티브 요약 생성 반환. JWT 인증. | §3.3 Route #4, §6.1 #4, §7.1, §13.4 | AI-001, AUTH-004 | M | 2 |

---

## Part 5 — 백엔드 Server Actions (MVP Core 4종)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| SA-001 | Server Action | `createWellnessEvent` (`app/actions/events.ts`) — 이벤트 DB 삽입 + integrityHash 생성 | §3.3.1 Action #1, §13.2 | DB-004, DB-009 | M | 0 |
| SA-002 | Server Action | `updateDeviceStatus` (`app/actions/devices.ts`) — 하트비트 수신 시 lastHeartbeatAt 갱신 | §3.3.1 Action #4, §13.3 | DB-003, DB-009 | L | 1 |
| SA-003 | Server Action | `updateFalseAlarmFlag` (`app/actions/events.ts`) — isFalseAlarm 불리언 스위칭 | §3.3.1 Action #2, §13.4 | DB-004, DB-009 | L | 2 |
| SA-004 | Server Action | `generateDailyReport` (`app/actions/reports.ts`) — 전일 이벤트 집계 → sleepScore 산출 → bathroomVisitCount 카운트 → anomalyFlags 검출 → AI 요약 결합 → DailyReport 생성 → 이메일 발송 | §3.3.1 Action #3, §3.4.1, §13.4 | DB-004, DB-007, AI-002, EMAIL-001 | H | 2 |

---

## Part 6 — 스케줄러 / Cron (Scheduled Jobs)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| CRON-001 | Cron | Vercel Cron Job → `generateDailyReport` 트리거 (일 1회 아침) | §3.4.1, §13.4, CON-13, DEP-05 | INF-005, SA-004 | M | 2 |
| CRON-002 | Cron | 30일 초과 WellnessEvent 자동 삭제 배치 쿼리 (integrityHash 보존) | §4.2.6 REQ-NF-017, §13.3 | DB-004, DB-009 | M | 1 |

---

## Part 7 — AI 연동 (Gemini / Vercel AI SDK)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| AI-001 | AI | Vercel AI SDK + `@ai-sdk/google` 설치 및 `lib/ai.ts` 초기 설정 (AI_MODEL 환경 변수 기반 모델 스왑) | §7.1, §7.4, CON-10, CON-11, §9 | INF-003 | L | 2 |
| AI-002 | AI | AI 웰니스 서사 내러티브 생성 — System/User 프롬프트 구조 구현 (한국어 100단어 제한, 금지어 필터) | §7.2 NEW-01, §7.3 | AI-001 | M | 2 |
| AI-003 | AI | AI 이상 징후 해설 — anomalyFlags 감지 시 경고 내러티브 자동 생성 (+50% 화장실 체류 등) | §7.2 NEW-02, §7.3 | AI-002 | M | 2 |
| AI-004 | AI | Gemini API 장애 시 Fallback — aiSummary = null 처리, 사용자 에러 미노출 | §7.1 Fallback | AI-002 | L | 2 |

---

## Part 8 — 이메일 알림 (Resend Free)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| EMAIL-001 | Email | Resend API 클라이언트 유틸리티 (`lib/email.ts`) — 기본 발송 함수 구현 | §3.1, §8, REF-13 | INF-003 | L | 2 |
| EMAIL-002 | Email | 일일 웰니스 리포트 이메일 발송 (보호자 대상, Cron 후 자동) | §3.4.1, REQ-FUNC-020 | EMAIL-001, SA-004 | M | 2 |
| EMAIL-003 | Email | 긴급 응급 이벤트(EMERGENCY) 발생 시 즉각 경고 이메일 발송 (5분 내 마지노선) | §6.3.1, REQ-FUNC-004 | EMAIL-001, API-001 | M | 2 |
| EMAIL-004 | Email | 기기 오프라인(15분 하트비트 미수신) 경고 이메일 발송 (1회성) | §6.3.2, REQ-FUNC-008 | EMAIL-001, API-005 | M | 2 |

---

## Part 9 — Ops / 운영 모니터링

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| OPS-001 | Ops | UptimeRobot(또는 GitHub Actions) — Supabase Free 7일 일시정지 방지 일일 핑 설정 | §11 RISK-07, §13.2 | INF-004, DB-001 | L | 0 |
| OPS-002 | Ops | Slack/Discord Incoming Webhook 유틸리티 (`lib/slack.ts`) | §3.1, §8 | INF-003 | L | 2 |
| OPS-003 | Ops | 전체 기기 대비 ≥10% 오프라인 감지 시 Slack/Discord 긴급 알림 발송 | §4.2.2 REQ-NF-007, §6.3.2 | OPS-002, API-005 | M | 2 |
| OPS-004 | Ops | 하트비트 Route Handler 내 미접속 기기 일제 불시 검문 로직 (15분 기준, 3회 연속 결석) | §6.3.2, REQ-FUNC-008 | API-005, SA-002 | M | 2 |

---

## Part 10 — 보안 & 컴플라이언스

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| SEC-001 | Security | API Key 기반 인증 — events/ingest, heartbeat Route에 env 기반 검증 | §3.3, §6.1 | INF-003 | L | 0 |
| SEC-002 | Security | GitHub Actions CI — 금지어(diagnosis, medical, patient) Linter 게이트 | §4.2.7 REQ-NF-019, CON-04 | INF-004 | L | 0 |
| SEC-003 | Security | Prisma 스키마 PII 점검 (인체 식별 가능 필드 0건 확인) | §4.2.3 REQ-NF-009 | DB-008 | L | 0 |
| SEC-004 | Security | TLS 1.3 적용 확인 — Vercel Hobby 기본 제공 검증 | §4.2.3 REQ-NF-008 | INF-004 | L | 0 |

---

## Part 11 — UI/UX 디자인 (화면 설계·시각 디자인)

> **주의:** 이 파트는 코드 구현이 아닌 화면 레이아웃/비주얼 디자인 작업입니다.

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| UI-001 | UI/UX Design | 공통 Root Layout 와이어프레임 (헤더, 사이드바, 반응형 구조) | §8 layout.tsx | None | L | 0 |
| UI-002 | UI/UX Design | 로그인/회원가입 화면 디자인 (`(auth)/login`, `(auth)/register`) | §8, §3.2 | UI-001 | L | 1 |
| UI-003 | UI/UX Design | B2C 보호자 포털: 대시보드 메인 화면 디자인 (일간 요약 카드 배치) | §3.2, §8 `(guardian)/dashboard` | UI-001 | M | 1 |
| UI-004 | UI/UX Design | B2C 보호자 포털: 일일 리포트 상세 뷰 디자인 (수면점수, 화장실횟수, AI요약, anomaly 배너) | §8 `(guardian)/reports/[date]` | UI-003 | M | 1 |
| UI-005 | UI/UX Design | B2C 보호자 포털: 오작동 신고 피드백 버튼 UI 디자인 | REQ-FUNC-005 | UI-004 | L | 2 |
| UI-006 | UI/UX Design | B2B 관제 대시보드: 다중 병상 신호등 타일 그리드 디자인 (Red/Yellow/Green Badge) | §3.2, §8 `(admin)/dashboard` | UI-001 | M | 1 |
| UI-007 | UI/UX Design | B2B 관제 대시보드: Triage 우선순위 패널 디자인 (상단 고정, 점멸, 비프음 아이콘) | REQ-FUNC-012, §3.7 TriageEngine | UI-006 | M | 3 |
| UI-008 | UI/UX Design | 기기 상태 인디케이터 컴포넌트 디자인 (ACTIVE/INACTIVE/MAINTENANCE) | §8 `shared/device-status-indicator.tsx` | UI-001 | L | 1 |
| UI-009 | UI/UX Design | Anomaly Alert 경고 배너 컴포넌트 디자인 (기기 점검 소요 등) | §8 `reports/anomaly-alert.tsx`, REQ-FUNC-019 | UI-004 | L | 2 |
| UI-010 | UI/UX Design | 이메일 템플릿 디자인 — 일일 리포트용 / 긴급 경고용 / 기기 오프라인용 | EMAIL-002~004 | None | M | 2 |
| UI-011 | UI/UX Design | (Phase 3) 수면 트렌드 차트 뷰 디자인 (7일/30일 시계열 곡선) | REQ-FUNC-021, FR-06 | UI-004 | M | 3 |
| UI-012 | UI/UX Design | (Phase 3) B2B 대시보드 필터 UI 디자인 (동/호수 태그 필터링) | REQ-FUNC-023, FR-08 | UI-006 | L | 3 |
| UI-013 | UI/UX Design | 랜딩 페이지 디자인 (`app/page.tsx`) | §8 | UI-001 | L | 1 |

---

## Part 12 — 프론트엔드 개발 (Frontend Implementation)

### 12-A. 공통 / 인증

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| FE-001 | Frontend | Root Layout 구현 (`app/layout.tsx`, `globals.css`) | §8 | UI-001, INF-001 | L | 0 |
| FE-002 | Frontend | 로그인 페이지 구현 (`(auth)/login/page.tsx`) — NextAuth 연동 | §8, AUTH-002 | UI-002, AUTH-002 | M | 1 |
| FE-003 | Frontend | 회원가입 페이지 구현 (`(auth)/register/page.tsx`) | §8 | UI-002 | L | 1 |
| FE-004 | Frontend | Guardian 라우트 그룹 Layout (`(guardian)/layout.tsx`) — 인증 가드 | §8 | AUTH-004 | L | 1 |
| FE-005 | Frontend | Admin 라우트 그룹 Layout (`(admin)/layout.tsx`) — 인증 가드 | §8 | AUTH-004 | L | 1 |
| FE-006 | Frontend | 랜딩 페이지 구현 (`app/page.tsx`) | §8 | UI-013 | L | 1 |

### 12-B. B2C 보호자 포털

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| FE-007 | Frontend (B2C) | 보호자 대시보드 페이지 구현 — 연결된 기기 목록 + 최근 리포트 카드 렌더링 | §8 `(guardian)/dashboard/page.tsx`, §3.2 | UI-003, API-003, FE-004 | M | 1 |
| FE-008 | Frontend (B2C) | 일일 리포트 상세 페이지 구현 — sleepScore, bathroomVisitCount, aiSummary 표시 | §8 `(guardian)/reports/[date]/page.tsx`, FR-05 | UI-004, API-003 | M | 1 |
| FE-009 | Frontend (B2C) | 일일 리포트용 카드 컴포넌트 (`daily-report-card.tsx`) | §8, REQ-FUNC-016~018 | UI-004 | M | 1 |
| FE-010 | Frontend (B2C) | Anomaly Alert 배너 컴포넌트 (`anomaly-alert.tsx`) — anomalyFlags 기반 shadcn/ui Alert | §8, REQ-FUNC-019 | UI-009 | L | 2 |
| FE-011 | Frontend (B2C) | 오작동 신고 피드백 버튼 구현 — API-006 호출 → isFalseAlarm 마킹 | REQ-FUNC-005 | UI-005, API-006, SA-003 | M | 2 |
| FE-012 | Frontend (B2C) | INSUFFICIENT_DATA 상태 — "거주 부재 모드" 안내 화면 분기 처리 | REQ-FUNC-018 | FE-008 | L | 2 |
| FE-013 | Frontend (B2C) | SENSOR_ERROR 상태 — "기기 점검 소요" 경고 화면 분기 처리 | REQ-FUNC-019 | FE-008, FE-010 | L | 2 |

### 12-C. B2B 관제 대시보드

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| FE-014 | Frontend (B2B) | 다중 병상 신호등 대시보드 페이지 구현 — `traffic-light-card.tsx` 컴포넌트 + 30초 API 폴링 | §8 `(admin)/dashboard/page.tsx`, REQ-FUNC-011 | UI-006, API-004, FE-005 | H | 1 |
| FE-015 | Frontend (B2B) | 신호등 카드 컴포넌트 (`traffic-light-card.tsx`) — shadcn/ui Badge 기반 Red/Yellow/Green 색상 스왑 | §8, §3.7, REQ-FUNC-011 | UI-006 | M | 1 |
| FE-016 | Frontend (B2B) | 30초 간격 클라이언트 폴링 로직 (setInterval → GET /api/dashboard/status) | REQ-FUNC-011, GAP-01 | API-004 | M | 1 |
| FE-017 | Frontend (B2B) | 기기 상태 인디케이터 공용 컴포넌트 (`device-status-indicator.tsx`) | §8 | UI-008 | L | 1 |

### 12-D. Phase 3 확장

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| FE-018 | Frontend | Triage 정렬 패널 구현 (`triage-list.tsx`) — 3건 이상 동시 Red 시 위험도 기반 자동 정렬 + 비프음 | REQ-FUNC-012, §3.7 TriageEngine, `lib/triage.ts` | UI-007, FE-014 | M | 3 |
| FE-019 | Frontend | 수면 트렌드 차트 구현 — Recharts 라이브러리, 7일/30일 시계열 곡선 | REQ-FUNC-021, FR-06 | UI-011, API-003 | M | 3 |
| FE-020 | Frontend | B2B 대시보드 필터 UI 구현 — 동/호수별 필터링, shadcn/ui DataTable | REQ-FUNC-023, FR-08 | UI-012, FE-014 | L | 3 |
| FE-021 | Frontend | PWA 전환 — `manifest.json` + Service Worker + 오프라인 지원 | §1.2, GAP-02, §13.5 | FE-001 | M | 3 |
| FE-022 | Frontend | Vercel Analytics / Umami 통계 트래커 SDK 삽입 | §3.1, REQ-NF-015, §13.5 | INF-004 | L | 3 |

---

## Part 13 — 핵심 도메인 로직 (비즈니스 규칙)

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| BIZ-001 | Business Logic | 수면 질 평가 점수 산출 알고리즘 (0~100점, 오차율 <10%) | REQ-FUNC-016, REQ-NF-003, §3.4.1 | DB-004 | H | 2 |
| BIZ-002 | Business Logic | 화장실 방문 횟수 카운트 로직 | REQ-FUNC-016, §3.4.1 | DB-004 | M | 2 |
| BIZ-003 | Business Logic | 화장실 체류 시간 +50% 이상 이상치 감지 → anomalyFlags 플래그 삽입 | REQ-FUNC-017, §3.4.1 | BIZ-002 | M | 2 |
| BIZ-004 | Business Logic | 문짝 50회 이상 등 극단적 아웃라이어 데이터 여과 → "기기 점검 소요" 플래그 | REQ-FUNC-019, §3.4.1 | DB-004 | M | 2 |
| BIZ-005 | Business Logic | 하루 데이터 <5시간 시 INSUFFICIENT_DATA 상태 처리 (빈 리포트 발행) | REQ-FUNC-018, §3.4.1 | DB-004, DB-007 | L | 2 |
| BIZ-006 | Business Logic | 기기 오프라인 판정 — lastHeartbeatAt > 15분, 3회 연속 결석 카운트 | REQ-FUNC-008, §6.3.2 | SA-002, API-005 | M | 2 |
| BIZ-007 | Business Logic | Triage 위험 점수 연산 로직 (`lib/triage.ts`) — 동시 3건 이상 Red 시 우선순위 정렬 | REQ-FUNC-012, §3.7 | DB-004 | M | 3 |
| BIZ-008 | Business Logic | 신호등 상태 결정 로직 — 이벤트 유형/시간/confidence에 따른 Red/Yellow/Green 분류 | REQ-FUNC-011, §3.7 | DB-004 | M | 1 |

---

## Part 14 — 품질 검증 / 테스팅

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| QA-001 | QA | p95 레이턴시 ≤5,000ms 부하 테스트 (50대 동시 접속 시뮬레이션) | REQ-NF-001, REQ-NF-004 | API-001 ~ API-007 | M | 1 |
| QA-002 | QA | Vercel Hobby Serverless 100 GB-hr 사용량 모니터링 검증 | CON-13, RISK-08, ASM-07 | INF-004 | L | 1 |
| QA-003 | QA | Supabase Free 500MB DB 용량 모니터링 및 30일 클린업 동작 확인 | CON-14, ASM-08, REQ-NF-017 | CRON-002 | L | 1 |
| QA-004 | QA | Gemini API 무료 쿼터 (15 RPM, 1500/일) 내 동작 확인 (50대 × 1일 1요청) | CON-15, ASM-09, RISK-06 | AI-002, CRON-001 | L | 2 |
| QA-005 | QA | Resend Free 일 100건 이메일 한도 내 동작 확인 | RISK-09, §15 | EMAIL-002 | L | 2 |
| QA-006 | QA | isFalseAlarm 월간 누적 ≤2건/가구 PMF 지표 검증 쿼리 | REQ-NF-014, EXP-01 | SA-003 | L | 2 |
| QA-007 | QA | 수면점수 오차율 <10% 검증 — 수기 대조 | REQ-NF-003, EXP-02 | BIZ-001 | M | 2 |
| QA-008 | QA | 월 운영비 $0 확인 — Vercel + Supabase 청구 페이지 교차 검증 | REQ-NF-012 | INF-004, DB-001 | L | 0 |
| QA-009 | QA | NextAuth JWT 보안 체크리스트 — 실전 배포 전 수동 감사 | RISK-10, REQ-NF-011 | AUTH-001 ~ AUTH-004 | M | 1 |

---

## Part 15 — Wave 2 (별도 일정, 전문 개발자 필요)

> SRS에 명시되어 있으나 MVP Core 범위 밖이며, 별도 일정·인력으로 추진하는 항목입니다.

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 | Phase |
|---|---|---|---|---|---|---|
| WAVE2-001 | Edge AI/HW | UWB 레이더 센서 + Edge AI Validator (딥러닝 추론) 구현 | FR-01, REQ-FUNC-001~003 | N/A (HW 전문) | H | Wave 2 |
| WAVE2-002 | Edge AI/HW | 제로 마찰 센서: 설치 후 무개입 운영, 자동 캘리브레이션 | FR-02, REQ-FUNC-006~007 | WAVE2-001 | H | Wave 2 |
| WAVE2-003 | Edge AI/HW | 프라이버시: Edge 내 로우 데이터 비식별 메타 변환, 영상 0 | FR-03, REQ-FUNC-009~010 | WAVE2-001 | H | Wave 2 |
| WAVE2-004 | EMR | EMR Webhook (`/api/webhooks/emr`) — HMAC-SHA256 인증 + 3회 백오프 재시도 | FR-04 REQ-FUNC-013~014, §3.4.4 | DEP-01 (벤더 계약) | H | Wave 2 |
| WAVE2-005 | EMR | DeadLetterEvent 스키마 복원 및 전송 실패 격리 로직 | §3.4.4, GAP-08 | WAVE2-004 | M | Wave 2 |
| WAVE2-006 | Notification | SMS/카카오톡 유료 폴백 알림 채널 | FR-07 REQ-FUNC-022 | N/A (유료 계약) | M | Wave 2 |
| WAVE2-007 | Notification | FCM/Web Push 네이티브 실시간 푸시 (<60초) | §3.1, REQ-FUNC-004 (궁극) | FE-021 (PWA) | H | Wave 2 |
| WAVE2-008 | Ops | PagerDuty Sev1 장애 에스컬레이션 연동 | REQ-NF-007 (풀버전), §3.1 | N/A (유료) | M | Wave 2 |
| WAVE2-009 | Storage | 90일+ 콜드 아카이빙 (Supabase Storage / 별도 빙하 저장소) | REQ-NF-017 (풀버전) | N/A | M | Wave 2 |
| WAVE2-010 | Auth | Full RBAC 미들웨어 — 역할 기반 접근 제어 | REQ-NF-011 (풀버전) | AUTH-003 | H | Wave 2 |
| WAVE2-011 | Edge AI/HW | OTA 무선 펌웨어 업데이트 파이프라인 | REQ-NF-013, §6.3.3 | WAVE2-001 | H | Wave 2 |
| WAVE2-012 | API | 콜드 아카이브 열람 API (`/api/events/archive`) + RBAC 인증 | §3.3 Route #11 | WAVE2-009, WAVE2-010 | M | Wave 2 |
| WAVE2-013 | API | 수면 트렌드 전용 API (`/api/reports/trend/[deviceId]`) | §3.3 Route #7 | FE-019 | L | Wave 2 |
| WAVE2-014 | API | Web Push 발사 API (`/api/notifications/push`) | §3.3 Route #8 | WAVE2-007 | M | Wave 2 |
| WAVE2-015 | API | 대시보드 필터 저장 API (`/api/dashboard/filters`) | §3.3 Route #9 | FE-020 | L | Wave 2 |
| WAVE2-016 | Edge AI/HW | 반려동물 분리 필터 (10kg 미만 골격 분류, 99% 정확도) | REQ-FUNC-003 | WAVE2-001 | H | Wave 2 |
| WAVE2-017 | Infra | Vercel Pro + Supabase Pro 업그레이드 (SLA ≥99.9%, 500대 확장) | REQ-NF-005(풀), REQ-NF-018(풀) | N/A | M | Wave 2 |
| WAVE2-018 | Business Logic | 해시 체인 무결성 검증 로직 (integrityHash 법적 증거력) | REQ-FUNC-015 일부 | DB-004 | H | Wave 2 |
| WAVE2-019 | Server Action | `createUser` — NextAuth 내장 대체가 아닌 직접 사용자 관리 | §3.3.1 Action #6 | WAVE2-010 | L | Wave 2 |
| WAVE2-020 | Server Action | `saveDashboardFilter` — 필터 프리셋 영구 저장 | §3.3.1 Action #5 | WAVE2-015 | L | Wave 2 |
| WAVE2-021 | Infra | Edge Runtime 전환 (p95 ≤2,000ms 목표) | REQ-NF-001 (Wave 2 목표) | WAVE2-017 | M | Wave 2 |
| WAVE2-022 | Infra | Rate Limiting (분당 100요청) 미들웨어 | §1.2 제외 항목 | WAVE2-017 | M | Wave 2 |
| WAVE2-023 | QA/Field | EXP-01 B2B 현장 베타 — 2~3개 요양병원 30~50대 4주 운영 | §6.4 EXP-01 | WAVE2-001~003 | H | Wave 2 |
| WAVE2-024 | QA/Field | EXP-03 제로마찰 증명 — CRM 환불/항의 CS 0건 달성 검증 | §6.4 EXP-03 | WAVE2-002 | M | Wave 2 |

---

## 종합 통계

| 구분 | 태스크 수 |
|---|---|
| Infrastructure (INF) | 6 |
| Database (DB) | 10 |
| Authentication (AUTH) | 4 |
| API Route Handlers (API) | 7 |
| Server Actions (SA) | 4 |
| Cron / Scheduler (CRON) | 2 |
| AI Integration (AI) | 4 |
| Email Notification (EMAIL) | 4 |
| Ops / Monitoring (OPS) | 4 |
| Security / Compliance (SEC) | 4 |
| UI/UX Design (UI) | 13 |
| Frontend Dev (FE) | 22 |
| Business Logic (BIZ) | 8 |
| QA / Testing (QA) | 9 |
| Wave 2 (WAVE2) | 24 |
| **총계** | **125** |

---

| Phase | MVP 태스크 수 (Wave 2 제외) |
|---|---|
| Phase 0 (Foundation, 1주) | 28 |
| Phase 1 (Core UI, 2주) | 26 |
| Phase 2 (Pipeline, 2주) | 33 |
| Phase 3 (Enhancements, 1~2주) | 14 |
| **MVP 소계** | **101** |
| Wave 2 (별도) | 24 |

---

> **면책 고지:** 본 태스크 리스트는 `SRS_V1.0(KOR_OPUS).md`에 명시된 내용만으로 도출하였으며, SRS에 기술되지 않은 기능은 포함하지 않았습니다. Wave 2 항목은 SRS 문서 내 명시된 연기 대상을 그대로 반영한 것입니다.
