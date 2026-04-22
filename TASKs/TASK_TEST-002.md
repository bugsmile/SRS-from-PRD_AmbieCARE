---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Test] TEST-002: [Unit] Prisma 스키마 무결성 테스트"
labels: 'test, backend, priority:medium, phase:0'
assignees: ''
---

## :dart: Summary
- 기능명: [TEST-002] Prisma 스키마 5개 모델 무결성 단위 테스트
- 목적: DB-001~005에서 정의한 5개 Prisma 모델(SensorDevice, WellnessEvent, UserAccount, UserDevice, DailyReport)의 PK 생성, FK 관계, unique 제약, 인덱스가 올바르게 적용되었는지 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 섹션: §3.6 ERD, §6.2.1~6.2.5
- 테스트 대상: [`/TASKs/TASK_DB-001.md`](./TASK_DB-001.md) ~ [`/TASKs/TASK_DB-005.md`](./TASK_DB-005.md)
- 마이그레이션: [`/TASKs/TASK_DB-007.md`](./TASK_DB-007.md)
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — TEST-002

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] 테스트 파일: `__tests__/db/schema.test.ts`
- [ ] Prisma 테스트 환경 설정 — `DATABASE_URL` 테스트 DB (SQLite in-memory 또는 별도 Supabase 테스트 프로젝트)
- [ ] `beforeAll`: `prisma.$connect()` + 마이그레이션 적용
- [ ] `afterAll`: `prisma.$disconnect()` + 테스트 데이터 정리
- [ ] 테스트 케이스:
  - SensorDevice `cuid()` PK 자동 생성 검증
  - WellnessEvent `deviceId` FK → SensorDevice 참조 무결성 검증 (존재하지 않는 deviceId → 에러)
  - UserAccount `email` unique 제약 검증 (중복 insert → 에러)
  - UserDevice 복합 PK `(userId, deviceId)` 검증
  - WellnessEvent `timestamp` 인덱스 존재 확인
  - DailyReport `(deviceId, date)` unique 제약 검증

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: cuid PK 자동 생성**
- Given: id 미지정으로 SensorDevice insert
- When: `prisma.sensorDevice.create({ data: { ... } })`
- Then: `id`가 cuid 형식 문자열로 자동 생성된다.

**Scenario 2: FK 참조 무결성**
- Given: 존재하지 않는 `deviceId`로 WellnessEvent 생성 시도
- When: `prisma.wellnessEvent.create(...)`
- Then: Prisma FK 에러 발생.

**Scenario 3: UserAccount email unique**
- Given: 동일 email로 두 번째 UserAccount 생성 시도
- When: 두 번째 `prisma.userAccount.create(...)`
- Then: unique 제약 에러 발생.

## :gear: Technical & Non-Functional Constraints
- **테스트 DB:** SQLite in-memory (`datasource db { provider = "sqlite" }`) 또는 격리된 테스트 Supabase
- **실행 속도:** 스키마 테스트는 마이그레이션 포함 30초 이내 완료

## :checkered_flag: Definition of Done (DoD)
- [ ] 5개 모델 전체 제약 조건이 테스트되었는가?
- [ ] 모든 테스트 GREEN?
- [ ] CI에서 자동 실행 가능한가?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_DB-007 (마이그레이션 배포 완료)
- **Blocks:** 없음
