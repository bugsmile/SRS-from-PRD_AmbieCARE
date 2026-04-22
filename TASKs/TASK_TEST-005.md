---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-005: [Unit] Dashboard Status API 테스트"
labels: 'test, backend, priority:high, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-005] GET /api/dashboard/status B2B Dashboard API 단위 테스트
- 목적: DASH-001에서 구현한 Dashboard Status API가 B2B 관리자 인증, 디바이스 상태 집계, Traffic Light 판정 로직을 올바르게 처리하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_DASH-001.md`](./TASK_DASH-001.md) — GET /api/dashboard/status
- API 계약: [`/TASKs/TASK_API-006.md`](./TASK_API-006.md) — Dashboard Status API 규격
- SRS 섹션: §5.1 B2B Dashboard, AC of API-006
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-005

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/api/dashboard/status.test.ts`
- [ ] Prisma Client Mock 설정
- [ ] NextAuth.js ADMIN 역할 세션 Mock 설정
- [ ] 테스트 케이스:
  - 미인증 요청 → 401
  - GUARDIAN 역할 요청 → 403 (ADMIN only)
  - 정상 요청, 디바이스 전체 NORMAL → 200 + GREEN 상태
  - ALERT 디바이스 1개 이상 → YELLOW 판정
  - OFFLINE 디바이스 비율 ≥ 10% → RED 판정
  - 빈 디바이스 목록 → 200 + 빈 배열

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: ADMIN 인증, 전체 NORMAL**
- Given: ADMIN 세션, 전체 디바이스 최근 이벤트 NORMAL
- When: GET /api/dashboard/status
- Then: 200 OK + `{ status: "GREEN", devices: [...] }`

**Scenario 2: ALERT 디바이스 존재 → YELLOW**
- Given: ADMIN 세션, 1개 이상 디바이스 ALERT 상태
- When: GET /api/dashboard/status
- Then: 200 OK + `{ status: "YELLOW", ... }`

**Scenario 3: OFFLINE ≥ 10% → RED**
- Given: ADMIN 세션, 전체 10개 중 1개 이상 OFFLINE
- When: GET /api/dashboard/status
- Then: 200 OK + `{ status: "RED", ... }`

## :gear: Technical & Non-Functional Constraints
- **테스트 격리:** Prisma Mock 사용
- **역할 검증:** ADMIN vs GUARDIAN 역할 분기 테스트 포함
- **Traffic Light 로직:** GREEN/YELLOW/RED 3가지 상태 모두 커버

## :checkered_flag: Definition of Done (DoD)
- [ ] 인증/역할/상태 판정 6개 케이스 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] Traffic Light 3개 상태 분기 100% 커버?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_DASH-001, TASK_API-006
- **Blocks:** 없음
