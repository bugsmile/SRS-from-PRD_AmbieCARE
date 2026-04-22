---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-006: [Unit] Heartbeat API 테스트"
labels: 'test, backend, priority:high, phase:0'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-006] POST /api/heartbeat Heartbeat API 단위 테스트
- 목적: HB-001에서 구현한 Heartbeat API가 API Key 인증, deviceId 유효성, 타임스탬프 업데이트를 올바르게 처리하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_HB-001.md`](./TASK_HB-001.md) — POST /api/heartbeat
- API 계약: [`/TASKs/TASK_API-003.md`](./TASK_API-003.md) — Heartbeat API 규격
- SRS 섹션: §5.4 Device Heartbeat, AC of API-003
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-006

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/api/heartbeat.test.ts`
- [ ] Prisma Client Mock 설정 (`sensorDevice.update` mock)
- [ ] 테스트 케이스:
  - API Key 미전송 → 401
  - 잘못된 API Key → 401
  - 존재하지 않는 `deviceId` → 404
  - 유효한 요청 → 200 + `{ lastSeenAt }` 업데이트 확인
  - `deviceId` 미전송 → 400
  - Prisma update 실패 시 → 500

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: API Key 미전송 → 401**
- Given: Authorization 헤더 없음
- When: POST /api/heartbeat `{ deviceId: "xxx" }`
- Then: 401 Unauthorized

**Scenario 2: 유효한 요청 → 200**
- Given: 유효한 API Key + 존재하는 deviceId
- When: POST /api/heartbeat `{ deviceId: "xxx" }`
- Then: 200 OK + `{ lastSeenAt: <ISO timestamp> }`

**Scenario 3: 존재하지 않는 deviceId → 404**
- Given: 유효한 API Key, 미등록 deviceId
- When: POST /api/heartbeat `{ deviceId: "unknown" }`
- Then: 404 Not Found

## :gear: Technical & Non-Functional Constraints
- **테스트 격리:** Prisma Mock 사용
- **타임스탬프:** `lastSeenAt` 업데이트 여부 mock 반환값으로 확인
- **실행 속도:** 단위 테스트 5초 이내 완료

## :checkered_flag: Definition of Done (DoD)
- [ ] 인증/유효성/정상 케이스 6개 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] Prisma `sensorDevice.update` mock 호출 횟수 검증?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_HB-001, TASK_API-003
- **Blocks:** 없음
