---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-018: [Unit] Triage 정렬 테스트"
labels: 'test, frontend, priority:low, phase:3'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-018] ENH-003 Triage 엔진 정렬 로직 단위 테스트
- 목적: ENH-003에서 구현한 Triage 엔진이 위험도 점수 기준으로 디바이스를 올바르게 정렬하고, 동점 시 최근 이벤트 시간 순으로 처리하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_ENH-003.md`](./TASK_ENH-003.md) — Triage 엔진
- SRS 섹션: §5.1.4 Triage Sorting, Phase 3 Enhancement
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-018

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/triage/sort.test.ts`
- [ ] 순수 함수 `sortByTriage(devices: DeviceStatus[]): DeviceStatus[]` 단위 테스트
- [ ] 테스트 케이스:
  - 위험도 점수 내림차순 정렬 확인
  - 동점 시 `lastEventAt` 최신 순으로 정렬
  - OFFLINE 디바이스가 최상단에 위치
  - ALERT > WARNING > NORMAL 순서 보장
  - 빈 배열 입력 → 빈 배열 반환 (에러 없음)
  - 단일 항목 → 그대로 반환

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 위험도 점수 내림차순**
- Given: 점수 [30, 90, 60]인 디바이스 3개
- When: `sortByTriage(devices)` 호출
- Then: `[90, 60, 30]` 순서로 정렬

**Scenario 2: 동점 시 최신 이벤트 우선**
- Given: 동일 점수, `lastEventAt` 다른 2개 디바이스
- When: `sortByTriage(devices)` 호출
- Then: 더 최근 `lastEventAt`을 가진 디바이스가 앞

**Scenario 3: OFFLINE 최상단**
- Given: OFFLINE 1개, ALERT 2개 혼재
- When: `sortByTriage(devices)` 호출
- Then: OFFLINE 디바이스가 index 0

## :gear: Technical & Non-Functional Constraints
- **순수 함수 테스트:** 외부 의존 없음, 입력→출력만 검증
- **정렬 안정성:** 동일 점수/시간 시 원래 순서 유지 (stable sort)
- **타입:** `DeviceStatus` 인터페이스 준수

## :checkered_flag: Definition of Done (DoD)
- [ ] 점수 정렬/동점 처리/OFFLINE 우선/엣지케이스 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] stable sort 동작 확인?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_ENH-003
- **Blocks:** 없음
