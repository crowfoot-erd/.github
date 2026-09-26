# Crowfoot

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

**[한국어](./README.md)** | **[English](./README.en.md)** | **[日本語](./README.ja.md)** | **简体中文**

**设计完成之时，就是数据库开始之时。**

Crowfoot 是一个在浏览器中运行的开源 ERD 编辑器。从逻辑建模到物理模型转换、团队协作，再到真实数据库的开通 — 数据工作的全程都在一个地方完成。Crowfoot 这个名字来源于**鸦脚(Crow's Foot)记法** — 用形似乌鸦脚印的符号来表示 ERD 中表之间关系的记法。

## 服务

| 类别 | 地址 |
| --- | --- |
| 网站(编辑器 · 仪表板) | https://crowfoot.java21.net |
| API 网关 | https://crowfoot-api.java21.net |
| 协作 WebSocket 服务器 | ws://crowfoot-ws.java21.net |
| 系统 ERD | https://crowfoot.java21.net/share/1KeFkNED0uTmx6MPWqmXph |

Crowfoot 系统自身的 ERD 也是用 Crowfoot 设计的 — 可以在上面的分享链接中查看。

## 免费托管数据库

Crowfoot 的核心功能。**每个账户最多可免费开通 5 个** PostgreSQL · MySQL 开发用数据库。

- 开通时**自动创建专用数据库账户** — 仅拥有 schema 级权限，实例 root 凭证不会在任何地方暴露
- 发放的密码使用 AES-256-GCM 加密存储，只有所有者才能查看连接信息
- 开通后立即进行连接测试，可以从任何外部客户端直接使用，不需要时可以随时撤销
- 面向开发、学习和测试用途的数据库

## 功能

### 浏览器 ERD 编辑器

- 无需安装 — 直接在浏览器中用鸦脚记法绘制表和关系
- **逻辑模型与物理模型并存** — 逻辑名、物理名并排显示，通用逻辑类型自动映射为各 DBMS 的物理类型
- **5 种 DBMS 模板** — PostgreSQL、MySQL、Oracle、MSSQL(+通用)，各自反映其类型、自增和注释语法
- 模型校验(名称重复、引用完整性等)、自动布局、按文档记忆视口(缩放 · 平移)
- 大型文档(100 张表)下依然流畅的 60fps 平移与缩放

### SQL 生成、部署与逆向工程

- 将文档转换为**对应 DBMS 的 DDL 脚本** — 预览、复制或下载(表注释跟随逻辑名)
- **正向工程** — 将生成的 DDL 直接部署到已开通或已登记的数据库
- **逆向工程** — 将现有数据库的 schema 和注释读回为 ERD 文档

### 实时协作

- 基于 WebSocket(STOMP)— 多人同时编辑同一文档，实时在线状态与变更同步
- 可以在文档上留下评论

### 工作空间与团队角色

- 用工作空间组织文档，邀请用户或团队
- 基于角色的权限 — **Owner、Editor、Commenter、Viewer** — 保障团队协作安全

### 接入你自己的数据库

- 保存现有数据库的连接配置(加密)，一键测试连接
- 托管数据库与个人数据库在同一个列表中管理

### 认证与管理

- **GitHub · Google OAuth2 登录** — 访问令牌保存在内存中，刷新令牌放在 `SameSite=Strict` Cookie 中，网关对每个请求进行 introspection 校验
- 管理控制台 — 管理用户、代码表、托管数据库实例、开通配额和审计日志

## 架构

五个服务，一个微服务架构。

```
  Browser
    │ HTTPS                          │ WebSocket (STOMP)
    ▼                                ▼
┌──────────────┐              ┌──────────────┐
│ crowfoot-web │              │crowfoot-collab│  presence · live sync
│  React SPA   │              └──────────────┘
└──────┬───────┘
       ▼
┌──────────────┐     ┌──────────────┐
│ api-gateway  │────▶│  auth        │  OAuth2 · JWT · introspection · Redis
└──────┬───────┘     └──────────────┘
       │             ┌──────────────┐
       └────────────▶│  core-api    │  domain (PostgreSQL) · managed DB
                     └──────┬───────┘  provisioning (PostgreSQL · MySQL)
```

## 仓库

| 仓库 | 说明 |
| --- | --- |
| [crowfoot-web](https://github.com/crowfoot-erd/crowfoot-web) | 前端 — React SPA。ERD 编辑器(React Flow)、仪表板 · 工作空间 · 团队、管理控制台、协作客户端(STOMP)、国际化(ko·en·ja·zh)· 深色模式 |
| [crowfoot-api-gateway](https://github.com/crowfoot-erd/crowfoot-api-gateway) | API 网关 — Spring Cloud Gateway。路由、Bearer 令牌 introspection 校验、用户身份头注入、公开路径白名单 |
| [crowfoot-auth](https://github.com/crowfoot-erd/crowfoot-auth) | 认证服务器 — OAuth2 登录(GitHub · Google · PKCE)、JWT 签发 · 刷新 · introspection、Redis 黑名单(登出) |
| [crowfoot-core-api](https://github.com/crowfoot-erd/crowfoot-core-api) | 核心 API — 用户 · 工作空间 · 团队 · 模型文档 · 评论、托管数据库开通(专用账户的发放与撤销)、SQL 生成 · 部署 · 逆向工程、代码表 · 审计日志 |
| [crowfoot-collab](https://github.com/crowfoot-erd/crowfoot-collab) | 协作服务器 — WebSocket(STOMP)。按文档的在线状态与编辑变更的实时广播，单实例部署 |

## 版本发布

| 版本 | 日期 | 主要内容 | 标签 | 发布说明 |
| --- | --- | --- | --- | --- |
| v1.17 | 2026-09-26 | 协作增强：实时光标·选区·移动、同时编辑收敛、编辑锁、版本冲突解决 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.17) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.17) · [collab](https://github.com/crowfoot-erd/crowfoot-collab/releases/tag/v1.17) | [查看](https://crowfoot.java21.net/release-notes/19) |
| v1.16 | 2026-09-25 | 全面支持 4 种语言(韩语、英语、日语、中文)、语言专属 URL·SEO、账号语言、多语言发布说明 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.16) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.16) | [查看](https://crowfoot.java21.net/release-notes/18) |
| v1.15 | 2026-09-25 | 系统词典批量扩充(34,075 个词条)、推理加载优化 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.15) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.15) | [查看](https://crowfoot.java21.net/release-notes/17) |
| v1.14 | 2026-09-25 | 术语词典面板 · 系统词典管理、推理语言选择、列物理名词典建议 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.14) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.14) | [查看](https://crowfoot.java21.net/release-notes/16) |
| v1.13 | 2026-09-24 | 逻辑分组(主题区域)、逻辑名自动推理、快捷键速查表 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.13) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.13) | [查看](https://crowfoot.java21.net/release-notes/15) |
| v1.12 | 2026-09-23 | 模型浏览器 · 统一搜索、SQL 导入 | [web](https://github.com/crowfoot-erd/crowfoot-web/releases/tag/v1.12) · [core](https://github.com/crowfoot-erd/crowfoot-core-api/releases/tag/v1.12) | [查看](https://crowfoot.java21.net/release-notes/14) |
| v1.11 | 2026-09-22 | 关系编辑 · 版本比较改进、更快的图片导出 | — | [查看](https://crowfoot.java21.net/release-notes/13) |
| v1.10 | 2026-09-21 | 版本比较 · 迁移 DDL | — | [查看](https://crowfoot.java21.net/release-notes/11) |
| v1.09 | 2026-09-20 | 文档版本历史 · 数据库同步 | — | [查看](https://crowfoot.java21.net/release-notes/10) |
| v1.08 | 2026-09-18 | 社区板块 · 实时聊天 · 编辑器安全防护 | — | [查看](https://crowfoot.java21.net/release-notes/9) |

每个版本的变更内容都会以[发布说明](https://crowfoot.java21.net/)的形式公开(首页的"最近发布"中有完整列表)。git 标签从 v1.12 起保留在各仓库中。

## 自行运行

### 前置条件

| 工具 | 版本 | 用途 |
| --- | --- | --- |
| Java (Temurin) | 21 | 四个服务器 |
| Maven | 3.9+ | 构建和运行服务器 |
| Node.js / pnpm | 20 / 10 | 前端 |
| PostgreSQL | 16+ | 领域数据库(`crowfoot` 数据库 + `crowfoot_core` schema) |
| Redis | 6+ | 认证登出黑名单 |

Schema 不会自动创建(`ddl-auto: none`)— 请使用 DDL 脚本初始化。

### 本地端口

| 服务 | 端口 | 备注 |
| --- | --- | --- |
| crowfoot-web (Vite) | 8080 | 将 `/api` 代理到网关(8000) |
| crowfoot-api-gateway | 8000 | 路由到 `auth`(8081)和 `core-api`(8082) |
| crowfoot-auth | 8081 | |
| crowfoot-core-api | 8082 | |
| crowfoot-collab | 8083 | WebSocket(STOMP)— 不经过网关直接连接 |

### 环境变量

每个服务器会自动读取仓库根目录下的 `.env-local` 文件(已 gitignore)— 复制 `.env-local.example` 并填入数值。网关和 collab 不需要环境变量。

**crowfoot-auth**

| 变量 | 说明 |
| --- | --- |
| `CROWFOOT_AUTH_JWT_SECRET` | JWT HS256 签名密钥 — Base64,32 字节以上(`openssl rand -base64 48`) |
| `CROWFOOT_AUTH_FLOW_SECRET` | `auth_flow` Cookie 的 HMAC-SHA256 密钥 — 与 JWT 密钥分开保管 |
| `CROWFOOT_AUTH_GITHUB_CLIENT_ID` / `..._SECRET` | GitHub OAuth 应用凭证 |
| `CROWFOOT_AUTH_GOOGLE_CLIENT_ID` / `..._SECRET` | Google OAuth 客户端凭证(PKCE) |
| `CROWFOOT_REDIS_PASSWORD` / `CROWFOOT_REDIS_DATABASE` | Redis 黑名单连接(主机在 `application-local.yml` 中) |

请在 OAuth 应用中注册回调 URI `http://localhost:8080/auth/callback`。

**crowfoot-core-api**

| 变量 | 说明 |
| --- | --- |
| `DB_URL` | PostgreSQL JDBC URL — `jdbc:postgresql://{host}:5432/crowfoot?currentSchema=crowfoot_core` |
| `DB_USERNAME` / `DB_PASSWORD` | 领域数据库账户 |

**crowfoot-web** — 开发环境使用默认值即可(`VITE_API_BASE_URL` 留空 → Vite 代理保持同源)。生产构建时注入 `VITE_API_BASE_URL`(API 网关)和 `VITE_WS_URL`(协作 WS)。

### 启动

```bash
# 四个服务器 — 在各仓库中运行(local 配置为默认)
mvn spring-boot:run

# 前端
pnpm install
pnpm dev        # http://localhost:8080
```

### 生产环境

生产环境中五个服务全部以容器镜像运行,服务器端口统一为 8080。密钥只存在于环境变量中 — 绝不写入代码或镜像。各仓库的 `application-prod.yml` 中列出了所需的变量。

## 技术栈

**前端** — React 19 · TypeScript · Vite · TanStack Query · Zustand · React Flow · ELK(自动布局) · Tailwind CSS · shadcn/ui (radix-ui) · i18next · Vitest·Testing Library·Playwright·MSW

**后端** — Java 21 · Spring Boot 4 · Spring Security (OAuth2 Client) · Spring Cloud Gateway · Spring Data JPA (Hibernate 7) · Querydsl · Spring WebSocket (STOMP)

**数据库** — PostgreSQL(领域 · 托管开通) · MySQL(托管开通) · Redis(认证会话)

**基础设施** — GitHub Actions · GHCR · Kubernetes (Rancher) · ArgoCD (GitOps)

## 许可证与贡献

所有仓库均以 [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0) 分发。欢迎提交 Bug 报告和贡献 — 请使用各仓库的 Issues。
