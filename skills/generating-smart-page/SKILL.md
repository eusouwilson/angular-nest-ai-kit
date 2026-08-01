---
name: generating-smart-page
description: Gera Smart Component (feature page) Angular que orquestra Facade e dumb components list/create/edit, com toast, confirmDialog, effect() para dados dependentes de escopo e dialogs montados por @if. Use quando o usuario pede "crie a pagina", "smart component", "feature page", "tela de listagem completa" ou "generate-smart-page".
---

# Gerando Smart Page Component

## Inicio rapido

Pedir o nome da entidade em PascalCase. Gera em `feature/`:

```
apps/<frontend-app>/src/app/{dominio}/feature/{kebab}-page/
  {kebab}-page.component.ts
  {kebab}-page.component.html
  {kebab}-page.component.scss   <- vazio
```

HTML e SCSS em arquivos separados, igual aos dumbs. Se o projeto de origem usava template
inline em smart pages, essa excecao foi removida de proposito: um unico padrao para todos.

## Template

Ver [TEMPLATE](TEMPLATE.md).

## Responsabilidade da smart page

Faz:
- injetar facades e services;
- decidir quando carregar dados (`effect()` para dependencias reativas);
- abrir/fechar dialogs;
- confirmar acoes destrutivas;
- traduzir evento do dumb em chamada de facade.

Nao faz:
- HTTP direto;
- guardar estado que pertence a facade;
- formatar dados que o dumb consegue formatar sozinho.

## Regras obrigatorias

- `providers: [ConfirmationService, MessageService]` no `@Component` (escopo por pagina).
- `@if (facade.createDialogOpen())` para montar/desmontar dialogs: garante form limpo a cada
  abertura e faz o padrao `dialogVisible` dos dumbs funcionar.
- `effect()` para recarregar quando um escopo reativo muda (tenant ativo, filtro global).
- `inject()` para todas as dependencias.
- `p-toast` e `p-confirmDialog` presentes no template.
- Sem titulo de pagina no toolbar quando o layout ja mostra breadcrumb.

## Pre-requisitos

Antes da smart page, gerar:
- [generating-data-layer](../generating-data-layer/SKILL.md) (facade com signals e CRUD)
- [generating-dumb-list](../generating-dumb-list/SKILL.md)
- [generating-dumb-create](../generating-dumb-create/SKILL.md)
- [generating-dumb-edit](../generating-dumb-edit/SKILL.md)

## Depois de gerar, adaptar

1. Remover o facade de escopo (tenant) se a entidade for global.
2. Remover o componente de edicao se a entidade so tiver create/list.
3. Ajustar `onToggleStatus` se a exclusao for fisica em vez de logica.
4. Registrar a rota com `loadComponent` e o metadado de breadcrumb que o layout exigir.
