---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Feature] EVT-002: createWellnessEvent Server Action 구현"
labels: 'feature, server-action, priority:critical, phase:0'
assignees: ''
---

## :dart: Summary
- 기능명: [EVT-002] `app/actions/events.ts` — `createWellnessEvent` Server Action 구현
- 목적: API-008 인터페이스 명세를 바탕으로, Prisma Client를 이용해 `WellnessEvent` 레코드를 생성하고 DB에 영속화하는 비즈니스 로직을 구현한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- API Spec: [`/TASKs/TASK_API-008.md`](./TASK_API-008.md) — Server Action I/O 타입
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#§3.3.1`](../SRS-Draft/SRS_V03(ENG_OPUS).md) — SA #1
- SRS 문서: [`/SRS-Draft/SRS_V03(ENG_OPUS).md#§6.2.2`](../SRS-Draft/SRS_V03(ENG_OPUS).md) — WellnessEvent 모델
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — EVT-002

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `app/actions/events.ts` 파일 생성 및 `'use server'` 디렉티브 추가
- [ ] API-008의 `CreateWellnessEventInput`, `CreateWellnessEventOutput` 인터페이스 임포트/선언
- [ ] Prisma Client (`lib/prisma`) 임포트
- [ ] `createWellnessEvent` 함수 구현:
  - `prisma.wellnessEvent.create({ data: { ... } })` 실행
  - `deviceId`가 `SensorDevice` 테이블에 존재하는지(FK 무결성) 예외 처리
- [ ] `try-catch` 블록으로 에러 감싸기 및 `{ success: false, error: string }` 포맷 반환
- [ ] `EMERGENCY` 타입 이벤트인 경우, 옵셔널하게 Resend Email/Slack 알림 호출(Phase 2에서 PIPE-001 등에 통합될 수 있으나, 즉각 알림은 여기서 고려 가능. MVP는 Report 기반이므로 즉시 알림은 연기 가능 - SRS §12.1 GAP-09 참조). 여기서는 저장 로직에 집중.

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 정상적인 DB Insert**
- Given: 유효한 Input 객체
- When: `createWellnessEvent` 호출
- Then: DB에 레코드가 생성되고 `success: true`가 반환된다.

**Scenario 2: 존재하지 않는 디바이스 오류**
- Given: DB에 없는 `deviceId`
- When: `createWellnessEvent` 호출
- Then: Prisma 에러가 catch되고, `success: false`와 함께 에러 메시지가 반환된다.

## :gear: Technical & Non-Functional Constraints
- **Server Action 제약:** 클라이언트에서 직접 호출될 수 있는 구조이므로, 직렬화 가능한 객체만 반환해야 한다 (에러 객체 직접 반환 지양).
- **Phase:** Phase 0

## :checkered_flag: Definition of Done (DoD)
- [ ] `'use server'` 디렉티브가 선언되었는가?
- [ ] Prisma를 통해 WellnessEvent가 정상적으로 Insert 되는가?
- [ ] 예외가 `{ success: false, error: string }` 형식으로 처리되는가?

## :construction: Dependencies & Blockers
- **Depends on:** DB-007 (Prisma 연동), API-008 (타입 명세)
- **Blocks:** EVT-001 (호출자)
