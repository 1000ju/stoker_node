# Stocker Node Backend

주식 학습·투자 습관 형성 앱의 백엔드. 기획부터 ERD, API 명세, Node.js 구현, Docker 기반 배포까지 백엔드 전 과정을 맡았습니다. 이 프로젝트는 팀 프로젝트이며, 저는 백엔드 파트를 전담했습니다.

## 제작 배경 / 문제 정의
- 투자 입문자가 **매일 학습 습관**을 만들기 어렵다는 문제에 착안해, 출석 퀴즈·투자 성향 검사·학습 콘텐츠·오답 노트를 한 번에 제공하는 백엔드를 설계했습니다.
- 프런트 팀이 빠르게 실험할 수 있도록 **일관된 API 규격**과 **초기 데이터 시드**를 제공하는 것을 목표로 삼았습니다.

## 내가 담당한 역할 (백엔드)
- 기획·ERD·API 설계·Node.js 구현·Docker 배포까지 백엔드 전 과정을 전담했습니다.
- 인증/인가, 학습·평가 도메인 API, 배포 파이프라인을 모두 구현하고 운영 환경 설정을 서포트 했습니다.
- Swagger(OpenAPI) 문서, 마이그레이션·시드 스크립트, 헬스체크 엔드포인트를 정리해 팀원 온보딩을 지원했습니다.

## 주요 기능 (데이터 흐름·API 관점)
- **인증·프로필**: 이메일 가입/로그인 → bcrypt 해시 저장 → Access/Refresh 토큰 발급 → 만료 시 `x-refresh-token`으로 자동 재발급.
- **데일리 출석 퀴즈**: 랜덤 3문제 제공 → 정답 제출 시 출석 기록 생성 → 월별 출석 조회로 습관 데이터 제공.
- **투자 성향 검사**: 설문 응답을 4차원 코드(CLDP 등)로 산출 → `InvestmentProfile` 업서트 → 성향과 유사한 투자 거장 추천.
- **학습/퀴즈/오답노트**: 챕터·이론 진행도 동기화 → 객관식 퀴즈 자동 채점 → 오답만 `WrongNote`로 관리 → 재시도/힌트 제공.
- **학습 메모**: 일지/복기/체크리스트/자유/재무제표 템플릿별 CRUD 제공.
- **공통 인프라**: JWT 미들웨어, CORS 화이트리스트, `/healthz | /readyz | /api/health` 헬스체크, Swagger UI(`/api-docs`).

## 기술 스택 (선택 이유)
- **Node.js 20 + Express 4**: 빠른 프로토타이핑과 생태계 활용을 위해 선택, 미들웨어 기반으로 인증/로깅을 모듈화.
- **Sequelize + MySQL 8.0**: 명시적 모델링과 마이그레이션 관리가 필요했고, 팀이 MySQL에 익숙해 러닝 커브 최소화.
- **JWT(access/refresh)**: 모바일/웹 클라이언트 공통 사용성을 고려해 상태 비저장 인증 채택.
- **Swagger UI(OpenAPI)**: 프런트 팀과의 계약을 코드에 가깝게 유지하기 위해 스키마를 단일 소스로 관리.
- **Docker & docker-compose**: 개발·배포 환경을 동일하게 재현하고, MySQL 포함 멀티 컨테이너 구성을 단순화.

## 배포 및 운영 경험
- **Linux 서버 + Docker**: Dockerfile 멀티스테이지로 이미지 빌드, `docker-compose.yml`로 앱·MySQL 동시 기동.
- **환경 변수 분리**: `.env.production`을 통해 DB·JWT 시크릿·CORS 오리진을 주입, 이미지 내 하드코딩 방지.
- **로그/모니터링**: `NODE_ENV=development`에서 상세 로그, 프로덕션은 헬스체크(`/api/health`)로 서비스 상태 확인. 필요 시 `docker logs`로 컨테이너 단위 분석.

## 프로젝트 구조 요약
```
src
├── app.js                 # Express 부트스트랩, 공통 미들웨어, Swagger 로더
├── config/db.js           # MySQL 연결 설정
├── middleware/auth...     # JWT 인증 및 재발급
├── model/                 # Sequelize 모델/어소시에이션
├── migrations/, seeders/  # 스키마·초기 데이터 버전 관리
├── user/, attendance/, investment_profile/, memo/
├── chapter/, theory/, quiz/, wrong_note/
└── utils/jwt.util.js      # 토큰 헬퍼
```
`init.sql` 및 `mysql/init.sql`로 전체 스키마와 샘플 데이터를 한 번에 초기화할 수 있습니다.

## 기술 스택
| 영역 | 기술 | 메모 |
| --- | --- | --- |
| 런타임 | Node.js 20, Express 4 | 경량 REST API, 미들웨어 확장성 |
| 인증 | JWT(access/refresh) | 무상태 인증으로 모바일·웹 공통 사용 |
| DB/ORM | MySQL 8.0, Sequelize 6 | 모델·마이그레이션 표준화, 트랜잭션 활용 |
| 문서화 | Swagger UI, OpenAPI 3.0 | `openapi.yml` 단일 출처로 계약 관리 |
| 인프라 | Docker, docker-compose | 동일 환경 재현, 앱+DB 멀티 컨테이너 |

## 개발하며 배운 점
- 요구사항이 자주 바뀌는 초기 단계에서 **OpenAPI 기반 계약**이 프런트/백 협업 속도를 크게 높인다는 것을 체감했습니다.
- **Refresh 토큰 재발급 플로우**를 구현하며, 만료·탈취·중복 로그인 등 경계 사례에 대한 테스트의 중요성을 배웠습니다.
- Docker로 DB까지 포함한 로컬 스택을 제공하니 신규 팀원이 바로 실 서버와 동일한 흐름을 경험할 수 있어 온보딩 비용이 줄었습니다.

> 이 프로젝트는 팀 프로젝트이며, 저는 백엔드 파트를 전담했습니다. 기획 단계부터 ERD 설계, API 명세 작성, Node.js 구현, Docker 기반 배포까지 담당했습니다.
