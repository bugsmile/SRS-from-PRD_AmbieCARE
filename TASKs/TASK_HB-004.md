---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] HB-004: 15분 이상 Heartbeat 미수신 시 디바이스 INACTIVE 상태 업데이트"
labels: 'feature, mutation, priority:medium, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [HB-004] [Command] 15분 이상 Heartbeat 미수신 시 디바이스 status → `INACTIVE` 업데이트
- 목적: HB-003에서 식별된 디바이스들을 실제로 `INACTIVE` 상태로 변경하여 Dashboard Traffic Light (RED)에 즉각 반영되도록 처리한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#REQ-FUNC-008`](../SRS-Draft/SRS_V03(ENG_OPUS).md)
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — HB-004

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] HB-003의 `checkInactiveDevices` 결과 리스트 수신
- [ ] 식별된 `deviceId` 배열에 대해 Prisma `updateMany` 호출:
  - `status` 필드를 `"INACTIVE"`로 변경
- [ ] 실행 시점 검토: `generateDailyReport` Cron 수행 시 일괄 실행하거나, Dashboard Status(API-003) API 조회 시 동적으로 상태 업데이트 유발 고려. (MVP는 폴링 API 호출 시 상태 점검 유발 등 단순화 접근 허용)

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: INACTIVE 전환**
- Given: HB-003 로직에 의해 식별된 기기
- When: 업데이트 로직 실행
- Then: 해당 기기의 상태가 DB에 `INACTIVE`로 기록된다.

## :gear: Technical & Non-Functional Constraints
- **트랜잭션/배치:** 여러 대의 업데이트 시 `updateMany`로 단일 처리하여 통신 오버헤드 축소.
- **Phase:** Phase 1

## :checkered_flag: Definition of Done (DoD)
- [ ] `updateMany` 로직이 올바르게 작성되었는가?
- [ ] 상태가 `INACTIVE`로 성공적으로 전환되는가?

## :construction: Dependencies & Blockers
- **Depends on:** HB-003
- **Blocks:** Dashboard 상태 갱신 정확도 향상
