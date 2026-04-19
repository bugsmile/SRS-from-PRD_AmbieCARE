# SRS v0.1 vs v0.3 비교 분석 보고서

> **문서 ID:** SRS_V01-03_Compare  
> **작성일:** 2026-04-20  
> **비교 대상:** `SRS_v0.1(KOR_OPUS).md` (Rev 1.0, 2026-04-18) ↔ `SRS_V03(KOR_OPUS).md` (Rev 3.0, 2026-04-20)

---

## 1. 기술 스택의 명확성

v0.1은 기술 스택을 본문 곳곳에 흩뿌리며 암시적으로만 언급한 반면, v0.3은 문서 헤더부터 기술 스택을 선언하고 전체 요구사항에 Phase 태그/제약 ID를 부여하여 구현 가능성을 명시적으로 통제합니다.

### 1.1 아키텍처 및 인프라

| 비교 항목 | SRS v0.1 | SRS v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **코어 프레임워크** | 명시 없음 (암시적 클라우드 백엔드) | **Next.js App Router** 풀스택 (문서 헤더에 공식 선언) | 프론트/백엔드 이원화 제거, 단일 코드베이스 명시 |
| **데이터베이스** | 추상적 "이벤트 DB" 표기 | **Prisma ORM + Supabase Free PostgreSQL** (500MB 한도 명시) | DB 제품·한도·ORM까지 확정 |
| **배포 인프라** | AWS Lambda, S3, CloudWatch, Glacier | **Vercel Hobby** (100 GB-hr, 일 1회 Cron, 10초 타임아웃) | AWS 완전 탈피 → Vercel 서버리스 전환 |
| **AI/LLM 연동** | 없음 | **Vercel AI SDK + Google Gemini 1.5 Flash** (RPM 15, 일 1,500회 Free Quota) | AI 웰니스 내러티브 요약 기능 신설 |
| **실시간 통신** | WebSocket (내부 구현 가정) | **Phase 1: API 폴링 30초** → Phase 2: Supabase Realtime | 무료 티어 한계에 맞춘 단계적 전환 명시 |
| **푸시 알림** | FCM + APNs (HTTP/2 Push) | **Resend Email API Free** (일 100건) — FCM/Web Push는 Wave 2 | 유료 서비스 완전 배제, 이메일로 대체 |
| **모니터링/Ops** | PagerDuty, Datadog APM | **Slack/Discord Webhook** (무료) + UptimeRobot Free | 유료 Ops 도구 전면 철거 |
| **분석 도구** | Amplitude / Mixpanel (SDK) | **Vercel Analytics / Umami** (무료 오픈소스) | 유료 분석 플랫폼 교체 |
| **아카이브 스토리지** | S3 Hot 90일 → Glacier Cold 3년 | **PostgreSQL 30일 핫** → 자동 삭제. 콜드 아카이빙은 Wave 2 | 냉동 스토리지 MVP에서 완전 제거 |

### 1.2 API/백엔드 구조

| 비교 항목 | SRS v0.1 | SRS v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **API 스타일** | 전통 REST API 엔드포인트 11개 | **Next.js Route Handlers 6개** (MVP Core) + Server Actions 4개 | 엔드포인트 45% 축소, Server Actions 패턴 신설 |
| **인증 방식** | JWT Bearer Token + RBAC (완전 구현 가정) | **NextAuth.js JWT** (Phase 1) → RBAC 미들웨어는 Phase 2 분리 | 인증 라이브러리 구체화, 권한 제어 단계화 |
| **API 인증** | TLS 1.3 Client 인증서 / API Key + HMAC | **환경 변수 API Key** (외부) / **NextAuth JWT** (내부) — HMAC은 Wave 2 | 보안 수준을 MVP 현실에 맞춤 |
| **EMR Webhook** | HTTP POST + HMAC-SHA256 (Must) | **Wave 2로 연기** (DEP-01 미체결, HMAC은 전문가 필요) | Must → Could로 우선순위 하향 |

### 1.3 데이터 모델

| 비교 항목 | SRS v0.1 | SRS v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **모델 수** | 5개 (SensorDevice, WellnessEvent, UserAccount, DailyReport, Facility) | **5개** (SensorDevice, WellnessEvent, UserAccount, **UserDevice**, DailyReport) | Facility 제거, M:N 조인 테이블(UserDevice) 신설 |
| **ID 타입** | UUID | **String (cuid())** — SQLite 호환성 확보 | Prisma/SQLite 호환 구조 |
| **ENUM 처리** | 네이티브 ENUM 사용 | **String 필드**로 대체 (SQLite ENUM 미지원) | 크로스 DB 호환 |
| **`facilityId` FK** | SensorDevice, UserAccount에 존재 | **제거** — Facility 모델 자체 Wave 2 이관 | B2B 시설 관리 기능 MVP 제외 |
| **DeadLetterEvent** | 암시적 존재 (EMR 실패 큐) | **삭제** — EMR Wave 2 연기에 따라 불필요 | 모델 간소화 |
| **`aiSummary` 필드** | 없음 | **DailyReport에 신설** (Gemini 생성 텍스트 저장) | AI 내러티브 기능 반영 |

### 1.4 환경변수 및 설정

| 비교 항목 | SRS v0.1 | SRS v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **변수 수** | 명시 없음 | **8개** 명확히 정의 (§9) | 실제 구현에 필요한 설정을 문서화 |
| **제거된 변수** | — | `FCM_SERVER_KEY`, `VAPID_*`, `EMR_WEBHOOK_*` 등 5개 | 유료/미구현 서비스 관련 변수 정리 |
| **신규 변수** | — | `RESEND_API_KEY`, `SLACK_WEBHOOK_URL` | 무료 대체 서비스 도입 반영 |

---

## 2. MVP 목표 및 가치전달 조정 내용

v0.3은 "전액 무료 인프라에서 1인 바이브 코딩으로 5–7주 내 MVP 론칭"이라는 현실적 제약을 수용하면서 핵심 가치(데일리 웰니스 리포트 + AI 요약)를 더 명확하게 강화합니다.

### 2.1 개발 방식 및 대상

| 비교 항목 | SRS v0.1 | SRS v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **대상 개발자** | 개발팀(복수) 가정 | **1인 개발자** (SW 경험 3개월, IT 기획 3년) — 프로필 표로 명시 | 인력 현실 반영 |
| **개발 방식** | 명시 없음 (전통 개발 가정) | **바이브 코딩** (AI 코딩 어시스턴트 기반) — 문서 헤더 선언 | 개발 방법론의 명확한 정의 |
| **인프라 예산** | 기기당 월 ≤ 500 KRW | **전액 무료 ($0/월)** — AI 도구 구독료만 허용 | 비용 제약 극단적 강화 |
| **개발 기간** | 명시 없음 | **5–7주** (Phase 0–3 + Wave 2 별도) | 타임라인 공식 명시 |

### 2.2 범위(Scope) 3-Tier 체계

| 비교 항목 | SRS v0.1 | SRS v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **범위 계층** | In-Scope / Out-of-Scope 이분법 | **3-Tier**: MVP Core (Phase 0–3) → Phase 2 확장 → Wave 2 | 점진적 구현 로드맵 추가 |
| **Phase 태그** | 없음 | 모든 FR/NFR에 **Phase 태그** 부여 (Phase 0–3 / Wave 2) | 개발 우선순위 즉시 파악 가능 |
| **클라이언트 앱** | iOS 앱 (MVP), Android (Wave 2) | **웹 앱** (MVP) → **PWA** (Phase 3) → 네이티브 (Wave 2) | iOS 네이티브 포기, 웹 우선 전략 |

### 2.3 기능 우선순위 변경 (MoSCoW 조정)

| 기능 영역 | v0.1 우선순위 | v0.3 우선순위 | 변경 사유 |
| :--- | :--- | :--- | :--- |
| **FR-05: 데일리 웰니스 리포트** | Should | **Must ⬆️ (격상)** | MVP 킬러 기능으로 재정의 — 바이브 코딩에 최적화된 핵심 가치 |
| **FR-01: AI 오경보 필터링 엔진** | Must | Must (유지, **Wave 2 지연**) | Edge/HW 영역 — 바이브 코딩 범위 밖 |
| **FR-02: Zero-Friction 센서** | Must | Must (유지, **Wave 2 지연**) | 하드웨어 영역 — 전문 엔지니어 필요 |
| **FR-03: 프라이버시 비영상 트래킹** | Must | Must (유지, **Wave 2 지연**) | Edge 데이터 처리 — 바이브 코딩 범위 밖 |
| **FR-04: B2B 대시보드 + EMR** | Must (통합) | Must (UI만) / **Could ⬇️** (EMR 부분 하향) | EMR 연동 Wave 2 분리 |
| **FR-07: SMS/카카오톡 폴백** | Could | **Won't ⬇️ (강등)** | 유료 서비스 — 무료 원칙 위배 |

### 2.4 핵심 가치 전달 변경

| 가치 영역 | v0.1 접근 | v0.3 접근 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **핵심 킬러 기능** | 오경보 제로 AI 엔진 (Edge 의존) | **AI 웰니스 내러티브 리포트** (Gemini 기반) | HW 없이 소프트웨어만으로 차별화 |
| **알림 방식** | 실시간 푸시 (FCM/APNs, p95 ≤ 2초) | **이메일 알림** (5분 내) — 실시간 푸시 Wave 2 | "관찰 중심 정적 모니터링"으로 재정의 |
| **SLA 목표** | ≥ 99.9% (월 43.8분 다운타임) | **Best Effort ~99%** (무료 인프라 한계 수용) | 현실적 가용성 목표 설정 |
| **데이터 보관** | Hot 90일 → Cold 3년 (해시 체인) | **Hot 30일 → 자동 삭제** (500MB 한계) | 저장소 제약 반영, 법적 증거 Wave 2 이관 |
| **E2E 레이턴시** | p95 ≤ 2,000 ms | **p95 ≤ 5,000 ms** (콜드 스타트 감안) | 서버리스 무료 티어 실현 가능 목표 |
| **동시 접속 규모** | 1,000대 (스트레스 5,000대) | **50대 한정** (Free Tier 제약) | 무료 인프라에 맞춘 MVP 스케일 |
| **AI 오경보 지표** | 가구당 월 ≤ 0.3건 (자체 달성) | 가구당 월 ≤ 0.3건 (**Wave 2 Edge AI 의존**) | MVP에서는 Edge 없이 검증 불가 명시 |
| **B2B EMR 이중입력** | 자동 연동, 이중 입력 0건 | 자동 연동, 이중 입력 0건 **(Wave 2)** | MVP에서 달성 불가 명시 |

### 2.5 검증 계획(Validation Plan) 조정

| 실험 ID | v0.1 조건 | v0.3 조건 | 변경 사유 |
| :--- | :--- | :--- | :--- |
| **EXP-01** | 요양원 5곳, 150병상 | **2–3개 요양병원, 30–50대** | 현실 가능한 MVP 베타 규모로 축소 |
| **EXP-02** | 100–200 가구 오픈 베타 | **20–50 가구** | 50대 기기 한정에 맞춤 |
| **EXP-02 분석도구** | Amplitude `view_daily_report` | **Vercel Analytics / Umami** | 무료 대체 도구로 전환 |
| **EXP-01/03 시점** | 명시 없음 | **Wave 2** (Edge AI 필요) | HW 의존 실험은 Wave 2로 명시 이관 |

---

## 3. 기타 차이점

### 3.1 문서 구조 및 분량

| 비교 항목 | SRS v0.1 | SRS v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **총 분량** | 961줄 / 약 70KB | **1,533줄 / 약 191KB** | 約 2.7배 확대 |
| **섹션 수** | 6개 (§1–§6) | **15개 (§1–§15)** | 9개 섹션 신설 |
| **개정 이력** | 없음 | **개정 이력 테이블** (v1.0→v2.0→v3.0) | 변경 추적 체계 도입 |

### 3.2 신규 추가 섹션 (v0.3에만 존재)

| 섹션 번호 | 섹션명 | 내용 요약 |
| :--- | :--- | :--- |
| **§7** | AI 지능 연계 스펙 | Vercel AI SDK + Gemini 규격, Use Cases, Prompt 구조, 모델 교체 전략 |
| **§8** | 프로젝트 폴더 트리 구조 | Next.js 기반 MVP 전체 디렉토리 구조 명세 |
| **§9** | 환경 변수 명세 | 8개 환경변수 전체 목록 및 용도 |
| **§10** | Sprint Estimation | 기능별 바이브 코딩 소요 기간 견적 (총 5–7주) |
| **§11** | Risk Assessment | 10개 리스크 시나리오 (v0.1의 3개 → 10개로 확장) |
| **§12** | Gap Analysis & Mitigation | 10개 약점 갭 식별 + 6개 신규 역량 자산 정리 |
| **§13** | Phase별 마일스톤 상세 | Phase 0–3 + Wave 2 단계별 태스크/산출물/연관 규격 |
| **§14** | Mock Data 규격 | Seed 데이터 스크립트 명세, 샘플 JSON 포함 |
| **§15** | Free Tier 족쇄 규격 | 6개 무료 서비스별 한도/여유분/오버플로우 대응 표 |

### 3.3 시퀀스 다이어그램 변경

| 다이어그램 | v0.1 | v0.3 | 변경 내용 |
| :--- | :--- | :--- | :--- |
| **§3.4.1 데일리 리포트** | UWB 센서 → 클라우드 → FCM/APNs | **Mock Generator → Server Action → Prisma → Gemini AI → Resend Email** | 전체 파이프라인 재설계 |
| **§3.4.2 AI Validator** | Edge → Cloud → FCM 푸시 | Edge → **Route Handler → Resend Email** (Wave 2 명시) | FCM → 이메일 대체, Wave 2 태그 |
| **§3.4.4 EMR 동기화** | Cloud → EMR Webhook + WebSocket | **Route Handler → Supabase Realtime → EMR** (Wave 2 명시) | 전체 Wave 2로 이관 표기 |
| **§6.3.2 오프라인 감지** | Heartbeat Monitor → PagerDuty 호출 | **Route Handler → Slack/Discord Webhook** (PagerDuty 제거) | 무료 Ops 대체 |
| **§6.3.3 OTA 펌웨어** | Cloud → OTA Service → Edge | 구조 유지, **Wave 2 명시** | Edge/FW 범위 명확화 |

### 3.4 컴포넌트 다이어그램 변경

| 비교 항목 | v0.1 | v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **제거된 컴포넌트** | — | PagerDuty, FCM, APNs, Web Push, EMR 시스템, Cold Archival (S3/Glacier), Supabase Realtime (Phase 1 기준) | 유료/미구현 외부 서비스 정리 |
| **신규 컴포넌트** | — | Resend Email API, Slack/Discord Webhook, Mock Data Generator, Vercel Analytics/Umami | 무료 대체 서비스 + 시뮬레이터 추가 |
| **개조된 구조** | Route Handlers 11개, Cron 다수 | **Route Handlers 6개, Cron 일 1회**, Realtime → API 폴링 30초 | 무료 티어 한계에 맞춘 경량화 |

### 3.5 제약사항(Constraints) 확장

| 비교 항목 | v0.1 | v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **제약 수** | 5개 (CON-01 ~ CON-05) | **16개** (CON-01 ~ CON-16) | 기술 스택 및 무료 티어 제약 11개 추가 |
| **신규 제약 유형** | — | 기술 스택 강제 (CON-06~12), Free Tier 한도 (CON-13~15), 완전 무료 원칙 (CON-16) | 구현 환경의 물리적 제약을 문서로 통제 |

### 3.6 가정(Assumptions) 변경

| 비교 항목 | v0.1 | v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **가정 수** | 3개 (ASM-01 ~ ASM-03) | **9개** (ASM-01 ~ ASM-09, ASM-04 삭제) | 기술 스택 관련 가정 6개 추가 |
| **ASM-04 삭제** | — | ~~Vercel Pro SLA 99.99%~~ → Hobby 플랜은 SLA 보장 없음 | Pro → Hobby 전환에 따른 가정 무효화 |
| **신규 가정** | — | Vercel Hobby 한도 충분(ASM-07), Supabase 500MB 충분(ASM-08), Gemini Free Quota 충분(ASM-09) | 무료 티어 용량 적합성 가정 |

### 3.7 의존성(Dependencies) 변경

| 비교 항목 | v0.1 | v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **DEP-01 EMR 벤더** | 제휴 필수 | 제휴 필수 **(Wave 2, 미체결 명시)** | 미체결 상태를 문서에 공식 기록 |
| **DEP-03 푸시** | FCM/APNs 의존 | **Resend Email API Free**로 전환 | 외부 푸시 서비스 의존 해소 |
| **DEP-04 Realtime** | — | Supabase Realtime **(Phase 2, MVP Core는 API 폴링)** | 단계별 의존성 명시 |
| **DEP-05 Cron** | — | Vercel Cron **(Hobby 일 1회 제한)** | 제한 사항을 의존성으로 기록 |
| **DEP-06 AI SDK** | — | Vercel AI SDK + Gemini (Free Quota) | LLM 의존성 신규 등록 |

### 3.8 리스크 관리 확대

| 비교 항목 | v0.1 | v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **리스크 수** | 제약사항으로 간접 관리 | **10개 공식 리스크** (RISK-01 ~ RISK-10) | 별도 리스크 섹션 신설, 확률/영향도 정량화 |
| **해결 완료** | — | RISK-01(콜드스타트), RISK-03(Realtime), RISK-04(스케일) — v3.0에서 아키텍처 변경으로 해결 | 설계 결정이 리스크를 해소한 이력 추적 |
| **v0.3 신규** | — | RISK-07(Supabase 7일 정지), RISK-08(Vercel 한도), RISK-09(Resend 한도), RISK-10(NextAuth 보안) | 무료 티어 특유의 리스크 사전 식별 |

### 3.9 용어 및 문체 변경

| 비교 항목 | SRS v0.1 | SRS v0.3 | 변경 의미 |
| :--- | :--- | :--- | :--- |
| **용어** | "오경보(False Alarm)" | **"오작동(False Alarm)"** | 용어 통일 |
| **신규 용어** | — | PWA, Route Handler, Server Action, Supabase Realtime, Vibe-Coding, Free Tier 등 6개 추가 | 기술 스택 전환에 따른 용어 보강 |
| **참고 문헌** | 7개 (REF-01 ~ REF-07) | **15개** (REF-01 ~ REF-15) | Next.js, Prisma, Supabase, Vercel AI SDK, Resend 등 8개 추가 |
| **문체 톤** | 공식적·학술적 | 공식적 + **개발자 친화적 구어체** (특히 §10 이후) | 바이브 코딩 1인 개발자에게 맞춘 실전적 톤 |

---

## 요약

| 관점 | 핵심 변경 한 줄 요약 |
| :--- | :--- |
| **1. 기술 스택 명확성** | AWS 추상 아키텍처 → **Next.js + Prisma + Supabase Free + Vercel Hobby + Gemini Flash** 구체 확정. 환경변수·폴더구조·Free Tier 한도까지 문서화. |
| **2. MVP 목표/가치전달** | "실시간 응급 푸시 + Edge AI 오경보 제거" → **"AI 내러티브 웰니스 리포트(이메일) + 대시보드 조회"** 중심으로 핵심 가치 재정의. HW 의존 기능은 Wave 2, 유료 서비스는 무료 대체. |
| **3. 기타 차이점** | 문서 2.7배 확대(961→1,533줄), 9개 섹션 신설(AI 스펙·Sprint 견적·리스크·갭 분석·Phase 가이드·Mock Data·Free Tier 규격), 제약 5→16개, 리스크 0→10개로 MVP 실현 가능성 대폭 강화. |
