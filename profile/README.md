# Crowfoot

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

**한국어** | **[English](./README.en.md)**

**설계가 끝나면, 데이터베이스가 시작됩니다.**

Crowfoot은 브라우저에서 동작하는 오픈소스 ERD 에디터입니다. 논리 모델 설계부터 물리 모델 변환, 팀 협업, 그리고 실제 데이터베이스 발급까지 — 데이터 작업의 처음부터 끝까지 한 곳에서 제공합니다. 이름인 Crowfoot은 테이블 간 관계를 까마귀발 모양의 기호로 표기하는 **까마귀발(Crow's Foot) 표기법**에서 따왔습니다.

## 서비스

| 구분 | 주소 |
| --- | --- |
| 웹 (에디터·대시보드) | https://crowfoot.java21.net |
| API 게이트웨이 | https://crowfoot-api.java21.net |
| 협업 WebSocket 서버 | ws://crowfoot-ws.java21.net |
| 시스템 ERD | https://crowfoot.java21.net/share/1KeFkNED0uTmx6MPWqmXph |

Crowfoot 시스템 자체의 ERD도 Crowfoot으로 직접 설계했습니다 — 위의 공유 링크에서 확인할 수 있습니다.

## 무료 매니지드 데이터베이스

Crowfoot의 핵심 기능입니다. PostgreSQL·MySQL 개발용 데이터베이스를 **계정당 최대 5개까지 무료로 발급**합니다.

- 발급 시 **전용 DB 계정이 자동 생성**됩니다 — 스키마 한정 권한을 가진 계정이고, 인스턴스 루트 자격은 어디에도 노출되지 않습니다
- 발급된 비밀번호는 AES-256-GCM으로 암호화해 보관하며, 소유자만 접속 정보를 조회할 수 있습니다
- 발급 즉시 연결 테스트를 거쳐 외부 클라이언트에서 바로 사용할 수 있고, 필요 없어지면 철회할 수 있습니다
- 개발·학습·테스트 용도의 데이터베이스입니다

## 주요 기능

### 브라우저 ERD 에디터

- 설치·설정 없이 브라우저에서 바로 — 까마귀발(Crow's Foot) 표기법으로 테이블과 관계를 그립니다
- **논리 모델과 물리 모델을 함께 관리** — 논리명·물리명 병행, 공용 논리 타입을 DBMS별 물리 타입으로 자동 매핑
- **DBMS 템플릿 5종** — PostgreSQL·MySQL·Oracle·MSSQL(＋공용) 각각의 타입·AI·코멘트 문법을 반영
- 모델 검증(이름 중복·참조 무결성 등)과 자동 레이아웃, 문서별 화면(줌·팬) 기억
- 대용량 문서(100 테이블)에서도 60fps 팬·줌을 유지합니다

### SQL 생성·배포·역설계

- 문서를 해당 DBMS에 맞는 **DDL 스크립트로 생성** — 미리보기·복사·다운로드 (테이블 코멘트는 논리명으로)
- **Forward Engineering** — 생성한 DDL을 발급받은 DB 또는 직접 등록한 DB에 바로 배포
- **Reverse Engineering** — 기존 데이터베이스의 스키마·코멘트를 읽어 ERD 문서로 가져오기

### 실시간 협업

- WebSocket(STOMP) 기반 — 같은 문서를 여럿이 동시에 편집하고, 접속 현황(presence)과 변경사항이 실시간으로 동기화됩니다
- 문서에 댓글을 남길 수 있습니다

### 워크스페이스·팀 권한

- 워크스페이스 단위로 문서를 정리하고, 사용자·팀을 초대합니다
- **소유자·편집자·댓글 작성자·뷰어** 역할 기반 권한으로 팀 작업을 안전하게 관리합니다

### 데이터베이스 연동

- 직접 운영하는 DB의 접속 정보를 저장(암호화)하고 연결을 테스트할 수 있습니다
- 매니지드 DB와 개인 DB를 구분 없이 하나의 목록에서 관리합니다

### 인증·관리

- **GitHub·Google OAuth2 로그인** — Access 토큰은 메모리에만, Refresh는 `SameSite=Strict` 쿠키로 보관하고 게이트웨이가 매 요청 introspection 검증합니다
- 관리자 콘솔 — 사용자·코드 테이블·매니지드 DB 인스턴스·발급 한도·감사 로그 관리

## 아키텍처

MSA 5종으로 구성됩니다.

```
  브라우저
    │ HTTPS                          │ WebSocket (STOMP)
    ▼                                ▼
┌──────────────┐              ┌──────────────┐
│ crowfoot-web │              │crowfoot-collab│  presence·실시간 동기화
│  React SPA   │              └──────────────┘
└──────┬───────┘
       ▼
┌──────────────┐     ┌──────────────┐
│ api-gateway  │────▶│  auth        │  OAuth2·JWT·introspection·Redis
└──────┬───────┘     └──────────────┘
       │             ┌──────────────┐
       └────────────▶│  core-api    │  도메인(PostgreSQL)·매니지드 DB
                     └──────┬───────┘  프로비저닝(PostgreSQL·MySQL)
```

## 저장소

| 저장소 | 설명 |
| --- | --- |
| [crowfoot-web](https://github.com/crowfoot-erd/crowfoot-web) | 프론트엔드 — React SPA. ERD 에디터(React Flow), 대시보드·워크스페이스·팀, 관리자 콘솔, 협업 클라이언트(STOMP), i18n(ko·en)·다크 모드 |
| [crowfoot-api-gateway](https://github.com/crowfoot-erd/crowfoot-api-gateway) | API 게이트웨이 — Spring Cloud Gateway. 라우팅, Bearer 토큰 introspection 검증, 사용자 식별 헤더 주입, 공개 경로 화이트리스트 |
| [crowfoot-auth](https://github.com/crowfoot-erd/crowfoot-auth) | 인증 서버 — OAuth2 로그인(GitHub·Google·PKCE), JWT 발급·갱신·introspection, Redis 블랙리스트(로그아웃) |
| [crowfoot-core-api](https://github.com/crowfoot-erd/crowfoot-core-api) | 코어 API — 회원·워크스페이스·팀·모델 문서·댓글, 매니지드 DB 프로비저닝(전용 계정 발급·철회), SQL 생성·배포·리버스 엔지니어링, 코드 테이블·감사 로그 |
| [crowfoot-collab](https://github.com/crowfoot-erd/crowfoot-collab) | 협업 서버 — WebSocket(STOMP). 문서별 presence(접속 현황)와 편집 변경사항의 실시간 브로드캐스트, 단일 인스턴스 운영 |

## 릴리스

| 버전 | 날짜 | 주요 내용 | 태그 | 릴리스 노트 |
| --- | --- | --- | --- | --- |
| v1.14 | 2026-09-25 | 용어 사전 패널·시스템 사전 관리, 추론 언어 선택, 컬럼 물리명 사전 제안 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.14) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.14) | [보기](https://crowfoot.java21.net/release-notes/16) |
| v1.13 | 2026-09-24 | 논리 그룹(주제 영역), 논리명 자동 추론, 단축키 치트시트 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.13) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.13) | [보기](https://crowfoot.java21.net/release-notes/15) |
| v1.12 | 2026-09-23 | 모델 익스플로러·통합 검색, SQL 가져오기 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.12) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.12) | [보기](https://crowfoot.java21.net/release-notes/14) |
| v1.11 | 2026-09-22 | 관계 편집·버전 비교 개선, 빠른 이미지 내보내기 | — | [보기](https://crowfoot.java21.net/release-notes/13) |
| v1.10 | 2026-09-21 | 버전 비교·마이그레이션 DDL | — | [보기](https://crowfoot.java21.net/release-notes/11) |
| v1.09 | 2026-09-20 | 문서 버전 기록·DB 동기화 | — | [보기](https://crowfoot.java21.net/release-notes/10) |
| v1.08 | 2026-09-18 | 커뮤니티 게시판·실시간 채팅·에디터 안전장치 | — | [보기](https://crowfoot.java21.net/release-notes/9) |

릴리스마다 변경 사항을 [릴리스 노트](https://crowfoot.java21.net/)로 정리해 공개한다(랜딩의 최근 릴리스에서 전체 목록을 볼 수 있다). git 태그는 v1.12부터 각 리포에 남긴다.

## 직접 실행하기

### 사전 요건

| 도구 | 버전 | 대상 |
| --- | --- | --- |
| Java (Temurin) | 21 | 서버 4종 |
| Maven | 3.9+ | 서버 4종 빌드·기동 |
| Node.js / pnpm | 20 / 10 | 프론트엔드 |
| PostgreSQL | 16+ | 도메인 DB (`crowfoot` 데이터베이스 + `crowfoot_core` 스키마) |
| Redis | 6+ | 인증 로그아웃 블랙리스트 |

스키마는 자동 생성하지 않는다(`ddl-auto: none`) — DDL 스크립트로 초기화한다.

### 로컬 포트 구성

| 서비스 | 포트 | 비고 |
| --- | --- | --- |
| crowfoot-web (Vite) | 8080 | `/api` 요청을 게이트웨이(8000)로 프록시 |
| crowfoot-api-gateway | 8000 | `auth`(8081)·`core-api`(8082)로 라우팅 |
| crowfoot-auth | 8081 | |
| crowfoot-core-api | 8082 | |
| crowfoot-collab | 8083 | WebSocket(STOMP) — 게이트웨이 경유 없이 직접 연결 |

### 환경변수

각 서버는 리포 루트의 `.env-local`(gitignore 대상)을 자동으로 읽는다 — `.env-local.example`을 복사해 값을 채운다. 게이트웨이와 collab은 필요한 환경변수가 없다.

**crowfoot-auth**

| 변수 | 설명 |
| --- | --- |
| `CROWFOOT_AUTH_JWT_SECRET` | JWT HS256 서명 키 — Base64 32바이트 이상 (`openssl rand -base64 48`) |
| `CROWFOOT_AUTH_FLOW_SECRET` | `auth_flow` 쿠키 HMAC-SHA256 서명 키 — JWT 키와 용도 분리 |
| `CROWFOOT_AUTH_GITHUB_CLIENT_ID` / `..._SECRET` | GitHub OAuth 앱 자격 |
| `CROWFOOT_AUTH_GOOGLE_CLIENT_ID` / `..._SECRET` | Google OAuth 클라이언트 자격 (PKCE) |
| `CROWFOOT_REDIS_PASSWORD` / `CROWFOOT_REDIS_DATABASE` | Redis 블랙리스트 접속 (호스트는 `application-local.yml`) |

OAuth 앱에는 리디렉션 URI `http://localhost:8080/auth/callback`을 등록한다.

**crowfoot-core-api**

| 변수 | 설명 |
| --- | --- |
| `DB_URL` | PostgreSQL JDBC URL — `jdbc:postgresql://{host}:5432/crowfoot?currentSchema=crowfoot_core` |
| `DB_USERNAME` / `DB_PASSWORD` | 도메인 데이터베이스 계정 |

**crowfoot-web** — 개발은 기본값 그대로 (`VITE_API_BASE_URL` 빈 값 → Vite 프록시로 같은 오리진 유지). 운영 빌드는 `VITE_API_BASE_URL`(API 게이트웨이 주소)·`VITE_WS_URL`(협업 WS 주소)을 주입한다.

### 기동

```bash
# 서버 4종 — 각 리포에서 (로컬 프로필이 기본값)
mvn spring-boot:run

# 프론트엔드
pnpm install
pnpm dev        # http://localhost:8080
```

### 운영

운영에서는 5종 전부 컨테이너 이미지로 빌드되고 서버 포트는 8080으로 통일한다. 시크릿은 코드·이미지에 두지 않고 환경변수로만 주입한다 — 각 리포의 `application-prod.yml`이 참조하는 환경변수가 필요하다.

## 기술 스택

**프론트엔드** — React 19 · TypeScript · Vite · TanStack Query · Zustand · React Flow · ELK(자동 레이아웃) · Tailwind CSS · shadcn/ui(radix-ui) · i18next · Vitest·Testing Library·Playwright·MSW

**백엔드** — Java 21 · Spring Boot 4 · Spring Security(OAuth2 Client) · Spring Cloud Gateway · Spring Data JPA(Hibernate 7) · Querydsl · Spring WebSocket(STOMP)

**데이터베이스** — PostgreSQL (도메인·매니지드 발급) · MySQL (매니지드 발급) · Redis (인증 세션)

**인프라** — GitHub Actions · GHCR · Kubernetes(Rancher) · ArgoCD (GitOps)

## 라이선스·기여

모든 저장소는 [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)로 배포됩니다. 버그 제보와 기여는 언제나 환영합니다 — 각 저장소의 Issue를 이용해 주세요.
