# Software Requirements Specification (SRS)
Document ID: SRS-001
Revision: 1.0
Date: 2026-04-16
Standard: ISO/IEC/IEEE 29148:2018
기반문서: PRD_Rooted_V0.2.md

-------------------------------------------------
1. Introduction

   1.1 Purpose
   본 문서는 비접촉 앰비언트 케어 시장에서 B2B(요양시설), B2G(지자체), B2C(원격 보호자) 사용자가 공통으로 겪고 있는 기존 제품의 한계(오탐에 따른 알람 피로도 및 사망 사고 유발, 충전·착용의 마찰성, 프라이버시 침해)를 해결하기 위한 목적을 지닌 "Rooted — 비접촉 AI 앰비언트 홈 안전 솔루션"의 소프트웨어 요구사항(능력, 기능, 제약)을 완전하고 상세하게 정의한다.

   1.2 Scope (In-Scope / Out-of-Scope)
   이 시스템의 아키텍처와 제품 릴리스가 포괄하는 업무 범위와 시스템 개발 경계는 다음과 같다.
   - **In-Scope:**
     - UWB 레이더 HW 연동(비접촉 센서 모듈)을 통한 실내 앰비언트 인지 로직(동선/호흡/심박/체류 센싱 시스템 통제).
     - 오탐 제로화 AI 필터링 엔진 엣지 로직, 월간 오탐률 저하(월 0.3건 이하 통제 목표).
     - 어르신을 위한 0회 조작(Zero-Friction) 자동 데이터 인제스트 서비스 환경.
     - B2C 사용자를 위한 원격 보호자 앱(푸시 알림 처리 및 일간 데일리 리포트 제공).
     - B2B 사용자를 위한 요양시설 관제 대시보드(신호등 UI) 시스템.
     - 기존 의료 정보 전산망(EMR) 간의 외부 연동(Webhook API) 메커니즘.
   - **Out-of-Scope:**
     - 스마트홈(Home Assistant, 조명, 가전) 제어 및 연동 로직 (공간 제어는 루트의 핵심가치인 안전을 흐리므로 배제).
     - 제품 마케팅, 앱 UI 메시지 내 '돌봄', '케어', '노인용' 과 같은 타겟 층 특정적 마케팅 언어 노출 통제 로직(낙인방지 기능).
     - B2G 입찰 맞춤형 조달 단가 축소 스펙 모델 및 정부 기관 관제 SLA 맞춤 시스템 확립 (출시 지연 방지를 위해 제외).

   1.3 Definitions, Acronyms, Abbreviations
   - **JTBD (Jobs to be Done):** 사용자가 특정한 맥락 속에서 달성해야 하는 구체적인 과업(예: 아무 개입 없이 실 위급 상황 시에만 연락 받기).
   - **AOS (Adjusted Opportunity Score):** 기회 점수를 보정한 지표로써, 각 기능 및 통점의 우선 순위를 판별하는 정량 도구.
   - **DOS (Discovered Opportunity Score):** 도출된 핵심 기회 점수로 Rooted 제품에서는 '오탐률 제로화 엔진'의 달성이 3.8점(가장 높은 가치)으로 정의됨.
   - **Validator (검증기/검증자):** UWB 레이더가 포착한 미세 움직임에서 노이즈(이불 움직임, 환풍기, 10kg 이하 반려동물)와 유효 이벤트(인간의 낙상, 호흡 등 특성)를 분리하여 판별하는 AI 필터의 엣지 로직 및 그 메커니즘.

   1.4 References (REF-XX)
   - **REF-01:** Rooted — 비접촉 AI 앰비언트 홈 안전 솔루션 PRD v0.2
   - **REF-02:** 시장 조사 및 KSF 논거(Porter's Five Forces 등) 증빙 지표 자원 파일 명세.
   
   1.5 Constraints and Assumptions
   *(ADR, Risk, Assumption을 통합하여 정의함)*
   - **Constraints (시스템 및 아키텍처 제약사항):**
     - 규제 우회(식약처 의료기기 인허가 면제 보장): 앱 UI, 데이터 모델, API 엔드포인트 명칭 등 전 구간에 걸쳐 'diagnosis', 'medical', 'patient' 단어는 전면 금지된다. 반드시 라이프케어 웰니스 목적으로 기술되어야 한다.
     - 개인정보보호법 및 비영상 처리 강제 보장: 99% 무결성 데이터라도 서버로 원시 레이더 파형 전송은 원천 금지되며, 반드시 엣지단에서 이진수, 수치 위주의 이벤트 결과값으로 변환(비식별 메타데이터) 후 전송해야 한다. B2B 관제 시에는 RBAC와 동의서가 선행되어야 시스템 열람이 허용된다.
     - 백엔드 종속성 방어: 1기 개발 MVP 버전 상에서는 어떠한 SI(개별 인테리어) 구축도 거절하며, 반드시 독립된 SaaS형 대시보드와 범용 1위 벤더 EMR만 수용하는 '표준 플러그인' 기반으로 동작해야 한다.
   - **Assumptions (사전 가정):**
     - NXP 및 Infineon 등 핵심 UWB 칩셋 부품 조달이 개발 로드맵 진행 중에 안정적으로 이어진다.
     - 기본 제공 패키지인 반경별 1실당 침실 1개, 화장실 1개의 하드웨어만으로도 비접촉 모니터링 요구사항 기능의 100% 충족에 무리가 없다.
     - 요양 병원 1위 그룹인 메인 EMR 벤더사의 데이터 수신 인터페이스 규격이 본 표준 API를 수용하는 형태의 파트너십으로 합의된다.

2. Stakeholders
   본 솔루션 이해관계자의 시스템 역할, 상호작용 및 주요 관심사 정의는 다음과 같다.
   - **박지수 (Core - 보호자/B2C):**
     - 역할/책임: 모바일 애플리케이션의 1차 운영자이며 서비스 구매 책임자.
     - 관심사: 시스템의 오탐 알림으로 인한 심야 시간대의 공포/피로 극복 여부, 충전 및 조작을 기피하는 부모님을 위한 제품 효용성, 주 단위 데일리 리포트 건강 징후 알림(오차율 < 10%).
   - **정민석 (Adjacent - 지자체 B2G 모니터링 담당):**
     - 역할/책임: 응급 장기안전안심 장비의 제품 교체 기획 관리자.
     - 관심사: 대규모 설치 환경의 오류 안정성과 기기 간 오탐 확률에 대한 명확한 데이터 증빙.
   - **장영희 (Extreme - 사망 사고 분쟁 요양병원 유가족):**
     - 역할/책임: 신규 요양원 선택 및 안전 기준 판별자.
     - 관심사: 직원의 오탐 무시 행위가 불가하도록 제한할 기술적 통제 장치 유무, 유사시 법적 조치가 가능한 최소 90일 치 이상의 로깅/데이터 무결성 시스템.
   - **고태식 (Non-user - 장치 거주 대상자):**
     - 역할/책임: 부모/시니어 사용자이나, 시스템에 능동적 개입이 의도적으로 없어야 하는 대상자.
     - 관심사: 레이더로 구동하는 비영상(카메라 없음) 장치를 통한 완벽한 사생활/프라이버시 침해 제로 체감. '환자', '사용자' 등의 조작 마찰(Friction) Zero 경험.
   - **설치 시설 관리자 (B2B - 실무 운영 요원):**
     - 역할/책임: 야간 모니터링 B2B 대시보드 당직 운영자.
     - 관심사: 직관적인 상태판별(신호등 UI 등), 외부 EMR 채널 자동연동을 통해 이중 기록 수기 입력 업무가 없어지는지 여부.

3. System Context and Interfaces

   3.1 External Systems
   - **외부 Facility EMR Network:** 요양 수용 시설별 고유의 Electronic Medical Record 환자 차트 관리 시스템. 해당 시스템은 본 솔루션 플랫폼에서 발생한 Event Webhook을 상시 POST 방식으로 통합 연계한다.
   - **Push/Notification Gateway (FCM/APNs):** 긴급 데이터 및 예약된 데일리 웰니스 패턴 리포트의 보호자 모바일 기기 푸시 발송을 담당하는 서드파티 통신 시스템.
   
   3.2 Client Applications
   - **B2C 보호자 모바일 애플리케이션:** iOS 기반 및 안드로이드 기반 앱 플랫폼. 웰니스 리포트 시각화 및 Alert PUSH 실시간 처리 지원(설정 임계 시간 이내 동작).
   - **B2B 관리 시설 웹 대시보드:** 병상과 관제 상태를 직관적인 UI 모듈(정상/경고/위험)로 렌더링하며 과거 이벤트를 열람하는 관리용 중앙 웹 SPA(Single Page Application).

   3.3 API Overview
   - **F-API-01: 외부 EMR Webhook 호출기:** B2B 공간의 응급, 수면/체류 로그 데이터를 Webhook POST 방식 송신. JSON Payload와 HMAC 기반 서명 인증 체계 포함.
   - **F-API-02: 알림 파견 엔진 API:** 외부 APNs/FCM으로 요청을 던지는 어댑터용 API. Daily 리포트 발송 트리거는 매일 오전 07:30 지정 스케줄러로 호출.
   - **I-API-01: 엣지 비식별화 인제스트 (내부):** 실내 Edge 센서가 5분 단위로 수치형 파형 데이터를 중앙 DB로 암호화(TLS 1.3)하여 Push.
   - **I-API-02: 90일 누적 보존 열람 조회 (내부):** 아카이빙 된 해시값 무결성 증명 보호자/관리자 열람 보장 RESTFul API.

   3.4 Interaction Sequences
        3.4.1 문서 자동 생성 시퀀스 (데일리 웰니스 리포트 자동 발행)
        ```mermaid
        sequenceDiagram
            participant Edge as Sensor Device (Edge)
            participant Cloud as Rooted Cloud Platform
            participant App as B2C Guardian App
            
            Edge->>Cloud: 5분 주기: 비식별 메타 이벤트 동기화 (I-API-01)
            Cloud-->>Cloud: 24시간 동안 사용자 활동 누적 집계 체류
            Cloud->>Cloud: 오전 07:00: 수면 시간 및 화장실 방문 빈도 리포트 오브젝트 빌드
            Cloud->>App: 오전 07:30: 푸시 알림 및 데일리 요약 발송 (F-API-02)
            App-->>Cloud: ACK 확인 및 UI 렌더링 준비
        ```

        3.4.2 검증기(Validator) 실행 시퀀스 (오탐 필터링 및 응급 이벤트 결단)
        ```mermaid
        sequenceDiagram
            participant Person as Target Person (in Room)
            participant Edge as Validator (AI Edge Filtering)
            participant Cloud as Core Real-time Engine
            participant App as Guardian / Dashboard
            
            Person->>Edge: 낙상(응급) / 반려동물 지나감(노이즈)
            Edge->>Edge: UWB 파형 딥러닝 패턴 대조 판별기 동작
            alt Weight/Pattern == Dog & Noise 
                Edge-->>Edge: "False Alarm" 검증 처리 (패킷 Drop / 무시)
            else Pattern == Human Fall (응급 확률 > 90%)
                Edge->>Cloud: 'fall_alert' 이벤트 Dispatch
                Cloud->>App: 긴급 PUSH 발송 (엔드투엔드 지연 측정 시작)
                App-->>Cloud: 알람 도달 (지연 시간 <= 2초 이내 달성)
            end
        ```

        3.4.3 PMF(Product-Market Fit) 진단 시퀀스 (고객 피드백/리텐션 감지 로직)
        ```mermaid
        sequenceDiagram
            participant App as Guardian B2C App
            participant DB as Analytics/Log DB
            participant Engine as PMF Monitor Engine
            
            App->>DB: `view_daily_report` 이벤트 발송 (유저 앱 진입)
            App->>DB: 사용자가 앱에서 수동으로 "오탐 신고" 제출 시 플래그 연동
            engine-->>DB: Cron 배치 주기적으로 PMF 상태 체크 (Weekly)
            Engine->>Engine: 오탐 집계 > 월 0.3건 이하 여부 검증
            Engine->>Engine: 보고서 주간 WAU > 60% 활성율 판정 
            Engine-->>엔지니어: Alerting 도는 Success Notification 라우팅
        ```

        3.4.4 노션/지라(Notion/Jira) 동기화 시퀀스 (B2B EMR/외부 관제 시스템 연동 장애 복구망)
        ```mermaid
        sequenceDiagram
            participant Cloud as Rooted Gateway
            participant EMR as Local Facility EMR Target
            participant Board as B2B Dashboard UI
            
            Cloud->>Cloud: `wellness_score` 또는 `alert` 데이터 생출
            Cloud->>EMR: POST Payload 전송 시도
            alt 성공 (HTTP 20X)
                EMR-->>Cloud: 이벤트 수신 확정
                Cloud->>Board: 정상 처리 상태 녹색 LED 업데이트
            else EMR 서버 다운 장애 (HTTP 500 등)
                EMR-->>Cloud: 수신 실패 또는 Timeout
                Cloud->>Cloud: 지수 백오프 기반 최대 3회 재전송 시도 (Exponential Backoff)
                Cloud-->>Board: 연동 실패 알림 및, 대시보드 모듈상 Urgent Warning 적색 점멸 처리
            end
        ```

4. Specific Requirements

   4.1 Functional Requirements (기능 요구사항)

| Requirement ID | Priority | Source Story | Requirement Description | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| REQ-FUNC-011 | Must | Story 1 | **반려동물 및 일상 노이즈 여과 필터:** Validator 엔진은 반경 내의 대상 물체가 10kg 이하 반려동물인지 단순 1회성 이불 뒤척임인지 사람의 응급 요소인지 즉시 구분한다. | AC-1.4: 사람과 다른 행위 패턴 검출 시 알림 발생 정확도 99% 무시 방어 보장. AC-1.1: 시스템은 정기 리포트 오탐 빈도를 검수 시 0.3건/월 이하로 누적되게 필터링해야 한다. |
| REQ-FUNC-012 | Must | Story 1 | **응급 낙상/징후 상황 실시간 푸시 발송:** Validator 엔진이 위급 패턴(5분 이상 약화 등)을 90% 신뢰도로 검증할 시, 즉각 로컬 장비 개입이나 버튼 없이도 앱 시스템 네트워크로 긴급 Push 이벤트를 발생시킨다. | AC-1.3: 감지 알고리즘에서 서버를 거쳐 클라이언트가 알림을 렌더링하기까지 60초 내 발송 프로토콜 준수. (권장 응답 NFR은 2초) |
| REQ-FUNC-021 | Must | Story 1 | **Zero-Friction 센서 구동 백그라운드 모델:** 사용자가 손목 부착 기기 등의 조작 없이 환경 동력 기반 혹은 어댑터 체결 방식으로 수집 데이터 인제스트가 계속 구동되도록 센서를 상시 핑 처리한다. | AC-1.2: 시스템 모니터링 라이프 사이클 동안 대상자(어르신)의 수동 조작/충전 마찰 횟수가 0건 산출. |
| REQ-FUNC-022 | Must | Story 1 | **안정성 감지 장애 알림 (Ping Check):** 클라우드는 Edge 디바이스에서 전송되어야 할 Heartbeat 신호가 15분 초과로 유실될 때에, 물리적 단절로 인식하고 관리용 앱으로 기기 오프라인 알림을 자체 생성하여 발송한다. | AC-1.5: 장애 상황 단선 15분 경과 후 시스템은 '디바이스 연결 단절' 모바일/웹 Push 메시지를 트리거한다. |
| REQ-FUNC-031 | Must | Story 2 | **공간 맥락 지표 처리 데이터베이스 구축:** 각 엣지가 제공하는 설치 환경 정보(침실, 욕실) 값을 바인딩하여 카메라 동작 없이 순수 센서 포지션과 파형만으로 공간 이동 지표 및 화장실 방문 빈도수 계산 엔진을 가동한다. | AC-2.1: 시스템은 24시간 주기로 일간 화장실 빈도수, 수면 중단 측정 분석 시 영상 기록 방식 대비 오차율을 허용치 10% 미만으로 통과해야 한다. |
| REQ-FUNC-041 | Must | Story 3 | **외부 시스템 관제 EMR Webhook 연동기 제공:** 발생한 응급 이벤트와 상태 모니터링 파라미터를 외부 B2B 운영 요원이 지정한 Endpoint로 JSON 포맷 기반으로 자동 POST 등록한다. | AC-3.2: EMR 시스템으로 데이터 전송되어 B2B 간호 기록 간 100% 이벤트 불일치 제거(이중 수기 0회). AC-3.4(Ex): EMR 500 에러 감지시, 즉시 자체 대시보드 경고 표출 및 장애 복구 로직 3회 가동. |
| REQ-FUNC-042 | Must | Story 3 | **B2B용 90일치 데이터 해시 열람 인벤토리 엔진:** 관제 관리자 페이지에서 과거 최소 90일 단위 기록을 백업 DB에서 호출, 분쟁 조정 활용 가능하도록 보안성 높은 검색/목록 조회를 제공해야 한다. | AC-3.3: 검색 API 필드는 90일 보존분까지 날짜 지정이 가능하며 데이터 무결성을 보장하는 해시 검증값을 포맷 응답 객체로 첨부해야 한다. |
| REQ-FUNC-051 | Should | Story 2 | **데일리 웰니스 리포트 종합 및 발송 스케줄러 설계:** 앱 측의 WAU 상승 및 'P5무가치 체감 예방'을 목적으로 매 24시간 활동 데이터를 묶어 직관적 수면/체류 시간 리포트 문서를 생성해 수신자에게 아침에 전송한다. | AC-2.2: 특정 구역 체류(예시 화장실 이용시간 +50% 초과) 이상 기준선 변경 알림 내장 포함 발송. AC-2.4(Ex): 24시간 중 유효 측정 데이터 5h 미만 존재 시 리포트는 발송하되, 본문을 '상태 미확보 부족 정보' 컴포넌트로 분기하여 표출. |

   4.2 Non-Functional Requirements (비기능 요구사항)

| Requirement ID | Category | Requirement Description | Threshold / Target Expectation |
| --- | --- | --- | --- |
| REQ-NF-001 | 성능 | Edge단 긴급 판별부터 Cloud 경유, B2C/B2B 최종 App 단말 알람 도착까지의 전체 End-to-End 지연 속도 기준 규격. | 응답 지연 p95 ≤ 2,000 ms 이내 달성 |
| REQ-NF-002 | 성능 | 실 구동 시 야간 오탐 오류 빈도 발생 최고 허용 수준 제한. | 발생 건수 제한 목표: ≤ 0.3건 / 월 / 가구 |
| REQ-NF-003 | 성능 | 일간 화장실 빈도 및 야간 활동 수집 패턴 수치상 신뢰 제한 범위. | 계산 웰니스 데이터 오차율 10% 미만 보장 |
| REQ-NF-004 | 가용성 | SaaS 클라우드 플랫폼 인프라의 장애 방어 목표 수준(주요 API, 푸시 등). | 월간 SLA 보장율 ≥ 99.9% (최대 오프라인 장애 허용 43.8분/월) |
| REQ-NF-005 | 신뢰성 | 무선 Edge와 클라우드간의 데이터 전송 시 최종 수치 손실 허용 오차. | 전송 유실률 ≤ 0.1% 미만 (Retry 로직 지원 바탕) |
| REQ-NF-006 | 보안 | Edge To Cloud 통신 구간의 패킷 레이어 암호화 필수 수준 조건 규격. | TLS 1.3 암호화를 강제하며 비식별 처리 이후 메타데이터 형태의 수치만 전송 |
| REQ-NF-007 | 보안 | 데이터베이스 PII 식별 필드 조회 접근 규정 및 다중 사용자 권한 법. | B2B 다자 동의서에 연계되는 RBAC (Role-Based Access Control) 권한 적용 보장 |
| REQ-NF-008 | 비용 | 솔루션 규모 확대를 무사히 소화하기 위한 단위 트랜잭션 당 설계 상 한계 통제. | 배치 및 이벤트 드리븐 아키텍처 연동 하에 AWS 등 클라우드 지출 가구당 월 500원 이내 고정 |
| REQ-NF-009 | 운영성 | B2B/B2C 배포 기기의 오탐 한계치 하드웨어 제어 등을 다루는 일괄 모니터링 관리 지원 범위. | OTA 방식 무선 양방향 펌웨어 수정 업데이트 원격 채널 인프라 구성 |
| REQ-NF-010 | 운영유지 | 의료 기관 분쟁 등의 법적 증명 요구 지원을 위한 백업 스토리지 보존 수준치. | 분쟁 해시가 처리된 로그 이벤트 데이터 파일링 형식 90일 의무 장기 아카이브 백업 |
| REQ-NF-011 | 제약 | B2G, 식약처 위반 리스크를 우회하기 위해 DB 필드 및 소스 코드 상 금지된 어휘. | 필드/API에 'diagnosis', 'medical', 'patient' 단어 사용 절대 금지 ('wellness_score' 등으로 치환) |
| REQ-NF-012 | 모니터링장애 | [Alert 설정] 엣지 네트워크 대규모 오프라인 징후 발생에 따른 PagerDuty 긴급 콜 전파 임계 기준. | 1시간 이내 단일망 장비 전체의 3% 이상 Heartbeat 하락 시 Severity 1 채널로 엔지니어 콜 트리거 |

5. Traceability Matrix

| User Story ID | Requirement ID | Validation Test Case (Acceptance Criteria 연계) | Validation Layer / Technique |
| --- | --- | --- | --- |
| Story 1 (안심/지속성) | REQ-FUNC-011 | AC-1.1, AC-1.4: 10kg 미만 생명체 구동 환경에서 1만건 AI 테스트 시 오분류 0.3건/월 미만 입증. | AI 엣지 모듈 환경 가속 블랙박스 테스팅 |
| Story 1 (안심/지속성) | REQ-FUNC-012 | AC-1.3: 대상 움직임 5분 정지 모사 시 서버 알림 렌더링 60s 확보 점검. | 모바일 End-to-End 이벤트 송수신 자동화 스크립트 기능 검사 |
| Story 1 (안심/지속성) | REQ-FUNC-021 | AC-1.2: 물리적인 착용 제약 없이 데이터 전송률 정기적 인제스트 정상 확인. | 하드웨어 미착용 1주일 무간섭 부하 검사 |
| Story 1 (안심/지속성) | REQ-FUNC-022 | AC-1.5: 백본 라우터 공유기 전원 수동 차단 15분 후 오프라인 경고 확인. | 네트워크 인프라 장애 폴백 시뮬레이션 |
| Story 2 (데이터 예방) | REQ-FUNC-031, REQ-FUNC-051 | AC-2.1, AC-2.2, AC-2.4: 24h 이상 실 배치 사용량과 화장실/위치 측정 오차의 카메라 대조 확인 검사. 부족분 결측치 예외 메일링 동작 확인. | 실 대상 모의 48h 교차 시스템 테스트(CCTV 대조군 활용) |
| Story 3 (B2B 운영) | REQ-FUNC-041 | AC-3.2, AC-3.4: EMR 서버 HTTP 500 에러 스텁 설정 시, 대시보드 경보 및 백오프 재시도 카운트 관측. | Webhook 연계 스텁(Stub) 모킹 검증 및 대시보드 상태 검사 |
| Story 3 (B2B 운영) | REQ-FUNC-042, REQ-NF-010 | AC-3.3: 저장된 과거 90일 임의 이벤트 ROW 쿼리 후 응답 해시의 무결성 손상 시도 변조 탐지율 검증. | API 보안 침투테스트 및 DB 트랜잭션 롤백 무결성 확인 |

6. Appendix

   6.1 API Endpoint List

   **[외부 EMR 연계용] EMR Webhook API (F-API-01)**
   - **Method/Path:** `POST /api/v1/webhook/emr`
   - **Purpose:** 오탐이 걸러진 최종 유효 관제 이벤트 셋을 외부 요양 시설의 기존 전산망(EMR)로 전송. API Key + HMAC 보안 로직 적용. Rate Limiting 부과(100 req/min/facility).

   **[외부 알림 파견] Push Notification Trigger (F-API-02)**
   - **Method/Path:** `POST /api/v1/notify`
   - **Purpose:** B2C 보호자 앱용 FCM 및 APNs 프로바이더에 페이로드 전달. 데일리 리포트의 경우 매일 아침 예약 발송 구동.

   **[내부 엣지 수신] Edge Ingest Data (I-API-01)**
   - **Method/Path:** `POST /api/v1/ingest`
   - **Purpose:** 센서 노드에서 가공된 이진수 1/0 상태, 식별 불가능한 형태의 5분 배치성 체류 타임과 파형 추세 메타데이터 수집 기능 (강제 TLS 1.3 암호화).

   **[내부 보안 검색] Log Archive Query (I-API-02)**
   - **Method/Path:** `GET /api/v1/logs/archive`
   - **Purpose:** 90일분 데이터 조회 권한을 보유한 기관 관리자가 호출 시 법적 보전 해시 키가 래핑 처리된 단일 기록 JSON 배열을 리턴하여 조회 지원.

   6.2 Entity & Data Model

| Entity Name | Field | Database Type | Constraint/Description |
| --- | --- | --- | --- |
| **SensorDevice** | `device_id` | UUID | 센서별 발급되는 고유 식별자 단일키 |
| **SensorDevice** | `location_zone` | ENUM | {bedroom, bathroom, living_room} 설치 구역 |
| **SensorDevice** | `firmware_version` | STRING | OTA를 위한 펌웨어 최신 버전 값 식별 |
| **SensorDevice** | `installation_date` | DATETIME | 시스템 초도 등록일 |
| **SensorDevice** | `status` | ENUM | {active, inactive, maintenance} 가동 상태 값 |
| **SensorDevice** | `calibration_status` | ENUM | {calibrated, pending} 환경 최적화 상태 플래그 |
| **WellnessEvent** | `event_id` | UUID | 유효 데이터별 식별 가능한 UUID |
| **WellnessEvent** | `device_id` | UUID (FK) | SensorDevice 참조 연결키 |
| **WellnessEvent** | `event_type` | ENUM | {activity_alert, wellness_score, emergency} 사건 형식 |
| **WellnessEvent** | `timestamp` | DATETIME | 레이더 엔진 로직에 의해 캡처된 시간 |
| **WellnessEvent** | `confidence_score` | FLOAT | 0.0 ~ 1.0 의 AI 정확성 추정 스코어 (90% 초과 등 활용) |
| **WellnessEvent** | `is_false_alarm` | BOOLEAN | 사용자 피드백 오탐 결과 (성공지표 검사용) |
| **WellnessEvent** | `zone` | ENUM | 이벤트가 일어난 섹터 |
| **UserAccount** | `user_id` | UUID | 관제 관리자 및 보호자 회원 ID |
| **UserAccount** | `role` | ENUM | {guardian, facility_admin} 접근 PII 권한 제한용 |
| **UserAccount** | `linked_devices` | UUID Array | 인가받은 관제 기기들 매핑 주소 |
| **UserAccount** | `notification_pref` | JSON Object | 사용자의 알림 수신 주기 및 활성화 타임스택 포맷 |
| **DailyReport** | `report_id` | UUID | 일간 웰니스 결산 통계 문서 ID |
| **DailyReport** | `device_id` | UUID (FK) | SensorDevice 참조 연결키 |
| **DailyReport** | `date` | DATE | 결산의 기준이 되는 윈도우 날짜 |
| **DailyReport** | `sleep_score` | INT | 수합된 야간 수면 질 점수 (0-100 형태) |
| **DailyReport** | `bathroom_visit_count` | INT | 야간 이석/욕실 이동 집계 합 |
| **DailyReport** | `anomaly_flags` | STRING Array | 체류지연(+50% 등) 및 이상 징후 감출 태그명 리스트 |
| **DailyReport** | `generated_at` | DATETIME | 시스템에서 PDF/JSON 객체가 생성된 타임스탬프 |

   6.3 Detailed Interaction Models
   (상세 시퀀스 다이어그램)
   
   **상세 API 기반 긴급 이벤트 및 폴백 아키텍처 구조 (Detailed Error Control)**
   ```mermaid
   sequenceDiagram
       participant Sensor as UWB Edge Sensor
       participant I_API as Ingest API Gateway (I-API-01)
       participant EMR_Webhook as EMR Webhook Engine (F-API-01)
       participant External_EMR as EMR DB System
       participant Admin as B2B Admin Dashboard
       
       Sensor->>I_API: 1. 긴급 이상 패턴 수치 객체 인제스트 시도
       I_API->>I_API: 2. PII 통제, TLS 검수 & 90일 데이터 해시 보존용 원거리 백업
       I_API->>EMR_Webhook: 3. 정상 이벤트 수집 확정 - EMR 배포 트리거
       
       EMR_Webhook->>External_EMR: 4. POST /webhook 발송 Payload(JSON) 전달
       
       alt HTTP 200/201 (정상 처리)
           External_EMR-->>EMR_Webhook: 5. 200 OK Response
           EMR_Webhook->>Admin: 6. 초록색 Safe UI 및 싱크 동기화 아이콘 상태 출력
       else HTTP 5XX / Connection Timeout (외부 망 일시적 서버 장애)
           External_EMR-->>EMR_Webhook: 5. Failed Response 발생
           EMR_Webhook->>EMR_Webhook: 6. Retry 루프 가동 (Max 3회 설정, Exponential 방식 대기)
           EMR_Webhook-->>External_EMR: 재차 Payload 발송 (1회차 등)
           External_EMR-->>EMR_Webhook: 지속 Fail 발생
           EMR_Webhook->>Admin: 7. B2B 대면 대시보드 강제 '연동 끊김' 에러 및 <br>"비상 점검 권고" 수기 안내 UI 긴급 팝업 렌더링!
       end
   ```
