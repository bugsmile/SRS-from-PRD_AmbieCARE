---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-013: [Unit] 이상 징후 감지 + AI 설명 호출 테스트"
labels: 'test, backend, priority:high, phase:2'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-013] PIPE-004 이상 징후 감지 시 AI-004 Gemini 설명 생성 호출 테스트
- 목적: PIPE-004에서 이상 징후 감지 시 AI-004(Gemini 이상 설명 생성)를 올바르게 호출하고, Gemini 응답이 DailyReport에 저장되는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_PIPE-004.md`](./TASK_PIPE-004.md) — 이상 징후 감지 파이프라인
- 테스트 대상: [`/TASKs/TASK_AI-004.md`](./TASK_AI-004.md) — Gemini 이상 설명 생성
- SRS 섹션: §5.5.4 Anomaly Explanation, §5.6 AI Integration
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-013

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/pipe/anomalyExplanation.test.ts`
- [ ] `@ai-sdk/google` generateText Mock 설정
- [ ] Prisma Mock 설정
- [ ] 테스트 케이스:
  - `anomalyFlag: true`인 DailyReport → Gemini `generateText` 1회 호출 확인
  - `anomalyFlag: false` → Gemini 호출 없음 확인
  - Gemini 응답 텍스트가 `DailyReport.anomalyExplanation` 필드에 저장 확인
  - Gemini API 오류 시 fallback 메시지 저장 확인 (AI-003 fallback 적용)
  - 프롬프트에 디바이스 ID + 이벤트 요약 포함 확인

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 이상 징후 → Gemini 호출**
- Given: `anomalyFlag: true`, 이상 이벤트 목록 존재
- When: 파이프라인 실행
- Then: `generateText` 1회 호출, 프롬프트에 이벤트 데이터 포함

**Scenario 2: 정상 → Gemini 미호출**
- Given: `anomalyFlag: false`
- When: 파이프라인 실행
- Then: `generateText` 호출 없음

**Scenario 3: Gemini 오류 → fallback**
- Given: `generateText` throws Error
- When: 파이프라인 실행
- Then: `anomalyExplanation: "AI 분석을 일시적으로 사용할 수 없습니다"` 저장

## :gear: Technical & Non-Functional Constraints
- **AI Mock:** `jest.mock('@ai-sdk/google')` — 실제 API 호출 없음
- **Prisma Mock:** `dailyReport.update` 호출 검증
- **fallback 메시지:** AI-003 명세와 동일 문자열 사용

## :checkered_flag: Definition of Done (DoD)
- [ ] 이상 감지/정상/오류 fallback 케이스 모두 구현?
- [ ] 모든 테스트 GREEN?
- [ ] Gemini API 미호출 케이스 확인?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_PIPE-004, TASK_AI-004, TASK_AI-003
- **Blocks:** 없음
