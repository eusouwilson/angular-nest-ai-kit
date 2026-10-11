# Skills Genericas (Angular + NestJS)

Pacote de skills desacopladas de qualquer projeto especifico, extraidas de um monorepo
Nx com Angular standalone + PrimeNG no frontend e NestJS + Prisma no backend.

Copiar a pasta da skill para `.claude/skills/<nome>/` (ou `.ai/skills/`, `.cursor/skills/`)
de outro repositorio. Personas correspondentes em [agents/](../agents/AGENTS.md).

## Skills de geracao de codigo

| Skill | Stack | O que faz |
| --- | --- | --- |
| [generating-backend-module](generating-backend-module/SKILL.md) | NestJS + Prisma | Modulo completo: DTOs, Service, Controller, Module, specs |
| [generating-data-layer](generating-data-layer/SKILL.md) | Angular | Model + Service HTTP + Facade com signals |
| [generating-dumb-list](generating-dumb-list/SKILL.md) | Angular + PrimeNG | Componente burro de listagem (p-table) |
| [generating-dumb-create](generating-dumb-create/SKILL.md) | Angular + PrimeNG | Dialog de criacao com Reactive Forms |
| [generating-dumb-edit](generating-dumb-edit/SKILL.md) | Angular + PrimeNG | Dialog de edicao com patchValue reativo |
| [generating-smart-page](generating-smart-page/SKILL.md) | Angular | Pagina que orquestra facade + dumbs |

## Skills de padrao e revisao

| Skill | Escopo | O que cobre |
| --- | --- | --- |
| [angular-feature-architecture](angular-feature-architecture/SKILL.md) | frontend | Estrutura feature-sliced, nomenclatura, rotas, breadcrumb, anti-patterns |
| [angular-primeng-ui](angular-primeng-ui/SKILL.md) | frontend | Regras de UI, smart/dumb, dialogs, topbar, infra de UI |
| [angular-primeng-audit](angular-primeng-audit/SKILL.md) | frontend | Auditoria de conformidade de componentes existentes |
| [angular-testing-patterns](angular-testing-patterns/SKILL.md) | frontend | TDD, mock de facade com signals, o que cobrir e o que nao |
| [frontend-auth-bootstrap](frontend-auth-bootstrap/SKILL.md) | frontend | SSO entre apps, inicializacao sem race condition, permissoes no boot |
| [nestjs-api-standards](nestjs-api-standards/SKILL.md) | backend | Envelope de resposta, checklist Swagger, erro, paginacao |
| [prisma-migration-discipline](prisma-migration-discipline/SKILL.md) | backend | Migrations, comandos proibidos, shadow DB, performance |
| [rbac-security-auditor](rbac-security-auditor/SKILL.md) | backend | Auditoria RBAC com protocolo de confirmacao |
| [shared-contracts-library](shared-contracts-library/SKILL.md) | monorepo | O que entra na lib de contratos e o que nunca entra |
| [monorepo-workspace-standards](monorepo-workspace-standards/SKILL.md) | monorepo | Libs, aliases, cache, affected, Docker, CI |
| [commit-and-branch-conventions](commit-and-branch-conventions/SKILL.md) | processo | Conventional Commits, nome de branch, fluxo ate producao |
| [ai-operating-protocol](ai-operating-protocol/SKILL.md) | processo | Precedencia de regras, guardrails, enforcement automatico |

## Convencoes de placeholder

Todo placeholder abaixo precisa ser trocado pelo equivalente do projeto destino.

| Placeholder | Significado | Exemplo real |
| --- | --- | --- |
| `<frontend-app>` | nome do app Angular | `web`, `admin-portal` |
| `<backend-app>` | nome do app NestJS | `api`, `server` |
| `@org/shared` | lib de contratos compartilhados (TS puro) | `@acme/shared` |
| `@org/auth-contracts` | lib de contratos de auth/permissao | `@acme/auth` |
| `{Entity}` | entidade em PascalCase | `Product` |
| `{kebab}` | mesma entidade em kebab-case | `product` |
| `{model}` | nome do model no Prisma | `product` |
| `tenantId` / `tenant_id` | escopo multi-tenant opcional | `companyId`, `orgId` ou remover |
| `JwtAuthGuard` | guard de autenticacao | `AuthGuard('jwt')`, guard proprio |
| `PermissionsGuard` | guard de autorizacao RBAC | guard proprio |
| `*hasPermission` | diretiva estrutural de RBAC | `*appHasPermission` |
| `ApiResponse<T>` | envelope padrao de resposta | `{ message, statusCode, type, data }` |
| `BaseService` | base do backend (prisma + logger + erro) | criar se nao existir |
| `BaseFacade` / `BaseComponent` | bases do frontend | criar se nao existir |
| `NotificationService` | wrapper unico de toast | wrapper sobre `MessageService` |

## Premissas de arquitetura

- **Frontend**: Angular standalone, Signals, `inject()`, `@if`/`@for`, PrimeNG.
  Camadas por dominio: `data-access/`, `ui/`, `feature/`, `utils/`.
- **Backend**: NestJS, Prisma, DTOs com `class-validator`, Swagger completo, resposta sempre
  em `ApiResponse<T>`, banco em `snake_case` e API em `camelCase`.
- **Contratos publicos** nascem na lib compartilhada, nunca duplicados entre front e back.
- **RBAC** no formato `recurso:acao`, autorizacao centralizada em guard.
- **Monorepo** Nx com dependencia unidirecional app -> lib.

Projeto destino que diverge de alguma premissa: ajustar o SKILL.md antes de usar. As regras
"obrigatorias" das skills sao opinioes explicitas do padrao de origem, nao verdades universais.

## O que foi removido em relacao ao original

- Nome e marca do projeto de origem, paths internos de documentacao e nomes de dominio de negocio.
- Aliases de libs proprietarias, guards de auth especificos e IDs de app.
- Regras amarradas ao modelo de RBAC em duas camadas do projeto original, substituidas por
  uma versao generica e opcional.
- Referencias a comandos e targets especificos do workspace de origem.
