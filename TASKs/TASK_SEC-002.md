---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Security] SEC-002: Prisma 스키마 PII 필드 0건 검증"
labels: 'security, backend, priority:high, phase:0'
assignees: ''
---

## :dart: Summary
- 기능명: [SEC-002] Prisma 스키마 및 API 응답에서 PII(개인식별정보) 노출 0건 검증
- 목적: Supabase DB 스키마와 API 응답에 주민등록번호, 실명, 전화번호 등 SRS에서 금지된 PII 필드가 포함되지 않았음을 정적 분석 및 코드 리뷰로 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 관련 스키마: [`/TASKs/TASK_DB-001.md`](./TASK_DB-001.md) ~ [`/TASKs/TASK_DB-005.md`](./TASK_DB-005.md) — Prisma 모델
- SRS 섹션: §7.2 Privacy Requirements, NFR-SEC-002 (PII 최소화)
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — SEC-002

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Prisma 스키마 (`schema.prisma`) PII 필드 목록 검토:
  - 금지 필드: `name`, `phone`, `address`, `birthDate`, `ssn`, `nationalId`
  - 허용 필드: `email` (인증 목적, 암호화 저장), `id` (cuid — 비식별)
- [ ] API 응답 타입 검토 — `SELECT *` 대신 필요 필드만 select 확인
- [ ] GitHub 코드 검색: `grep -r "name\|phone\|address\|birthDate" prisma/schema.prisma`
- [ ] CI 린터 규칙 추가 (SEC-003 참조): PII 키워드 감지 시 빌드 실패
- [ ] Prisma `UserAccount` 모델: `email` 외 식별 정보 없음 확인
- [ ] API 응답에서 `email` 필드 마스킹 처리 (`u***@example.com`) 확인

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: 스키마 PII 필드 부재**
- Given: `prisma/schema.prisma` 파일
- When: PII 키워드(`name`, `phone`, `address`, `birthDate`) 검색
- Then: 0건 — 해당 필드명 없음

**Scenario 2: API 응답 email 마스킹**
- Given: 보호자 프로필 API 응답
- When: `email` 필드 포함 응답
- Then: `u***@domain.com` 형식으로 마스킹

**Scenario 3: SELECT 최소화**
- Given: Prisma 쿼리 코드
- When: `findMany` / `findUnique` 호출
- Then: `select: { id: true, ... }` 명시적 필드 선택, `SELECT *` 없음

## :gear: Technical & Non-Functional Constraints
- **GDPR/개인정보보호법:** email 암호화 저장 또는 마스킹 처리
- **데이터 최소화 원칙:** 서비스에 불필요한 개인정보 수집 금지
- **CI 자동화:** SEC-003 Linter와 연계하여 PR 시 자동 검사

## :checkered_flag: Definition of Done (DoD)
- [ ] Prisma 스키마 PII 필드 0건 확인?
- [ ] API 응답 email 마스킹 처리?
- [ ] CI PII 키워드 감지 규칙 추가?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_DB-001~005
- **Blocks:** SEC-003 (Linter 규칙 정의에 활용)
