---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-017: [Integration] Heartbeat 오프라인 감지 통합 테스트"
labels: 'test, backend, priority:high, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-017] HB-003/HB-004 오프라인 감지 → EMAIL-003/OPS-002 알림 전송 통합 테스트
- 목적: Heartbeat 오프라인 감지 Cron(HB-003)이 `lastSeenAt` 기준 임계값 초과 시 오프라인 판정하고, EMAIL-003(보호자 이메일) + OPS-002(Slack 알림)를 연계 호출하는 전체 흐름을 통합 테스트로 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_HB-003.md`](./TASK_HB-003.md) — 오프라인 감지 Cron
- 테스트 대상: [`/TASKs/TASK_HB-004.md`](./TASK_HB-004.md) — 오프라인 상태 업데이트
- 테스트 대상: [`/TASKs/TASK_EMAIL-003.md`](./TASK_EMAIL-003.md) — 오프라인 알림 이메일
- 테스트 대상: [`/TASKs/TASK_OPS-002.md`](./TASK_OPS-002.md) — Slack 오프라인 비율 알림
- SRS 섹션: §5.4.3 Offline Detection, §5.8.2 Offline Alert
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-017

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/integration/heartbeat-offline.test.ts`
- [ ] Prisma Mock + Resend Mock + fetch Mock (Slack Webhook)
- [ ] `jest.useFakeTimers()` — 시간 경과 시뮬레이션
- [ ] 테스트 케이스:
  - `lastSeenAt`이 임계값(15분) 이내 → 정상, 알림 없음
  - `lastSeenAt`이 15분 초과 → 오프라인 판정, `status: "OFFLINE"` 업데이트
  - 오프라인 판정 → Resend 이메일 발송 1회 확인
  - OFFLINE 비율 ≥ 10% → Slack Webhook 호출 확인
  - OFFLINE 비율 < 10% → Slack 미호출
  - 중복 알림 방지 — 15분 이내 동일 device 재알림 없음

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 오프라인 감지 → 상태 업데이트**
- Given: `lastSeenAt`이 현재 -20분인 SensorDevice
- When: 오프라인 감지 Cron 실행
- Then: `sensorDevice.status: "OFFLINE"` 업데이트

**Scenario 2: 오프라인 → 이메일 + Slack 연계**
- Given: 오프라인 디바이스 1개 (전체 10개 중 1개)
- When: 오프라인 감지 Cron 실행
- Then: Resend 1회 호출, OFFLINE 비율 10% → Slack Webhook 호출

**Scenario 3: 중복 알림 방지**
- Given: 10분 전 오프라인 알림 발송 완료
- When: Cron 재실행
- Then: Resend 추가 호출 없음

## :gear: Technical & Non-Functional Constraints
- **통합 범위:** HB-003 → HB-004 → EMAIL-003 → OPS-002 전체 흐름
- **외부 Mock:** Resend, Slack Webhook, Prisma 모두 Mock
- **시간 조작:** `jest.useFakeTimers()` + `jest.setSystemTime()`

## :checkered_flag: Definition of Done (DoD)
- [ ] 오프라인 감지/이메일/Slack/중복방지 케이스 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] 외부 API 실제 호출 없음 확인?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_HB-003, TASK_HB-004, TASK_EMAIL-003, TASK_OPS-002
- **Blocks:** 없음
