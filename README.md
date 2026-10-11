# angular-nest-ai-kit

![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)

Kit de **18 skills** e **10 agentes** de IA para monorepos **Angular standalone + PrimeNG**
(frontend) e **NestJS + Prisma** (backend), organizados em **Nx**.

Sao instrucoes de processo e templates de codigo, nao uma biblioteca executavel: nada aqui
e instalado como dependencia. Voce copia as pastas para o diretorio que a sua ferramenta de
IA usa (Claude Code, Cursor, Codex e afins) e passa a chamar pelo nome. Ver [install.md](install.md).

## Inicio rapido

```bash
git clone https://github.com/eusouwilson/angular-nest-ai-kit.git

# no repositorio do seu projeto (exemplo Claude Code)
mkdir -p .claude/skills .claude/agents
cp -r ../angular-nest-ai-kit/skills/* .claude/skills/
cp -r ../angular-nest-ai-kit/agents/*.md .claude/agents/
rm .claude/agents/AGENTS.md   # e indice, nao uma persona
```

Depois, trocar os placeholders (`<frontend-app>`, `@org/shared`, etc.) pelos nomes do seu
workspace. Passo a passo em [install.md](install.md).

## Exemplo de uso: uma entidade ponta a ponta

Com o kit instalado, cada skill e acionada por um pedido em linguagem natural. Para uma
entidade `Product`, a sequencia do backend ate a tela fica assim:

| Pedido para a IA | Skill acionada | O que e gerado |
| --- | --- | --- |
| "Crie o modulo backend `products`" | `generating-backend-module` | DTOs, Service (TDD), Controller com Swagger, Module |
| "Gere a camada de dados de `Product`" | `generating-data-layer` | `product.model.ts`, `product.service.ts`, `product.facade.ts` |
| "Gere a listagem de `Product`" | `generating-dumb-list` | componente burro com `p-table` |
| "Gere os dialogs de criar e editar `Product`" | `generating-dumb-create` / `generating-dumb-edit` | dialogs com Reactive Forms |
| "Gere a pagina de `Product`" | `generating-smart-page` | smart component que orquestra facade + dumbs |

Resultado esperado no workspace:

```
apps/<backend-app>/src/products/
  dto/  products.service.ts  products.service.spec.ts
  products.controller.ts  products.module.ts

apps/<frontend-app>/src/app/products/
  data-access/  product.model.ts  product.service.ts  product.facade.ts
  ui/           product-list/  product-create/  product-edit/
  feature/      product-page/
```

Para conduzir tudo com um unico ponto de entrada, chame o agente `architect`: ele decide
quem executa cada etapa seguindo o fluxo `schema-keeper -> backend-implementer ->
backend-quality-guard -> data-access-specialist -> ui-component-engineer ->
frontend-qa-engineer -> security-specialist`.

As skills de padrao e revisao (`angular-primeng-audit`, `rbac-security-auditor`,
`prisma-migration-discipline`, entre outras) entram depois, para auditar o que foi gerado.

## Conteudo

```
angular-nest-ai-kit/
├── skills/     # 18 skills: geracao de codigo e padroes de revisao
├── agents/     # 10 personas especializadas
└── install.md  # como instalar em Claude Code, Cursor, Codex e afins
```

## Skills de geracao de codigo

| Skill | Stack | O que faz |
| --- | --- | --- |
| [generating-backend-module](skills/generating-backend-module/SKILL.md) | NestJS + Prisma | Modulo completo: DTOs, Service, Controller, Module, specs |
| [generating-data-layer](skills/generating-data-layer/SKILL.md) | Angular | Model + Service HTTP + Facade com signals |
| [generating-dumb-list](skills/generating-dumb-list/SKILL.md) | Angular + PrimeNG | Componente burro de listagem (p-table) |
| [generating-dumb-create](skills/generating-dumb-create/SKILL.md) | Angular + PrimeNG | Dialog de criacao com Reactive Forms |
| [generating-dumb-edit](skills/generating-dumb-edit/SKILL.md) | Angular + PrimeNG | Dialog de edicao com patchValue reativo |
| [generating-smart-page](skills/generating-smart-page/SKILL.md) | Angular | Pagina que orquestra facade + dumbs |

## Skills de padrao e revisao

| Skill | Escopo | O que cobre |
| --- | --- | --- |
| [angular-feature-architecture](skills/angular-feature-architecture/SKILL.md) | frontend | Estrutura feature-sliced, nomenclatura, rotas, breadcrumb, anti-patterns |
| [angular-primeng-ui](skills/angular-primeng-ui/SKILL.md) | frontend | Regras de UI, smart/dumb, dialogs, topbar, infra de UI |
| [angular-primeng-audit](skills/angular-primeng-audit/SKILL.md) | frontend | Auditoria de conformidade de componentes existentes |
| [angular-testing-patterns](skills/angular-testing-patterns/SKILL.md) | frontend | TDD, mock de facade com signals, o que cobrir e o que nao |
| [frontend-auth-bootstrap](skills/frontend-auth-bootstrap/SKILL.md) | frontend | SSO entre apps, inicializacao sem race condition, permissoes no boot |
| [nestjs-api-standards](skills/nestjs-api-standards/SKILL.md) | backend | Envelope de resposta, checklist Swagger, erro, paginacao |
| [prisma-migration-discipline](skills/prisma-migration-discipline/SKILL.md) | backend | Migrations, comandos proibidos, shadow DB, performance |
| [rbac-security-auditor](skills/rbac-security-auditor/SKILL.md) | backend | Auditoria RBAC com protocolo de confirmacao |
| [shared-contracts-library](skills/shared-contracts-library/SKILL.md) | monorepo | O que entra na lib de contratos e o que nunca entra |
| [monorepo-workspace-standards](skills/monorepo-workspace-standards/SKILL.md) | monorepo | Libs, aliases, cache, affected, Docker, CI |
| [commit-and-branch-conventions](skills/commit-and-branch-conventions/SKILL.md) | processo | Conventional Commits, nome de branch, fluxo ate producao |
| [ai-operating-protocol](skills/ai-operating-protocol/SKILL.md) | processo | Precedencia de regras, guardrails, enforcement automatico |

## Agentes

| Agente | Dominio |
| --- | --- |
| [architect](agents/architect.md) | orquestracao e delegacao |
| [schema-keeper](agents/schema-keeper.md) | Prisma, migrations, queries |
| [backend-implementer](agents/backend-implementer.md) | modules, controllers, services, DTOs |
| [backend-quality-guard](agents/backend-quality-guard.md) | specs backend, code review, Swagger |
| [data-access-specialist](agents/data-access-specialist.md) | model, service, facade no frontend |
| [ui-component-engineer](agents/ui-component-engineer.md) | smart/dumb components, templates |
| [frontend-qa-engineer](agents/frontend-qa-engineer.md) | specs Angular (Jasmine/Karma) |
| [security-specialist](agents/security-specialist.md) | RBAC, guards, validacao, exposicao de dados |
| [monorepo-specialist](agents/monorepo-specialist.md) | workspace, libs, build, cache, CI |
| [docs-master](agents/docs-master.md) | Swagger, READMEs, docs tecnicas |

Indice completo com fluxo full-stack em [agents/AGENTS.md](agents/AGENTS.md).

## Placeholders

Nada aqui referencia um projeto real. Trocar antes de usar:

| Placeholder | Significado |
| --- | --- |
| `<frontend-app>` / `<backend-app>` | nomes dos apps no workspace |
| `@org/shared` | lib de contratos compartilhados (TypeScript puro) |
| `@org/auth-contracts` | lib de contratos de auth e permissao |
| `{Entity}` / `{kebab}` / `{model}` | entidade em PascalCase / kebab-case / nome no Prisma |
| `tenantId` / `tenant_id` | escopo multi-tenant opcional (ou remover) |
| `JwtAuthGuard` / `PermissionsGuard` | guards de autenticacao e autorizacao |
| `*hasPermission` | diretiva estrutural de RBAC no template |
| `ApiResponse<T>` | envelope padrao de resposta HTTP |
| `BaseService` / `BaseFacade` / `BaseComponent` | classes base do projeto |
| `NotificationService` | wrapper unico de toast |

Tabela detalhada com exemplos em [skills/README.md](skills/README.md).

## Premissas

- **Frontend**: Angular standalone, Signals, `inject()`, `@if`/`@for`, PrimeNG.
  Camadas por dominio: `data-access/`, `ui/`, `feature/`, `utils/`.
- **Backend**: NestJS, Prisma, DTOs com `class-validator`, Swagger completo, resposta sempre
  em `ApiResponse<T>`, banco em `snake_case` e API em `camelCase`.
- **Contratos publicos** nascem na lib compartilhada, nunca duplicados entre front e back.
- **RBAC** no formato `recurso:acao`, autorizacao centralizada em guard.
- **Monorepo** Nx com dependencia unidirecional app para lib.

Projeto que diverge de alguma premissa: ajustar o `SKILL.md` correspondente antes de usar.
As regras marcadas como obrigatorias sao opinioes explicitas deste padrao, nao verdades
universais.

## Adaptando para outra stack

- **Sem PrimeNG**: as skills `angular-primeng-*` e os templates de dialog assumem a lib.
  A estrutura smart/dumb, o padrao de facade e as regras de arquitetura seguem valendo.
- **Sem Prisma**: `prisma-migration-discipline` vira especifica demais; o restante do backend
  (envelope, Swagger, erro, RBAC) e agnostico de ORM.
- **Sem Nx**: `monorepo-workspace-standards` perde a parte de cache e affected; libs,
  aliases e Docker continuam aplicaveis.

## Licenca

[MIT](LICENSE).
