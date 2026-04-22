---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-004: [Unit] Daily Report 조회 API 테스트"
labels: 'test, backend, priority:high, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-004] GET /api/reports/daily 조회 API 단위 테스트
- 목적: RPT-001에서 구현한 Daily Report 조회 API가 날짜 파라미터 유효성, 인증, 데이터 반환 형식을 올바르게 처리하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_RPT-001.md`](./TASK_RPT-001.md) — GET /api/reports/daily
- API 계약: [`/TASKs/TASK_API-005.md`](./TASK_API-005.md) — Daily Report API 규격
- SRS 섹션: §5.3 Daily Report Query, AC of API-005
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-004

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/api/reports/daily.test.ts`
- [ ] Prisma Client Mock 설정 (`jest.mock('@/lib/prisma')`)
- [ ] NextAuth.js 세션 Mock 설정
- [ ] 테스트 케이스:
  - 미인증 요청 → 401
  - `date` 파라미터 누락 → 400
  - `date` 형식 오류 (non-ISO) → 400
  - 유효한 요청, 데이터 있음 → 200 + DailyReport 객체
  - 유효한 요청, 데이터 없음 → 404
  - `deviceId` 권한 없는 device → 403

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 미인증 요청 → 401**
- Given: NextAuth 세션 없음
- When: GET /api/reports/daily?date=2025-01-01
- Then: 401 Unauthorized 반환

**Scenario 2: 정상 조회 → 200**
- Given: 유효한 세션, 해당 날짜 DailyReport 존재
- When: GET /api/reports/daily?date=2025-01-01&deviceId=xxx
- Then: 200 OK + `{ id, deviceId, date, sleepScore, bathroomCount, anomalyFlag }` 반환

**Scenario 3: 데이터 없음 → 404**
- Given: 유효한 세션, 해당 날짜 DailyReport 미존재
- When: GET /api/reports/daily?date=2025-01-01&deviceId=xxx
- Then: 404 Not Found 반환

## :gear: Technical & Non-Functional Constraints
- **테스트 격리:** Prisma Mock 사용, 실제 DB 연결 없음
- **인증 Mock:** `next-auth/jwt` 또는 `getServerSession` Mock
- **응답 스키마:** Zod 또는 수동 타입 검증

## :checkered_flag: Definition of Done (DoD)
- [ ] 인증/파라미터/데이터 존재 여부 6개 케이스 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] Prisma Mock이 실제 DB를 호출하지 않는가?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_RPT-001, TASK_API-005
- **Blocks:** 없음
