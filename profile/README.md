# Crowfoot

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

**설계가 끝나면, 데이터베이스가 시작됩니다.**

Crowfoot은 브라우저에서 동작하는 오픈소스 ERD 에디터입니다. 논리 모델 설계부터 물리 모델 변환, 팀 협업, 그리고 실제 데이터베이스 발급까지 — 데이터 작업의 처음부터 끝까지 한 곳에서 제공합니다.

## 서비스

| 구분 | 주소 |
| --- | --- |
| 웹 (에디터·대시보드) | https://crowfoot.java21.net |
| API 게이트웨이 | https://crowfoot-api.java21.net |
| 협업 WebSocket 서버 | crowfoot-ws.java21.net |

## 무료 매니지드 데이터베이스

Crowfoot의 핵심 기능입니다. PostgreSQL·MySQL 개발용 데이터베이스를 **계정당 최대 5개까지 무료로 발급**합니다.

- 발급 시 **전용 DB 계정이 자동 생성**됩니다 — 스키마 한정 권한을 가진 계정이고, 인스턴스 루트 자격은 어디에도 노출되지 않습니다
- 발급된 비밀번호는 AES-256-GCM으로 암호화해 보관하며, 소유자만 접속 정보를 조회할 수 있습니다
- 발급 즉시 연결 테스트를 거쳐 외부 클라이언트에서 바로 사용할 수 있고, 필요 없어지면 철회할 수 있습니다
- 개발·학습·테스트 용도의 데이터베이스입니다

## 주요 기능

### 브라우저 ERD 에디터

- 설치·설정 없이 브라우저에서 바로 — 카우풋(Crow's Foot) 표기법으로 테이블과 관계를 그립니다
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

## 기술 스택

**프론트엔드** — React 19 · TypeScript · Vite · TanStack Query · Zustand · React Flow · ELK(자동 레이아웃) · Tailwind CSS · shadcn/ui(radix-ui) · i18next · Vitest·Testing Library·Playwright·MSW

**백엔드** — Java 21 · Spring Boot 4 · Spring Security(OAuth2 Client) · Spring Cloud Gateway · Spring Data JPA(Hibernate 7) · Querydsl · Spring WebSocket(STOMP)

**데이터베이스** — PostgreSQL (도메인·매니지드 발급) · MySQL (매니지드 발급) · Redis (인증 세션)

**인프라** — GitHub Actions · GHCR · Kubernetes(Rancher) · ArgoCD (GitOps)

## 라이선스·기여

모든 저장소는 [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)로 배포됩니다. 버그 제보와 기여는 언제나 환영합니다 — 각 저장소의 Issue를 이용해 주세요.
