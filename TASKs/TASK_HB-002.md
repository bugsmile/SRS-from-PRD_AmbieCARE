---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] HB-002: updateDeviceStatus Server Action 구현"
labels: 'feature, server-action, priority:high, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [HB-002] `app/actions/devices.ts` — `updateDeviceStatus` Server Action 구현
- 목적: API-008 인터페이스 명세를 바탕으로, Prisma Client를 이용해 `SensorDevice`의 하트비트 시간과 상태를 업데이트하는 비즈니스 로직을 구현한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- API Spec: [`/TASKs/TASK_API-008.md`](./TASK_API-008.md)
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#§3.3.1`](../SRS-Draft/SRS_V03(ENG_OPUS).md) — SA #4
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — HB-002

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `app/actions/devices.ts` 파일 생성 및 `'use server'` 추가
- [ ] `updateDeviceStatus` 함수 구현:
  - Prisma `update` 수행. `deviceId`로 조회하여 `lastHeartbeatAt`과 `status` 갱신
  - 레코드 부재 시 catch하여 에러 반환 포맷(`{ success: false, error: ... }`) 적용

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 상태 업데이트 성공**
- Given: 등록된 `deviceId`
- When: Action 호출
- Then: DB에 업데이트 반영 및 `success: true`

## :gear: Technical & Non-Functional Constraints
- **Phase:** Phase 1
- **직렬화:** 반환 객체는 순수 직렬화 가능 객체(Plain Object)여야 함.

## :checkered_flag: Definition of Done (DoD)
- [ ] `updateDeviceStatus` 로직이 정상 동작하는가?
- [ ] 에러 핸들링이 적용되었는가?

## :construction: Dependencies & Blockers
- **Depends on:** API-008, DB-007
- **Blocks:** HB-001, HB-004
