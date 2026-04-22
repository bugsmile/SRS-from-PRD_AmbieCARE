---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-012: [Unit] 이상치 필터링 테스트"
labels: 'test, backend, priority:medium, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-012] PIPE-003 이상치(Anomaly) 감지 필터링 로직 단위 테스트
- 목적: PIPE-003에서 구현한 이상치 감지 로직이 confidenceScore 임계값, 연속 이상 이벤트 패턴, false alarm 필터링을 올바르게 처리하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_PIPE-003.md`](./TASK_PIPE-003.md) — 이상치 감지 로직
- SRS 섹션: §5.5.3 Anomaly Detection Algorithm
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-012

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/pipe/anomalyFilter.test.ts`
- [ ] 순수 함수 `detectAnomalies(events: WellnessEvent[]): AnomalyResult` 단위 테스트
- [ ] 테스트 케이스:
  - `confidenceScore < 0.7` 이벤트 필터링 (노이즈로 제거)
  - 연속 3개 이상 ALERT 이벤트 → `anomalyFlag: true` 판정
  - 단일 ALERT 이벤트 → `anomalyFlag: false` (일시적 노이즈)
  - false alarm 처리된 과거 이벤트 → 이상치 집계에서 제외
  - 이상치 없는 정상 이벤트 배열 → `anomalyFlag: false`

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 낮은 confidenceScore 필터링**
- Given: `confidenceScore: 0.5`인 ALERT 이벤트 다수
- When: `detectAnomalies(events)` 호출
- Then: 해당 이벤트 무시, `anomalyFlag: false`

**Scenario 2: 연속 ALERT → 이상치 감지**
- Given: confidenceScore ≥ 0.7인 ALERT 이벤트 3개 연속
- When: `detectAnomalies(events)` 호출
- Then: `anomalyFlag: true`, 이상 이벤트 목록 반환

**Scenario 3: false alarm 제외**
- Given: `falseAlarm: true` 처리된 이벤트 포함
- When: `detectAnomalies(events)` 호출
- Then: 해당 이벤트 집계 제외

## :gear: Technical & Non-Functional Constraints
- **임계값:** confidenceScore 0.7, 연속 횟수 3회 (SRS 기준)
- **순수 함수 테스트:** 외부 의존 없음
- **false alarm 데이터:** UserFeedback 관계 Mock 포함

## :checkered_flag: Definition of Done (DoD)
- [ ] confidenceScore 필터/연속 감지/false alarm 제외 케이스 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] 경계값(2회 vs 3회 연속) 케이스 포함?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_PIPE-003
- **Blocks:** 없음
