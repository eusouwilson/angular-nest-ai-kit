# Agentes Genericos (Angular + NestJS)

Personas de IA desacopladas de qualquer projeto especifico, para monorepos que seguem
o padrao **Angular standalone + Signals + PrimeNG** no frontend e **NestJS + Prisma** no backend.

Copiar cada arquivo para `.claude/agents/<nome>.md` (Claude Code), `.cursor/agents/`
(Cursor, renomeando para `.mdc`) ou equivalente da ferramenta usada.

## Elenco

| Agente | Dominio | Aciona quando |
| --- | --- | --- |
| [architect](architect.md) | orquestracao | ponto de entrada; decide quem executa |
| [schema-keeper](schema-keeper.md) | Prisma, migrations, queries | mudanca de schema, tabela nova, query lenta |
| [backend-implementer](backend-implementer.md) | NestJS: modules, controllers, services, DTOs | endpoint ou regra de negocio nova |
| [backend-quality-guard](backend-quality-guard.md) | specs backend, code review, Swagger | teste de service, revisao de PR backend |
| [data-access-specialist](data-access-specialist.md) | Angular: model, service, facade | camada de dados e estado do frontend |
| [ui-component-engineer](ui-component-engineer.md) | Angular + PrimeNG: smart/dumb, templates | pagina, lista, dialog, refactor de template |
| [frontend-qa-engineer](frontend-qa-engineer.md) | specs Angular (Jasmine/Karma) | teste de componente |
| [security-specialist](security-specialist.md) | RBAC, guards, permissoes, validacao | seguranca full-stack |
| [monorepo-specialist](monorepo-specialist.md) | workspace, libs, build, cache, CI | configuracao de monorepo e dependencias |
| [docs-master](docs-master.md) | Swagger, READMEs, docs tecnicas | documentacao desatualizada |

## Fluxo full-stack

```
schema-keeper -> backend-implementer -> backend-quality-guard
   -> data-access-specialist -> ui-component-engineer -> frontend-qa-engineer
   -> security-specialist (valida ponta a ponta)
```

Nao pular etapas para "ganhar tempo": o custo aparece no review.

## Convencoes assumidas

Todo agente foi escrito com estes placeholders. Trocar antes de usar:

| Placeholder | Significado |
| --- | --- |
| `<frontend-app>` / `<backend-app>` | nomes dos apps no workspace |
| `@org/shared` | lib de contratos compartilhados (TypeScript puro) |
| `@org/auth-contracts` | lib de contratos de autenticacao/permissao |
| `JwtAuthGuard` / `PermissionsGuard` | guards de auth e autorizacao |
| `*hasPermission` | diretiva estrutural de RBAC no template |
| `ApiResponse<T>` | envelope padrao de resposta HTTP |
| `BaseService` / `BaseFacade` | classes base do projeto |

## Relacao com as skills

Os agentes dizem **quem faz e com que criterio**; as skills em [skills/](../skills/README.md)
dizem **como gerar o codigo**. Um agente sem as skills correspondentes ainda funciona,
mas repete decisao de template a cada tarefa.
