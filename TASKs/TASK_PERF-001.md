---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Performance] PERF-001: E2E 응답 지연 p95 ≤ 5,000ms 검증"
labels: 'performance, backend, priority:high, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [PERF-001] 핵심 API 엔드포인트 p95 응답 지연 5,000ms 이하 검증
- 목적: SRS NFR-PERF-001에서 정의한 p95 ≤ 5,000ms 성능 목표를 달성하기 위해 주요 API 엔드포인트(Event Ingest, Dashboard Status, Daily Report 조회)의 응답 시간을 측정하고 임계값을 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 대상 API: [`/TASKs/TASK_EVT-001.md`](./TASK_EVT-001.md) — POST /api/events/ingest
- 대상 API: [`/TASKs/TASK_DASH-001.md`](./TASK_DASH-001.md) — GET /api/dashboard/status
- 대상 API: [`/TASKs/TASK_RPT-001.md`](./TASK_RPT-001.md) — GET /api/reports/daily
- SRS 섹션: §8.1 Performance Requirements, NFR-PERF-001
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — PERF-001

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 성능 테스트 도구 선택: `k6`, `autocannon`, 또는 `artillery`
- [ ] 테스트 시나리오 작성: 각 엔드포인트 30초간 10 VU(가상 사용자)
- [ ] Vercel 스테이징 환경 또는 프로덕션 배포 후 측정
- [ ] 측정 지표: p50, p95, p99, max 응답 시간
- [ ] Supabase 쿼리 최적화 — 인덱스 활용 쿼리 플랜 확인 (`EXPLAIN ANALYZE`)
- [ ] Vercel Function 타임아웃 10초 제한 — 9초 이내 완료 확인
- [ ] 느린 쿼리 발견 시 Prisma `select` 최적화 또는 인덱스 추가

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: Event Ingest p95 ≤ 5,000ms**
- Given: 10 VU, 30초, Supabase Free 환경
- When: POST /api/events/ingest 부하 테스트
- Then: p95 응답 시간 ≤ 5,000ms

**Scenario 2: Dashboard Status p95 ≤ 5,000ms**
- Given: 10 VU, Prisma 쿼리 포함
- When: GET /api/dashboard/status 부하 테스트
- Then: p95 응답 시간 ≤ 5,000ms

**Scenario 3: Vercel Function 타임아웃 이내**
- Given: 임의 API 요청
- When: 응답 시간 측정
- Then: 최대 응답 시간 < 9,000ms (Vercel 10초 제한 내)

## :gear: Technical & Non-Functional Constraints
- **Vercel Hobby:** 10초 Function timeout — 필수 준수
- **Supabase Free:** 동시 연결 수 제한 — 연결 풀링 (`?pgbouncer=true`) 적용
- **테스트 환경:** 프로덕션 Supabase 데이터로 측정 (mock 데이터 불가)
- **목표:** p95 ≤ 5,000ms (SRS NFR-PERF-001)

## :checkered_flag: Definition of Done (DoD)
- [ ] 3개 핵심 API p95 ≤ 5,000ms 달성?
- [ ] Vercel Function 타임아웃 이내 동작?
- [ ] 성능 측정 결과 리포트 생성?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_INFRA-001 (Vercel 배포), TASK_DB-001~005 (인덱스 적용)
- **Blocks:** 없음
