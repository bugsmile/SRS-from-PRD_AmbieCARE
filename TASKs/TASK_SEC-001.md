---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Security] SEC-001: TLS 1.3 Vercel 인증서 검증"
labels: 'security, infra, priority:high, phase:0'
assignees: ''
---

## :dart: Summary
- 기능명: [SEC-001] Vercel 배포 환경 TLS 1.3 인증서 자동 프로비저닝 검증
- 목적: Vercel Hobby 플랜에서 HTTPS가 자동으로 활성화되고 TLS 1.3 이상이 적용되었는지 확인하며, 민감 정보가 암호화 채널을 통해서만 전송되는지 보안 검증한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- 관련 인프라: [`/TASKs/TASK_INFRA-001.md`](./TASK_INFRA-001.md) — Vercel 배포 설정
- SRS 섹션: §7.1 Security Requirements, NFR-SEC-001
- 태스크 리스트: [`/TASKs/SRS_V1_TASKS_list_OPUS.md`](./SRS_V1_TASKS_list_OPUS.md) — SEC-001

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] Vercel 배포 후 `https://` 접속 확인 — HTTP 리다이렉트 자동 적용 확인
- [ ] SSL Labs 또는 `openssl s_client` 명령으로 TLS 버전 확인: `openssl s_client -connect <domain>:443 -tls1_3`
- [ ] Vercel Dashboard → Domains 탭 → SSL 인증서 만료일 확인
- [ ] `next.config.js` — `headers()` 설정으로 HSTS 헤더 추가:
  ```
  Strict-Transport-Security: max-age=31536000; includeSubDomains
  ```
- [ ] API 엔드포인트 `http://` 접근 시 자동 `https://` 리다이렉트 확인
- [ ] 환경변수 (`DATABASE_URL`, `NEXTAUTH_SECRET` 등) Vercel 대시보드 암호화 저장 확인

## :test_tube: Acceptance Criteria (BDD/GWT)
**Scenario 1: HTTPS 자동 활성화**
- Given: Vercel에 배포된 프로덕션 URL
- When: `http://` 접근
- Then: 301/302 → `https://` 리다이렉트

**Scenario 2: TLS 1.3 적용**
- Given: 프로덕션 도메인
- When: `openssl s_client -tls1_3` 연결
- Then: TLS 1.3 핸드셰이크 성공

**Scenario 3: HSTS 헤더 존재**
- Given: HTTPS 응답
- When: 응답 헤더 확인
- Then: `Strict-Transport-Security` 헤더 포함

## :gear: Technical & Non-Functional Constraints
- **Vercel 자동 관리:** Let's Encrypt 인증서 자동 갱신 (만료 전 30일)
- **HSTS max-age:** 최소 1년 (31536000초)
- **환경변수 보안:** Vercel Dashboard에서만 관리, `.env` 파일 git 제외

## :checkered_flag: Definition of Done (DoD)
- [ ] HTTPS 강제 리다이렉트 동작 확인?
- [ ] TLS 1.3 적용 검증?
- [ ] HSTS 헤더 설정 완료?
- [ ] 환경변수 `.env` 파일 `.gitignore` 포함 확인?

## :construction: Dependencies & Blockers
- **Depends on:** TASK_INFRA-001 (Vercel 배포 완료)
- **Blocks:** 없음
