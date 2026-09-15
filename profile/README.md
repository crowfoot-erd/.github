# Crowfoot

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

**설계가 끝나면, 데이터베이스가 시작됩니다.**

Crowfoot은 브라우저에서 동작하는 오픈소스 ERD 에디터입니다. ERD 설계, 팀 협업, 그리고 실제 데이터베이스 —
데이터 작업의 처음부터 끝까지 한 곳에서 제공합니다.

## 무료 매니지드 데이터베이스

Crowfoot의 핵심 기능입니다. PostgreSQL·MySQL 개발용 데이터베이스를 **계정당 최대 5개까지 무료로 발급**합니다.
발급 즉시 연결 테스트를 거쳐 바로 사용할 수 있습니다.

## 주요 기능

- **무료 매니지드 데이터베이스** — PostgreSQL·MySQL 개발용 데이터베이스를 계정당 최대 5개까지 무료 발급
- **브라우저 ERD 에디터** — 설치·설정 없이 브라우저에서 바로, 카우풋(Crow's Foot) 표기법으로 테이블과 관계를 설계
- **실시간 협업** — 같은 문서를 여럿이 동시에 편집, 접속 현황과 변경사항이 실시간으로 동기화
- **데이터베이스 연동** — 직접 운영하는 DB의 접속 정보 저장과 연결 테스트
- **워크스페이스·팀 권한** — 소유자·편집자·댓글 작성자·뷰어 역할 기반 권한 관리
- **오픈소스** — 모든 코드를 GitHub에 공개, 직접 확인하고 기여할 수 있습니다

## 저장소

| 저장소 | 설명 |
| --- | --- |
| [crowfoot-web](https://github.com/crowfoot-erd/crowfoot-web) | 프론트엔드 — React SPA (에디터·대시보드·협업) |
| [crowfoot-api-gateway](https://github.com/crowfoot-erd/crowfoot-api-gateway) | API 게이트웨이 — Spring Cloud Gateway, 인증 introspection |
| [crowfoot-auth](https://github.com/crowfoot-erd/crowfoot-auth) | 인증 서버 — OAuth2 로그인(GitHub·Google)·JWT 발급·갱신 |
| [crowfoot-core-api](https://github.com/crowfoot-erd/crowfoot-core-api) | 코어 API — 회원·워크스페이스·모델 문서·매니지드 데이터베이스 |
| [crowfoot-collab](https://github.com/crowfoot-erd/crowfoot-collab) | 협업 서버 — WebSocket(STOMP) 실시간 동기화 |
| [docs](https://github.com/crowfoot-erd/docs) | 요구사항·설계 문서 |
| [crowfoot-manifests](https://github.com/crowfoot-erd/crowfoot-manifests) | Kubernetes 매니페스트 (ArgoCD GitOps) |

## 기술 스택

**프론트엔드** — React 19 · TypeScript · Vite · TanStack Query · Zustand · React Flow · Tailwind CSS · i18next

**백엔드** — Java 21 · Spring Boot 4 · Spring Security(OAuth2 Client) · Spring Data JPA(Hibernate 7) · Querydsl · Spring WebSocket(STOMP)

**데이터베이스** — PostgreSQL · MySQL · Redis

**인프라** — GitHub Actions · GHCR · Kubernetes(Rancher) · ArgoCD

## 서비스

- 웹: <https://crowfoot.java21.net>
- API: <https://crowfoot-api.java21.net>

## 기여

버그 제보와 기여는 언제나 환영합니다. 각 저장소의 Issue를 이용해 주세요.
