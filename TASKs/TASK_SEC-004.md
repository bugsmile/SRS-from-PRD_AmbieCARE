---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Security] SEC-004: RBAC 미들웨어 역할 기반 접근 제어 (Phase 2)"
labels: 'security, backend, priority:high, phase:2'
assignees: ''
---

## :dart: Summary
- 기능명: [SEC-004] Next.js 미들웨어 RBAC — GUARDIAN/ADMIN 역할 기반 라우트 접근 제어
- 목적: Phase 2에서 GUARDIAN(보호자)과 ADMIN(병원 관리자) 두 역할을 도입하고, Next.js Middleware에서 JWT 역할 클레임 검증으로 인가되지 않은 라우트 접근을 차단한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 관련 인증: [`/TASKs/TASK_AUTH-002.md`](./TASK_AUTH-002.md) — JWT 역할 클레임
- 관련 인증: [`/TASKs/TASK_AUTH-003.md`](./TASK_AUTH-003.md) — 미들웨어 보호
- SRS 섹션: §4.3 Authorization, §7.4 RBAC, NFR-SEC-004
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — SEC-004

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] `middleware.ts` RBAC 로직 구현:
  ```typescript
  // 라우트 → 허용 역할 매핑
  const ROUTE_ROLES: Record<string, Role[]> = {
    '/dashboard': ['ADMIN'],
    '/reports': ['GUARDIAN', 'ADMIN'],
    '/admin': ['ADMIN'],
  }
  ```
- [ ] JWT `getToken()` → `token.role` 추출 → 허용 역할 확인
- [ ] 인가 실패 시: 미인증 → `/login` 리다이렉트, 역할 불일치 → `/403` 또는 `NextResponse.json({ error: 'Forbidden' }, { status: 403 })`
- [ ] GUARDIAN이 `/dashboard` 접근 시 → 403
- [ ] ADMIN이 `/reports` 접근 시 → 허용
- [ ] 공개 경로 (`/login`, `/api/events/ingest`) → 미들웨어 bypass
- [ ] `middleware.ts` `matcher` 설정으로 정적 파일 제외

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: GUARDIAN → /dashboard 접근 차단**
- Given: JWT `role: "GUARDIAN"`
- When: `/dashboard` 접근
- Then: 403 Forbidden 또는 `/403` 리다이렉트

**Scenario 2: ADMIN → /dashboard 접근 허용**
- Given: JWT `role: "ADMIN"`
- When: `/dashboard` 접근
- Then: 200 OK, 페이지 렌더링

**Scenario 3: 미인증 → /login 리다이렉트**
- Given: JWT 없음
- When: 보호 경로 접근
- Then: `/login?callbackUrl=...` 리다이렉트

## :gear: Technical & Non-Functional Constraints
- **미들웨어 위치:** `src/middleware.ts` (Next.js App Router 규격)
- **JWT 추출:** `next-auth/jwt` `getToken()` 사용
- **성능:** 미들웨어 실행 < 5ms (Edge Runtime)
- **공개 경로:** `/login`, `/api/events/ingest`, `/api/heartbeat`, `/_next/` 제외

## :checkered_flag: Definition of Done (DoD)
- [ ] GUARDIAN/ADMIN 역할별 라우트 접근 제어 동작?
- [ ] 미인증 리다이렉트, 역할 불일치 403 처리?
- [ ] 공개 경로 bypass 동작?
- [ ] TEST-008 인증 테스트와 연계 확인?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_AUTH-002, TASK_AUTH-003
- **Blocks:** 없음
