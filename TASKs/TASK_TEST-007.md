---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-007: [Unit] 30일 초과 데이터 삭제 테스트"
labels: 'test, backend, priority:medium, phase:0'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-007] DB-008 데이터 보존 정책 — 30일 초과 WellnessEvent 삭제 단위 테스트
- 목적: DB-008에서 구현한 데이터 보존 Cron이 30일 초과 WellnessEvent만 삭제하고 최근 데이터를 보존하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_DB-008.md`](./TASK_DB-008.md) — 30일 보존 정책 구현
- SRS 섹션: §6.4 Data Retention Policy, NFR-DATA-001
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-007

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/db/retention.test.ts`
- [ ] Prisma Mock 또는 SQLite in-memory 테스트 DB 설정
- [ ] `beforeEach`: 30일 이내 데이터 5개 + 31일 초과 데이터 3개 insert
- [ ] 테스트 케이스:
  - 삭제 함수 실행 후 30일 초과 레코드 0개 확인
  - 삭제 함수 실행 후 30일 이내 레코드 5개 보존 확인
  - 정확히 30일 경계 레코드 — 보존 여부 확인 (30일 포함 보존)
  - 삭제 레코드 수 반환값 === 3 확인
  - 빈 DB에서 삭제 실행 → 에러 없이 0 반환

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 30일 초과 데이터 삭제**
- Given: timestamp가 현재 -31일인 WellnessEvent 3개 존재
- When: retention 삭제 함수 실행
- Then: 해당 3개 레코드 삭제, 최근 데이터 보존

**Scenario 2: 경계값 보존**
- Given: timestamp가 정확히 -30일인 WellnessEvent 1개 존재
- When: retention 삭제 함수 실행
- Then: 해당 레코드 삭제되지 않음 (30일 포함 보존)

**Scenario 3: 삭제 카운트 반환**
- Given: 31일 초과 데이터 3개
- When: 삭제 함수 실행
- Then: 반환값 `deletedCount === 3`

## :gear: Technical & Non-Functional Constraints
- **날짜 조작:** `jest.useFakeTimers()` 또는 날짜 주입 패턴 사용
- **테스트 DB:** SQLite in-memory 권장 (실제 삭제 동작 검증)
- **경계값:** 정확히 30일 경계 케이스 필수 포함

## :checkered_flag: Definition of Done (DoD)
- [ ] 삭제/보존/경계/빈DB 4개 케이스 구현?
- [ ] 모든 테스트 GREEN?
- [ ] 날짜 Mock 처리 올바른가?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_DB-008
- **Blocks:** 없음
