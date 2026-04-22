---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-014: [Unit] False Alarm 피드백 처리 테스트"
labels: 'test, backend, priority:medium, phase:2'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-014] FA-001/FA-002 False Alarm 피드백 API + Server Action 단위 테스트
- 목적: FA-001(False Alarm API)과 FA-002(Server Action)가 올바른 사용자 권한으로 이상 징후를 false alarm으로 처리하고, 이후 이상치 집계에서 제외되는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_FA-001.md`](./TASK_FA-001.md) — False Alarm API Route
- 테스트 대상: [`/TASKs/TASK_FA-002.md`](./TASK_FA-002.md) — False Alarm Server Action
- SRS 섹션: §5.7 False Alarm Feedback
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-014

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/api/falseAlarm.test.ts`
- [ ] Prisma Mock (`wellnessEvent.update` + `userFeedback.create`)
- [ ] NextAuth 세션 Mock (GUARDIAN 역할)
- [ ] 테스트 케이스:
  - 미인증 요청 → 401
  - 다른 사용자의 deviceId → 403 (권한 없음)
  - 유효한 요청 → `wellnessEvent.falseAlarm: true` 업데이트 확인
  - UserFeedback 레코드 생성 확인 (`userId`, `eventId`, `feedback: "FALSE_ALARM"`)
  - 이미 false alarm 처리된 이벤트 재처리 → 409 Conflict 또는 idempotent 처리
  - Server Action 호출 후 UI revalidatePath 트리거 확인

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 정상 false alarm 처리**
- Given: GUARDIAN 세션, 자신의 device 이벤트 ID
- When: POST /api/false-alarm `{ eventId: "xxx" }`
- Then: 200 OK, `wellnessEvent.falseAlarm: true` 업데이트

**Scenario 2: 권한 없는 device → 403**
- Given: GUARDIAN 세션, 타인 device 이벤트 ID
- When: POST /api/false-alarm
- Then: 403 Forbidden

**Scenario 3: Server Action revalidation**
- Given: Server Action 실행 성공
- When: FA-002 Server Action 호출
- Then: `revalidatePath('/reports')` 호출 확인

## :gear: Technical & Non-Functional Constraints
- **Prisma Mock:** `update` + `create` 호출 순서 및 인수 검증
- **권한 검증:** UserDevice 관계 확인 로직 Mock 포함
- **Server Action:** `next/cache` `revalidatePath` Mock

## :checkered_flag: Definition of Done (DoD)
- [ ] 인증/권한/정상처리/중복처리/revalidation 케이스 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] UserFeedback 레코드 생성 확인?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_FA-001, TASK_FA-002
- **Blocks:** 없음
