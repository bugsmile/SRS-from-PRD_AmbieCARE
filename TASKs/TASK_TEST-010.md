---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-010: [Unit] sleepScore 정확도 테스트 (오차율 < 10%)"
labels: 'test, backend, priority:high, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-010] PIPE-002 sleepScore 계산 정확도 단위 테스트
- 목적: PIPE-002에서 구현한 sleepScore 계산 로직이 알려진 WellnessEvent 입력 데이터에 대해 기대값 대비 오차율 10% 미만으로 수면 점수를 계산하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_PIPE-002.md`](./TASK_PIPE-002.md) — sleepScore 계산 로직
- SRS 섹션: §5.5.2 Sleep Score Algorithm, NFR-QUALITY-001
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-010

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/pipe/sleepScore.test.ts`
- [ ] 순수 함수 `calculateSleepScore(events: WellnessEvent[]): number` 단위 테스트
- [ ] 픽스처 데이터 준비: 알려진 입력 → 기대 출력 쌍 5개 이상
- [ ] 테스트 케이스:
  - 8시간 연속 NORMAL 이벤트 → 점수 ≥ 85 (우수 수면)
  - BATHROOM 3회 이하 → 점수 패널티 최소화 확인
  - BATHROOM 5회 이상 → 점수 패널티 적용 확인
  - 야간 시간대(22:00~06:00) MOVEMENT 없음 → 점수 가중치 확인
  - 오차율 계산: `|실제값 - 기대값| / 기대값 < 0.10`
  - 빈 이벤트 배열 → 점수 0 또는 null 반환

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 이상적 수면 패턴 → 높은 점수**
- Given: 22:00~06:00 연속 NORMAL 이벤트, BATHROOM 2회
- When: `calculateSleepScore(events)` 호출
- Then: 반환값 ≥ 85, 기대값 대비 오차 < 10%

**Scenario 2: 빈번한 야간 활동 → 낮은 점수**
- Given: 야간 MOVEMENT 이벤트 다수, BATHROOM 6회
- When: `calculateSleepScore(events)` 호출
- Then: 반환값 ≤ 50

**Scenario 3: 오차율 허용 범위**
- Given: 기대값 70인 시나리오 입력
- When: 함수 실행
- Then: 반환값이 63~77 범위 내 (±10%)

## :gear: Technical & Non-Functional Constraints
- **순수 함수 테스트:** DB/외부 의존 없음, 입력→출력만 검증
- **픽스처:** `__fixtures__/sleepEvents.ts` 파일에 테스트 데이터 분리
- **오차 기준:** SRS NFR-QUALITY-001 — 10% 이내

## :checkered_flag: Definition of Done (DoD)
- [ ] 5개 이상 픽스처 케이스로 정확도 검증?
- [ ] 오차율 < 10% 모든 케이스 GREEN?
- [ ] 엣지 케이스(빈 배열, 전체 ALERT) 처리 확인?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_PIPE-002
- **Blocks:** 없음
