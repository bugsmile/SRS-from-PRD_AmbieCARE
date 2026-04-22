---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-011: [Unit] 데이터 부족 처리 테스트"
labels: 'test, backend, priority:medium, phase:1'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-011] PIPE-005 데이터 부족 처리 로직 단위 테스트
- 목적: PIPE-005에서 구현한 데이터 부족 감지 로직이 하루 WellnessEvent 수가 임계값 미만일 때 보고서 생성을 건너뛰고 적절한 상태를 반환하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_PIPE-005.md`](./TASK_PIPE-005.md) — 데이터 부족 처리
- SRS 섹션: §5.5.5 Insufficient Data Handling, NFR-QUALITY-002
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-011

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/pipe/insufficientData.test.ts`
- [ ] 순수 함수 `checkDataSufficiency(events: WellnessEvent[]): SufficiencyResult` 단위 테스트
- [ ] 테스트 케이스:
  - 이벤트 수 < 임계값(예: 144개 = 12시간 분량) → `{ sufficient: false, reason: "INSUFFICIENT_DATA" }` 반환
  - 이벤트 수 ≥ 임계값 → `{ sufficient: true }` 반환
  - 빈 배열 → `{ sufficient: false, reason: "NO_DATA" }` 반환
  - 보고서 생성 파이프라인에서 insufficient 시 DB 저장 스킵 확인
  - insufficient 상태 DailyReport에 `dataQuality: "INSUFFICIENT"` 플래그 저장 확인

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 데이터 부족 → 처리 스킵**
- Given: WellnessEvent 50개 (임계값 144 미만)
- When: `checkDataSufficiency(events)` 호출
- Then: `{ sufficient: false, reason: "INSUFFICIENT_DATA" }` 반환

**Scenario 2: 충분한 데이터 → 정상 처리**
- Given: WellnessEvent 288개
- When: `checkDataSufficiency(events)` 호출
- Then: `{ sufficient: true }` 반환

**Scenario 3: 파이프라인 스킵 확인**
- Given: insufficient 결과
- When: 보고서 생성 파이프라인 실행
- Then: `prisma.dailyReport.create` 미호출

## :gear: Technical & Non-Functional Constraints
- **임계값:** SRS 명세 기준 (최소 50% 데이터 = 144개/day)
- **순수 함수 테스트:** 외부 의존 없음
- **파이프라인 테스트:** Prisma Mock으로 create 호출 횟수 검증

## :checkered_flag: Definition of Done (DoD)
- [ ] 부족/충분/빈배열/파이프라인 스킵 케이스 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] Prisma create 미호출 검증?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_PIPE-005
- **Blocks:** 없음
