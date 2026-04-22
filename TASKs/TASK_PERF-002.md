---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Performance] PERF-002: 50 동시 디바이스 스트레스 테스트"
labels: 'performance, backend, priority:medium, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [PERF-002] 50개 동시 디바이스 Event Ingest + Heartbeat 스트레스 테스트
- 목적: SRS NFR-PERF-002에서 정의한 MVP 목표 — 50개 동시 디바이스 이벤트 수집 처리 — 를 스트레스 테스트로 검증하고 Supabase Free 연결 한계 내에서 안정적으로 동작함을 확인한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 대상 API: [`/TASKs/TASK_EVT-001.md`](./TASK_EVT-001.md) — POST /api/events/ingest
- 대상 API: [`/TASKs/TASK_HB-001.md`](./TASK_HB-001.md) — POST /api/heartbeat
- 인프라: [`/TASKs/TASK_INFRA-002.md`](./TASK_INFRA-002.md) — Supabase 연결 설정
- SRS 섹션: §8.2 Stress Testing, NFR-PERF-002
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — PERF-002

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 스트레스 테스트 스크립트: `scripts/stress-test.js` (k6 또는 artillery)
- [ ] 시나리오: 50 VU, 5분간 지속, Event Ingest + Heartbeat 혼합
- [ ] Supabase Free 연결 풀 한계 (20개 직접 연결) — PgBouncer 설정 확인:
  `DATABASE_URL=...?pgbouncer=true&connection_limit=1`
- [ ] 측정 지표: 에러율 < 1%, p95 < 5,000ms, DB 연결 거부 에러 0건
- [ ] Supabase Dashboard → Connections 탭에서 최대 연결 수 확인
- [ ] 스트레스 테스트 중 Vercel Function 동시 실행 수 확인 (Hobby: 제한 없음)

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 50 VU 동시 이벤트 수집**
- Given: 50개 가상 디바이스, 각 5분간 1건/분 이벤트 전송
- When: 스트레스 테스트 실행
- Then: 에러율 < 1%, p95 < 5,000ms

**Scenario 2: DB 연결 풀 한계 테스트**
- Given: 50 VU 동시 Prisma 쿼리
- When: 스트레스 테스트 실행
- Then: "too many connections" 에러 0건 (PgBouncer 효과)

**Scenario 3: Heartbeat 동시 처리**
- Given: 50개 디바이스 동시 Heartbeat 전송
- When: POST /api/heartbeat × 50 동시 실행
- Then: 모든 요청 200 응답, lastSeenAt 업데이트 확인

## :gear: Technical & Non-Functional Constraints
- **Supabase Free:** 직접 연결 20개 제한 → `?pgbouncer=true` 필수
- **Vercel Hobby:** 동시 Function 실행 제한 없음 (단, cold start 고려)
- **테스트 환경:** 프로덕션 또는 Supabase 전용 스트레스 테스트 프로젝트
- **목표 디바이스 수:** MVP 50개 (SRS NFR-PERF-002)

## :checkered_flag: Definition of Done (DoD)
- [ ] 50 VU 에러율 < 1% 달성?
- [ ] DB 연결 거부 에러 0건?
- [ ] p95 < 5,000ms 유지?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_INFRA-001, TASK_INFRA-002, TASK_EVT-001, TASK_HB-001
- **Blocks:** 없음
