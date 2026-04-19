# 🚀 Rooted MVP - 전구간 개발 태스크 (Epic/Feature) 리스트

제공된 `SRS_V1.0(KOR_OPUS).md`를 기반으로 '바이브 코딩'의 MVP 페이즈(Phase 0 ~ Phase 3)에 해당하는 실질 개발 타겟 기능만을 도출했습니다. 명시된 대로 디자인(UI/UX)과 백엔드/프론트 개발/인프라 관점을 명확히 분리하였습니다.

| Task ID | Epic (도메인) | Feature (기능명) | 관련 SRS 섹션 | 선행 태스크 (Dependencies) | 복잡도 (H/M/L) |
|---|---|---|---|---|---|
| INF-001 | Infra (인프라) | 공용 프로젝트 인프라 초기화 (Next.js, Tailwind, shadcn/ui) | §13.2 Phase 0 | None | L |
| INF-002 | Infra (인프라) | Vercel 환경 세팅 및 CI/CD(Git Push) 파이프라인 구성 | §13.2 Phase 0 | INF-001 | L |
| DB-001 | DB (데이터베이스) | Prisma ORM 매핑 및 5개 핵심 모델 스키마 설계 | §6.2, §13.2 | INF-001 | M |
| DB-002 | DB (데이터베이스) | Supabase PostgreSQL 연동 및 마이그레이션 반영 | §3.1, §13.2 | DB-001 | L |
| DB-003 | DB (데이터베이스) | 모의 테스트용 더미 시드(Seed) 생성기 (`seed.ts`) 스크립트 작성 | §14.1, §13.2 | DB-001 | L |
| BE-001 | Backend (인증) | NextAuth.js 기반 로그인/Auth 자물쇠 세팅 (테스트용 개방) | §13.2 Phase 0 | DB-002 | L |
| BE-002 | Backend (API) | 모의 이벤트 데이터 주입망 API 구현 (`/api/events/ingest`) | §6.1, §13.2 | DB-001 | M |
| BE-003 | Backend (API) | 일일 정산 리포트 조회 API 개발 (`/api/reports/daily`) | §6.1, §13.3 | DB-001 | M |
| BE-004 | Backend (API) | B2B 다중 병실 신호등 상태 반환 API (`/api/dashboard/status`) | §6.1, §13.3 | DB-001 | M |
| BE-005 | Backend (API) | 기기 하트비트 생존 신고 수신 기록 API (`/api/devices/heartbeat`) | §6.1, §13.3 | DB-001 | L |
| BE-006 | Backend (API) | 가짜 경보 오작동 항의 접수 API (`/api/events/false-alarm`) | §6.1, §13.4 | DB-001 | L |
| BE-007 | Backend (스케줄러)| 30일 초과 단기 기억(로우 데이터) 스케줄러 폐기/클린업 쿼리 구성 | §4.2.6, §13.3 | DB-001 | M |
| PIPE-001 | AI & Pipeline | Vercel AI SDK + Gemini 1.5 Flash 웰니스 해설 생성 프롬프트 및 연동 | §7.1, §13.4 | None | M |
| PIPE-002 | AI & Pipeline | 1일 1회 Vercel Cron 트리거 웰니스 리포트 자동 생성 (`generateDailyReport`) | §3.4.1, §13.4 | DB-001, PIPE-001 | H |
| 3RD-001 | 3rd-Party | Resend API 연동 메일 발송망 유틸리티 개발 (무료티어 한계) | §3.1, §13.4 | None | L |
| 3RD-002 | 3rd-Party | 비상사태 및 일일 리포트 알람 이메일 발송 자동화 결합 | §6.3.1, §13.4 | BE-002, 3RD-001 | M |
| OPS-001 | Ops (관제) | 무료 DB 일시정지 회피용 외부 핑 봇(UptimeRobot) 세팅 | §11 RISK-07 | INF-002 | L |
| OPS-002 | Ops (관제) | Slack/Discord Webhook 시스템 에러(10% 이상 정전) 알림 연동 | §6.3.2, §13.4 | BE-005 | M |
| UI-001 | UI/UX 디자인 | 시스템 공통 Layout, Header, Auth(NextAuth 로그인) 스크린 스케치 | §8, §13.3 | INF-001 | L |
| UI-002 | UI/UX 디자인 | B2C 보호자 포털 웹: 일간 웰니스 AI 리포트 뷰 패널/카드 UI | §8, §13.3 | UI-001 | M |
| UI-003 | UI/UX 디자인 | B2B 관리 보드 웹: 구역 신호등(다중 병상 컬러 타일) 레이아웃 뷰 | §4.1, §13.3 | UI-001 | M |
| UI-004 | UI/UX 디자인 | B2B 관리 보드 웹: 환자 위험 상태 우선 배치(Triage) 패널 구조 | §4.1, §13.3 | UI-001 | M |
| FE-001 | Frontend (App) | B2C 포털: 리포트 API 통신 렌더링 및 가짜 경보 피드백 버튼 액션 연동 | §13.3 Phase 1 | UI-002, BE-003, BE-006 | M |
| FE-002 | Frontend (App) | B2B 보드: 30초 클라이언트 폴링 호출 및 Red/Yellow 상태 스왑 컴포넌트 | §6.3.1, §13.3 | UI-003, BE-004 | H |
| FE-003 | Frontend (App) | B2B 보드: 프론트엔드 연산 기반 3건 이상 위급 환자 트리아지 정렬 로직 | §6.1, §13.5 | UI-004, FE-002 | M |
| UI-005 | UI/UX 디자인 | (Phase 3) Recharts 기반의 시계열별 웰니스 스코어 트렌드 렌더 차트 | §4.1, §13.5 | UI-002 | M |
| FE-004 | Frontend (App) | (Phase 3) B2B 대시보드 리스트 필터링 트리 액션 적용 | §4.1, §13.5 | UI-003 | L |
| ENH-001 | Frontend (App) | (Phase 3) PWA 앱 래퍼 변환 (Service Worker + Manifest 부여) | §13.5 Phase 3 | INF-001 | L |
| ENH-002 | Frontend (App) | (Phase 3) Vercel Analytics 및 Umami 트래커 스크립트 도포 | §13.5 Phase 3 | INF-002 | L |

> **⚠️ 무결성을 위한 본 MVP 제외 목록 (상상해서 임의 추가하지 않은 기능 - Wave 2 지연분)**:
> - Edge/하드웨어 자체 AI 추론 부문 (UWB 연동)
> - PagerDuty 엔터프라이즈 장애 관제 / 외부망 EMR Webhook 직연동(HMAC)
> - Supabase Realtime(웹소켓 실시간), Push 네이티브(FCM/Web Push), SMS/카카오톡 유료 회선
> - 90일+ 초장기 Cold Archiving 스토리지, 세분화된 RBAC 권한 적용
