---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] EVT-001: POST /api/events/ingest/route.ts — API Route Handler 구현"
labels: 'feature, api, priority:critical, phase:0'
assignees: ''
---

## :dart: Summary
- 기능명: [EVT-001] POST `/api/events/ingest/route.ts` 구현
- 목적: API-001 명세를 바탕으로, Edge 디바이스(혹은 Mock)로부터 수신된 이벤트 요청을 검증하고 API Key 인증을 처리한 뒤 Server Action을 호출하여 DB에 저장하는 파이프라인의 진입점을 구현한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- API Spec: [`/TASKs/TASK_API-001.md`](./TASK_API-001.md) — DTO 및 에러 코드 명세
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#§3.3`](../SRS-Draft/SRS_V03(ENG_OPUS).md) — Route #1
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#§14.1`](../SRS-Draft/SRS_V03(ENG_OPUS).md) — `mock=true` 관련 사양
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — EVT-001

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `app/api/events/ingest/route.ts` 파일 생성
- [ ] Zod 스키마 구현 (`EventIngestRequest` 검증):
  - `eventType`: enum `["ACTIVITY_ALERT", "WELLNESS_SCORE", "EMERGENCY"]`
  - `confidenceScore`: `min(0).max(1)`
  - ISO 8601 타임스탬프 등
- [ ] API Key 검증 로직 구현:
  - Header의 `x-api-key` 또는 `Authorization: Bearer`와 `process.env.API_KEY` 비교
  - 실패 시 `401 Unauthorized` 반환
- [ ] MOCK-004 대응 `mock=true` 쿼리 파라미터 분기:
  - `mock=true`일 경우 Zod 검증 생략하고 임의 데이터 생성 후 응답
- [ ] 유효성 검사 실패 시 `400 Bad Request` 에러 상세 응답 구성
- [ ] 검증 성공 시 `createWellnessEvent` Server Action (EVT-002) 호출
- [ ] Server Action 성공 시 `201 Created`, 실패 시 적절한 에러 코드 반환

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 올바른 데이터 수신 및 DB 저장 호출**
- Given: 올바른 API Key와 Body 데이터
- When: `POST` 요청
- Then: `createWellnessEvent`가 호출되고 `201 Created`가 반환된다.

**Scenario 2: Zod 검증 오류 상세 응답**
- Given: `eventType` 필드 누락
- When: 요청
- Then: `400 Bad Request` 응답에 "eventType is required" 관련 정보가 포함된다.

**Scenario 3: API Key 미일치**
- Given: 잘못된 API Key
- When: 요청
- Then: `401 Unauthorized` 반환.

## :gear: Technical & Non-Functional Constraints
- **Validation:** 런타임 타입 안전성을 위해 Zod 라이브러리 사용 권장
- **Timeout:** 10초 이내 응답 (Serverless 제한)
- **보안:** 금지어 검증 포함 (API-001 명세 참조)
- **Phase:** Phase 0

## :checkered_flag: Definition of Done (DoD)
- [ ] `route.ts`가 정상 작동하며 Zod 검증이 적용되었는가?
- [ ] API Key 인증이 올바르게 동작하는가?
- [ ] `mock=true` 분기가 포함되었는가?
- [ ] `createWellnessEvent` Server Action 연동이 완료되었는가?

## :construction: Dependencies & Blockers
- **Depends on:** API-001 (명세), EVT-002 (Server Action 구현)
- **Blocks:** 실제 이벤트 수집 플로우
