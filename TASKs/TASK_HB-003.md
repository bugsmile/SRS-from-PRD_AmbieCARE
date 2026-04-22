---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] HB-003: 15분 초과 미수신 디바이스 식별 로직 (Query)"
labels: 'feature, query, priority:medium, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [HB-003] [Query] Heartbeat 요청 시 전체 활성 디바이스의 `lastHeartbeatAt` 확인 → 15분 초과 미수신 디바이스 식별 로직
- 목적: 매 하트비트 수신 시점이나 특정 폴링 주기에, 마지막 통신 후 15분이 경과한 디바이스를 쿼리하여 다운타임을 감지하기 위한 기반을 마련한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#§6.3.2`](../SRS-Draft/SRS_V03(ENG_OPUS).md) — Heartbeat Sequence
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#REQ-FUNC-008`](../SRS-Draft/SRS_V03(ENG_OPUS).md)
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — HB-003

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `checkInactiveDevices` 유틸리티 함수 또는 로직 구현 (`app/actions/devices.ts` 내)
- [ ] Prisma 쿼리 로직:
  - `status: "ACTIVE"`이고 `lastHeartbeatAt < (현재 시간 - 15분)` 인 레코드 조회
- [ ] 해당 디바이스들의 ID 목록 반환

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 15분 경과 디바이스 감지**
- Given: 20분 전 마지막 통신된 `ACTIVE` 디바이스 존재
- When: 쿼리 실행
- Then: 해당 디바이스 ID가 결과에 포함된다.

## :gear: Technical & Non-Functional Constraints
- **최적화:** 이 쿼리는 자주 실행될 수 있으므로, 인덱스 활용 및 가벼운 셀렉트 문 구성. (MVP 규모에서는 무리 없음)
- **Phase:** Phase 1

## :checkered_flag: Definition of Done (DoD)
- [ ] Prisma Query가 정확하게 15분 기준을 적용하는가?
- [ ] 식별된 ID 리스트를 정상 반환하는가?

## :construction: Dependencies & Blockers
- **Depends on:** HB-001
- **Blocks:** HB-004
