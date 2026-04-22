---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Performance] PERF-003: Vercel Analytics 응답 시간 모니터링"
labels: 'performance, infra, priority:low, phase:2'
assignees: ''
---

## :dart: Summary
- 기능명: [PERF-003] Vercel Analytics + Speed Insights 실사용자 응답 시간 모니터링 설정
- 목적: ENH-005에서 활성화한 Vercel Analytics를 활용해 실사용자(RUM) 기준 Core Web Vitals 및 API 응답 시간을 지속적으로 모니터링하고 성능 회귀를 조기에 감지한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 관련 태스크: [`/TASKs/TASK_ENH-005.md`](./TASK_ENH-005.md) — Vercel Analytics 활성화
- 관련 인프라: [`/TASKs/TASK_INFRA-001.md`](./TASK_INFRA-001.md) — Vercel 배포 설정
- SRS 섹션: §8.3 Observability, NFR-PERF-003
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — PERF-003

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Vercel Dashboard → Analytics 탭 활성화 확인
- [ ] `@vercel/analytics` + `@vercel/speed-insights` 패키지 설치 및 `layout.tsx` 주입 확인 (ENH-005 완료 전제)
- [ ] 모니터링 대상 지표 정의:
  - **LCP** (Largest Contentful Paint) ≤ 2,500ms
  - **FID/INP** (First Input Delay) ≤ 200ms
  - **CLS** (Cumulative Layout Shift) ≤ 0.1
  - **TTFB** (Time to First Byte) ≤ 600ms
- [ ] Vercel Analytics Dashboard에서 실사용자 데이터 수집 확인 (배포 후 24시간)
- [ ] 성능 회귀 알림 설정 — Vercel 이메일 알림 또는 Slack Webhook 연동
- [ ] 주간 성능 리포트 확인 프로세스 정의

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: Analytics 데이터 수집 확인**
- Given: Vercel Analytics 활성화 + 실사용자 방문
- When: Vercel Analytics Dashboard 확인
- Then: LCP, FID, CLS, TTFB 데이터 수집 중

**Scenario 2: Core Web Vitals 목표 달성**
- Given: 프로덕션 배포 후 24시간 데이터
- When: Analytics Dashboard 지표 확인
- Then: LCP ≤ 2,500ms, CLS ≤ 0.1, TTFB ≤ 600ms

**Scenario 3: 성능 회귀 감지**
- Given: 배포 후 LCP 급등 (3,000ms 초과)
- When: Vercel Analytics 이상 감지
- Then: 알림 발송 또는 대시보드 경고 표시

## :gear: Technical & Non-Functional Constraints
- **Vercel Hobby 무료:** Analytics 기본 제공 (유료 기능 제외)
- **데이터 보존:** Vercel Analytics 90일 보관
- **RUM vs 합성 테스트:** 이 태스크는 실사용자 기반 (PERF-001은 합성 테스트)

## :checkered_flag: Definition of Done (DoD)
- [ ] Vercel Analytics Dashboard에서 Core Web Vitals 데이터 수집 확인?
- [ ] LCP/FID/CLS 목표값 달성?
- [ ] 성능 회귀 알림 설정 완료?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_ENH-005, TASK_INFRA-001
- **Blocks:** 없음
