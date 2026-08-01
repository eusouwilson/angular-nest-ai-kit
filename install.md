# Instalacao

Nada aqui e instalado como dependencia. Skills e agentes sao arquivos markdown que a
ferramenta de IA le a partir de um diretorio conhecido. Instalar = copiar para esse diretorio.

## Claude Code

```bash
# no repositorio destino
cp -r /caminho/angular-nest-ai-kit/skills/* .claude/skills/
cp -r /caminho/angular-nest-ai-kit/agents/* .claude/agents/
```

- Cada skill precisa ficar em `.claude/skills/<nome>/SKILL.md`, com o `name:` do frontmatter
  igual ao nome da pasta.
- Cada agente e um arquivo solto em `.claude/agents/<nome>.md`.
- `agents/AGENTS.md` e indice, nao um agente: manter fora de `.claude/agents/` ou renomear
  para `README.md` para nao virar uma persona chamada "AGENTS".
- Para instalar para todos os projetos do usuario, usar `~/.claude/` no lugar de `.claude/`.

## Cursor

```bash
cp -r /caminho/angular-nest-ai-kit/skills/* .cursor/skills/
cp -r /caminho/angular-nest-ai-kit/agents/* .cursor/agents/
```

Cursor le regras em `.mdc`. Renomear os agentes e ajustar o frontmatter:

```yaml
---
description: <a mesma description do arquivo>
globs: apps/*/src/**/*.ts       # escopo onde a persona deve ativar
alwaysApply: false
---
```

## Codex, Gemini CLI, outros

A maioria le um arquivo de instrucoes na raiz (`AGENTS.md`, `GEMINI.md`). Duas opcoes:

1. Copiar as pastas e referenciar os caminhos no arquivo de instrucoes.
2. Colar o conteudo das skills mais usadas direto no arquivo de instrucoes (perde a carga
   sob demanda; use so para 2 ou 3 skills).

## Depois de copiar

### 1. Trocar os placeholders

Rodar uma substituicao global no diretorio instalado. Exemplo:

```bash
cd .claude/skills
grep -rl "@org/shared" . | xargs sed -i 's|@org/shared|@acme/contracts|g'
grep -rl "<frontend-app>" . | xargs sed -i 's|<frontend-app>|web|g'
grep -rl "<backend-app>" . | xargs sed -i 's|<backend-app>|api|g'
grep -rl "\*hasPermission" . | xargs sed -i 's|\*hasPermission|*appHasPermission|g'
```

Tabela completa de placeholders no [README.md](README.md).

### 2. Remover o que nao se aplica

- Projeto sem multi-tenant: apagar `tenantId` / `tenant_id` dos templates.
- Projeto com um unico frontend: apagar a secao de RBAC com escopo por aplicacao em
  `skills/rbac-security-auditor/SKILL.md` e a parte de SSO em
  `skills/frontend-auth-bootstrap/SKILL.md`.
- Sem PrimeNG: descartar `angular-primeng-ui`, `angular-primeng-audit` e os templates HTML
  dos dumbs; o resto continua valendo.
- Sem Prisma: descartar `prisma-migration-discipline`.

### 3. Conferir as premissas de infra

As skills assumem que existem no projeto: `BaseService` (backend), `BaseFacade` /
`BaseComponent` / `BasePaginatedListComponent` (frontend), `NotificationService`,
`ErrorHandlingService` e uma diretiva de permissao. Se nao existirem, criar antes ou trocar
as chamadas nos templates. Cada `SKILL.md` lista o que espera.

### 4. Validar com uma feature pequena

Rodar uma entidade simples ponta a ponta (schema, modulo backend, data layer, lista, dialogs,
pagina). O que a IA errar aponta o que ainda esta descrito com o vocabulario do padrao de
origem e nao com o do seu projeto: corrigir no `SKILL.md`, nao no codigo gerado.

## Manutencao

- Este kit e um fork do padrao de um projeto real. Ao evoluir o padrao no seu projeto,
  atualizar o `SKILL.md` correspondente no mesmo commit.
- Regra sem verificacao automatica vira sugestao. As que importam devem ter lint,
  script de checagem ou hook. Ver `skills/ai-operating-protocol/SKILL.md`.
