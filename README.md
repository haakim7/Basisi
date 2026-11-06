# Basisi (베시시)

**데이터로 검증하고 AI가 추천하는 프리미엄 베이비시터 매칭 플랫폼**

> "데이터로 검증된 안심, AI가 설계한 프리미엄 매칭으로 아이의 미소를 지켜주세요."

| 구분 | 내용 |
| --- | --- |
| 서비스명 | 국문 **베시시** (아이의 환한 웃음 형상화) / 영문 **Basisi** (Babysitter System) |
| 팀원 | 김준하, 서석현, 왕서빈, 임다솔, 이주영 |
| 개발 기간 | 15주 계획 (아래 [개발 일정](#7-개발-일정) 참고) |


## 1. 프로젝트 개요

베이비시터의 상세 이력과 활동 데이터를 바탕으로 부모가 직접 최적의 시터를 탐색하고 신청하는 플랫폼입니다. AI가 분석한 **성장형 활동 온도(불꽃)** 와 비동기 데이터 파이프라인으로 매칭의 신뢰도와 생동감을 높였습니다.

## 2. 핵심 가치

| 가치 | 설명 |
| --- | --- |
| User-Centric Direct Matching | 시터의 전문 이력과 자격증을 부모가 직접 검토하는 자율 선택권을 보장하고 AI 추천으로 탐색 효율을 높입니다. |
| Active Warmth (활동의 온기) | 수직적 티어제 대신 활동 횟수와 긍정 피드백 기반의 '성장형 불꽃'으로 시터의 열정과 최신 활동성을 직관적으로 보여 줍니다. |
| Seamless Reliability | 인기 시터에 예약이 몰릴 때 생기는 데이터 충돌을 Redisson 분산 락으로 해결해 중복 없는 예약 시스템을 구축합니다. |
| Proactive Insight | 서비스 종료 후 아이의 성장 지표와 활동 내역을 요약한 성장 리포트를 자동 발송합니다. |

## 3. 주요 타겟

| 대상 | 설명 |
| --- | --- |
| 부모 | 신원이 불분명한 플랫폼에 불안감을 느끼며 데이터로 검증된 고숙련 전문 시터를 찾는 맞벌이 부부 |
| 베이비시터 | 전문성을 입증하고 정당한 처우를 받고자 하는 전문 베이비시터와 보육 경력자 |

## 4. 백엔드 차별화 포인트

| 포인트 | 설명 |
| --- | --- |
| DB 확정 후 LLM 재순위 추천 | DB가 부모의 5가지 조건으로 후보를 먼저 확정하고 LLM은 그 안에서 재순위와 추천 사유만 맡습니다. LLM 응답이 후보와 어긋나면 알고리즘 결과로 폴백해 추천이 끊기지 않습니다. |
| 다중 조건 검색 | QueryDSL로 지역·성별·경력 등 조건을 동적으로 조합하고 페이징과 정렬을 지원합니다. 위치 검색을 위한 PostGIS 스키마도 준비했습니다. |
| 안정적인 동시성 제어 | Redisson 분산 락으로 예약과 결제 시 레이스 컨디션을 막습니다. |
| 비동기 AI 분석 파이프라인 | Kafka 기반 메시지 큐로 리뷰 분석과 리포트 생성을 비동기로 처리합니다. |

## 5. 팀원별 역할

| 이름 | 역할 | 핵심 기술 | 주요 작업 |
| --- | --- | --- | --- |
| 이주영 | 인프라 및 보안 | Spring Security, JWT, Docker, AWS, nGrinder | [Phase 1] 초기 환경 세팅, JWT 인증/인가와 로그인 API<br>[Phase 4] AWS 배포와 nGrinder 부하 테스트 |
| 김준하 | 데이터 설계, 검색 및 AI 시터 추천 | Spring Boot, JPA, QueryDSL, PostgreSQL, PostGIS, OpenAI API(WebClient) | [Phase 1] 도메인 모델(Entity) 설계와 프로필 CRUD API<br>[Phase 2] QueryDSL 다중 조건 검색, PostGIS 위치 검색 스키마<br>[Phase 3] DB 확정 + LLM 재순위 AI 시터 추천<br>[배포] Render 배포 구성 |
| 임다솔 | 비즈니스 로직 및 동시성 | Spring Transaction, Redis, Redisson | [Phase 2] 플랫폼 핵심 예약 워크플로우<br>[Phase 3] Redisson 분산 락으로 중복 예약 방지 |
| 서석현 | AI 및 메시징 파이프라인 | Kafka/RabbitMQ, LLM API, WebClient | [Phase 3] Kafka 메시지 브로커와 비동기 파이프라인 설계, '활동 온도(불꽃)' 알고리즘 |
| 왕서빈 | 통찰 제공 및 실시간 알림 | SSE, AI Prompting, JPA | [Phase 2] 리뷰와 평점 시스템<br>[Phase 3] 맞춤형 '아이 성장 리포트'<br>[Phase 4] SSE 기반 실시간 알림 |

## 6. 주요 구현 내용

### 6-1. 도메인 모델과 프로필 CRUD API

- 시터와 부모의 프로필 Entity와 연관 테이블을 JPA로 설계했습니다.
- 시터 프로필은 소개·나이·성별·경력·자격증 보유·활동 지역·국적·가능 활동·희망 시급·CCTV 동의·선호 연령을 담습니다.
- 부모 프로필은 지역·아이 정보·근무 형태·일정 유형·희망 시터 조건(나이·성별·경력·국적·지역)·시터에게 전할 말을 담습니다.
- 프로필 조회·등록·수정·삭제 API와 시터 프로필 사진 업로드 API를 구현하고 Swagger로 문서화했습니다.

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| GET / PUT / DELETE | `/api/profile/parent/me` | 부모 프로필 조회·저장·삭제 |
| GET / PUT / DELETE | `/api/profile/sitter/me` | 시터 프로필 조회·저장·삭제 |
| POST / DELETE | `/api/profile/sitter/me/images` | 시터 프로필 사진 업로드·삭제 |

### 6-2. QueryDSL 다중 조건 검색

- 지역·성별·최소 경력 조건을 `BooleanBuilder`로 동적으로 조합합니다.
- 불꽃 점수·경력·나이·최신순 정렬과 페이징을 지원합니다.
- 시터 정보와 활동 점수(불꽃 점수·평점·완료 예약 수·최근 활동 수)를 한 번에 조회하는 Projection으로 응답합니다.
- PostgreSQL에서 쓸 PostGIS 위치 컬럼(`geography(Point, 4326)`)과 GIST 공간 인덱스 스키마 SQL을 작성했습니다. (`src/main/resources/db/postgresql/phase2_postgis.sql`)

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| GET | `/api/sitters/search`, `/api/sitters/search/page` | 조건 검색, 페이징 검색 |
| GET | `/api/sitters/{sitterProfileId}` | 시터 상세 조회 |

### 6-3. AI 시터 추천 (DB 확정 + LLM 재순위)

부모 마이페이지에 등록한 조건으로 시터를 추천하는 기능입니다. **후보 선정은 DB가 확정하고 LLM은 그 후보 안에서 재순위와 추천 사유만 맡도록** 역할을 나눴습니다.

| 단계 | 처리 | 담당 |
| --- | --- | --- |
| 1 | 부모 프로필의 5가지 조건(나이·성별·경력·국적·지역)을 평면 파라미터로 정규화 | 서비스 |
| 2 | QueryDSL `CASE WHEN` 합산으로 조건 일치 개수를 DB에서 계산하고 일치 수와 불꽃 점수 순으로 후보를 조회 (요청 인원의 3배, 최소 20명) | DB |
| 3 | 모든 조건을 만족하는 시터가 없으면 1개 부족한 시터까지, 그래도 없으면 불꽃 점수 상위 시터로 단계적으로 완화 | 서비스 |
| 4 | 후보와 부모 조건을 JSON 프롬프트로 구성해 OpenAI(`gpt-4o-mini`)를 호출하고 재순위·추천 사유·요약을 JSON으로 받음 | LLM |
| 5 | 응답을 검증해 병합. 후보에 없는 ID나 중복·잘못된 순위가 오면 알고리즘 결과로 폴백 | 서비스 |

LLM 호출 구조는 다음과 같이 안정적으로 동작하도록 구성했습니다.

- **후보 고정:** 프롬프트에 후보 목록 밖의 시터는 출력하지 않도록 규칙을 두고 응답 ID를 후보 집합과 대조해 검증합니다.
- **폴백:** API 키 미설정·호출 실패·타임아웃(15초)·응답 검증 실패 시 알고리즘 순위를 그대로 반환하고 응답에 LLM 폴백 여부를 표시합니다.
- **캐시:** 같은 사용자와 같은 입력은 30분간 메모리 캐시로 재사용합니다.
- **호출 제한:** 사용자당 분당 2회로 제한해 비용과 과호출을 막습니다.
- **추천 근거 제공:** 조건별 일치 사유와 매칭률을 함께 내려 줘 부모가 직접 비교할 수 있습니다.

| 메서드 | 경로 | 설명 |
| --- | --- | --- |
| POST | `/api/recommendations/sitters` | 로그인한 부모 기준 AI 시터 추천 |

### 6-4. 배포

- Docker 멀티 스테이지 빌드(Gradle 빌드 후 JRE 이미지 실행)로 Render Web Service에 배포하도록 구성했습니다.
- Render 환경의 PORT·CORS 설정과 프론트엔드 빌드 설정을 정리했습니다.

## 7. 개발 일정

15주 계획으로 진행했습니다.

| 단계 | 기간 | 내용 |
| --- | --- | --- |
| 1단계 핵심 기능과 UI 골격 | 3~4주차 (Phase 1) | 백엔드 기초(Security, CRUD), 프론트엔드 디자인 시스템(Soft Sky Blue, Warm Coral)과 인증 페이지 |
| | 5~7주차 (Phase 2) | PostGIS 기반 지도 검색과 예약 신청 로직, 프론트엔드 지도 연동 |
| | 8주차 | 중간 발표: 부모 로그인 → 지도 탐색 → 이력 확인 → 예약 신청 풀 프로세스 시연 |
| 2단계 기술 고도화와 AI 서비스 | 9~11주차 (Phase 3) | Redis 분산 락, Kafka 기반 AI 비동기 파이프라인, AI 추천 채팅 UI |
| | 12~13주차 (Phase 3) | '불꽃' 애니메이션 시각화, AI 성장 리포트 대시보드 |
| | 14주차 (Phase 4) | nGrinder 부하 테스트, SSE 실시간 알림 |
| | 15주차 | 최종 발표: 동시성 방지 테스트, AI 성장 리포트와 활동 온도(불꽃) 변화 시연 |

## 8. 기술 스택

| 영역 | 기술 |
| --- | --- |
| Backend | Java 17, Spring Boot 3.3, Spring Security(JWT), Spring Data JPA, QueryDSL 5.1, WebClient |
| Database | PostgreSQL(PostGIS), H2(기본 로컬 프로필) |
| AI | OpenAI API (`gpt-4o-mini`) |
| Messaging / Lock | Kafka, Redisson |
| Docs | Swagger (springdoc-openapi) |
| Frontend | React 18, Vite |
| Infra | Docker, Render |

## 9. 실행 방법

### 백엔드

기본 프로필은 H2(파일 저장)입니다. PostgreSQL을 쓰려면 `--spring.profiles.active=local`로 실행하세요.

```bash
./gradlew bootRun
```

- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAI API 키가 없어도 앱은 실행되며 추천은 기본 알고리즘으로 동작합니다.
- AI 추천을 켜려면 환경변수 `BASISI_OPENAI_API_KEY`를 설정합니다. 자세한 방법은 [docs/AI-SETUP.md](docs/AI-SETUP.md)를 참고하세요.

### 프론트엔드

```bash
cd frontend
npm install
npm run dev
```
