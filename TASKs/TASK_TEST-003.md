---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-003: [Unit] Mock Seed 데이터 생성 검증 테스트"
labels: 'test, backend, priority:medium, phase:0'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-003] Mock Seed 스크립트 데이터 생성 무결성 단위 테스트
- 목적: MOCK-001~003에서 정의한 seed 스크립트가 SensorDevice 3개, WellnessEvent 288개/device/day, UserAccount + UserDevice 관계를 올바르게 생성하는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 테스트 대상: [`/TASKs/TASK_MOCK-001.md`](./TASK_MOCK-001.md) — SensorDevice seed
- 테스트 대상: [`/TASKs/TASK_MOCK-002.md`](./TASK_MOCK-002.md) — WellnessEvent seed
- 테스트 대상: [`/TASKs/TASK_MOCK-003.md`](./TASK_MOCK-003.md) — UserAccount + UserDevice seed
- SRS 섹션: §6.3 Mock Data Specification
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-003

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/db/seed.test.ts`
- [ ] `beforeAll`: 테스트 DB에 seed 스크립트 실행
- [ ] `afterAll`: 테스트 데이터 전체 정리 (`prisma.wellnessEvent.deleteMany()` 등)
- [ ] 테스트 케이스:
  - SensorDevice 레코드가 정확히 3개 생성되었는지 확인
  - 각 device별 WellnessEvent 288개 (5분 간격 × 24h) 생성 확인
  - WellnessEvent의 `eventType` 분포 — NORMAL 비율 ≥ 85%
  - UserAccount 생성 및 `email` 형식 유효성 확인
  - UserDevice 관계 레코드 존재 확인 (userId + deviceId 참조)
  - Seed 스크립트 중복 실행 시 에러 없이 idempotent하게 처리

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: SensorDevice 3개 생성**
- Given: 빈 테스트 DB
- When: seed 스크립트 실행
- Then: `prisma.sensorDevice.count()` === 3

**Scenario 2: WellnessEvent 288개/device**
- Given: seed 스크립트 완료
- When: 특정 deviceId로 WellnessEvent 조회
- Then: count === 288, timestamp 간격 5분

**Scenario 3: idempotent 재실행**
- Given: seed 스크립트 1회 완료
- When: seed 스크립트 재실행
- Then: 에러 없이 완료, 중복 레코드 없음

## :gear: Technical & Non-Functional Constraints
- **테스트 DB:** SQLite in-memory 또는 격리된 Supabase 테스트 프로젝트
- **실행 속도:** seed 포함 60초 이내 완료
- **데이터 격리:** 프로덕션 DB에 영향 없음

## :checkered_flag: Definition of Done (DoD)
- [ ] SensorDevice, WellnessEvent, UserAccount, UserDevice 4개 모델 seed 검증 완료?
- [ ] 모든 테스트 GREEN?
- [ ] idempotent 재실행 확인?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_MOCK-001, TASK_MOCK-002, TASK_MOCK-003
- **Blocks:** 없음
