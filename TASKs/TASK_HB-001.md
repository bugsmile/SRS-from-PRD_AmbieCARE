---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] HB-001: POST /api/devices/[deviceId]/heartbeat/route.ts — API Route Handler 구현"
labels: 'feature, api, priority:high, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [HB-001] POST `/api/devices/[deviceId]/heartbeat/route.ts` 구현
- 목적: API-004 명세를 바탕으로 하트비트 요청을 수신하고, API Key 인증 후 `updateDeviceStatus` Server Action을 호출하여 디바이스의 온라인 상태를 갱신한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- API Spec: [`/TASKs/TASK_API-004.md`](./TASK_API-004.md) — DTO 명세
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#§3.3`](../SRS-Draft/SRS_V03(ENG_OPUS).md) — Route #6
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#§6.3.2`](../SRS-Draft/SRS_V03(ENG_OPUS).md) — Heartbeat Sequence
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — HB-001

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `app/api/devices/[deviceId]/heartbeat/route.ts` 파일 생성
- [ ] API Key 검증 로직 구현 (`x-api-key` 헤더 검사)
- [ ] 파라미터(deviceId) 추출 및 `updateDeviceStatus` Server Action (HB-002) 호출
  - 파라미터: `deviceId`, `status: "ACTIVE"`, `lastHeartbeatAt: new Date()`
- [ ] 결과에 따른 HTTP 상태 코드 반환 (`200 OK`, `404 Not Found`, `401 Unauthorized`)

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 정상 하트비트 처리**
- Given: 올바른 API Key와 등록된 `deviceId`
- When: POST 요청
- Then: `200 OK`와 업데이트된 시간 반환

**Scenario 2: 미등록 디바이스**
- Given: 미등록 `deviceId`
- When: POST 요청
- Then: `404 Not Found`

## :gear: Technical & Non-Functional Constraints
- **Phase:** Phase 1
- **성능:** 매우 빈번하게 호출되므로(5분 간격), 로직을 최소화하여 빠른 응답 속도 보장.

## :checkered_flag: Definition of Done (DoD)
- [ ] Route Handler가 정상적으로 생성되었는가?
- [ ] 인증이 동작하는가?
- [ ] Server Action 호출이 연동되었는가?

## :construction: Dependencies & Blockers
- **Depends on:** API-004, HB-002
- **Blocks:** Dashboard 실시간 상태 모니터링
