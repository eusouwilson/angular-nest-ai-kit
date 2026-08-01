---
name: angular-feature-architecture
description: Estrutura feature-sliced, nomenclatura e anti-patterns para apps Angular standalone com Signals. Use ao criar um dominio novo no frontend, decidir onde um arquivo mora, revisar estrutura de pastas ou refatorar codigo legado para o padrao atual.
---

# Arquitetura de Feature no Angular

Regras de organizacao para apps Angular standalone com Signals. Independem de biblioteca
de UI; complementam as skills de geracao de codigo.

## Estrutura obrigatoria

```
{contexto}/{dominio}/
├── data-access/    # facades, services HTTP, models
├── ui/             # dumb components
├── feature/        # smart components (pages)
└── utils/          # helpers, pipes, validators
```

Agrupar por dominio, nao por tipo de arquivo. Uma pasta `components/` global com 80 itens
e o sintoma classico do agrupamento errado.

## Nomenclatura

| Item | Regra | Exemplo |
| --- | --- | --- |
| Arquivo | `kebab-case` | `product-list.component.ts` |
| Classe | `PascalCase` | `ProductListComponent` |
| Interface | prefixo `I` | `IProduct` |
| Facade | `{entidade}.facade.ts` | `product.facade.ts` |
| Service | `{entidade}.service.ts` | `product.service.ts` |
| Model | `{entidade}.model.ts` | `product.model.ts` |
| Smart | `{entidade}-page.component.ts` | `product-page.component.ts` |
| Dumb | `{entidade}-list|create|edit.component.ts` | `product-edit.component.ts` |

- HTML e SCSS sempre em arquivos separados (`templateUrl`, `styleUrl`), o `.scss` mesmo vazio.
- Nunca sufixo `-form`. Duas variantes aceitas para CRUD, escolher uma e manter no app inteiro:
  - **Unificada**: `{entidade}-list/` + `{entidade}-editor/` (um componente cobre criar e editar).
  - **Separada**: `{entidade}-list/` + `{entidade}-create/` + `{entidade}-edit/`.

  A unificada reduz duplicacao quando o formulario e o mesmo nos dois modos; a separada
  fica melhor quando criar e editar tem campos e validacoes bem diferentes.

## Estado

- Signals para estado local e de feature. Biblioteca de store global so quando houver
  necessidade real de estado compartilhado complexo, nao por default.
- Facade e a fronteira: componente le signal da facade, chama metodo da facade.
- Signal publico sempre readonly; escrita so dentro da facade.
- `computed()` para derivado; nunca recalcular no template.
- `effect()` para reagir a mudanca de dependencia, nao para orquestrar fluxo de negocio.

## Classes base

Vale ter tres bases no `shared/`, cada uma com uma responsabilidade:

- `BaseFacade<T>`: signals `items`, `selectedItem`, `isLoading`, `error` e CRUD comum.
- `BaseComponent`: signals de `loading`/`error` e helpers de notificacao/destroy.
- `BasePaginatedListComponent<T>`: lista server-side com tabela lazy.

Heranca aqui e para eliminar repeticao real. Se a base comeca a ganhar `if` por caso
particular, o caso particular nao pertence a base.

## Rotas e breadcrumb

- Lazy com `loadComponent`.
- Guard de autenticacao/acesso na rota, nao dentro do componente.
- **Toda rota de pagina carrega `data: { breadcrumb: 'Label' }`.** Um componente global de
  breadcrumb le esse metadado (inclusive em rotas aninhadas, via recursao no `firstChild`)
  e monta a trilha sozinho.
- Label do breadcrumb: substantivo no plural, no idioma da interface.
- Rotas de erro (`access-denied`, `not-found`) e `redirectTo` dispensam breadcrumb.
- Nunca repetir o label como titulo dentro da pagina: vira duplicidade visual.

## Robustez de dados

- Todo `.sort()` que faz parsing de string precisa de guarda contra `null`/`undefined`:

  ```typescript
  // certo
  const [mes = 0, ano = 0] = (a.periodo ?? '').split('-').map(Number);

  // errado: quebra com "Cannot read properties of undefined (reading 'split')"
  const [mes, ano] = a.periodo.split('-').map(Number);
  ```

- Dado vindo de API e opcional ate prova em contrario, mesmo quando o tipo diz que nao e.

## Anti-patterns

- Injecao por construtor em componente standalone (usar `inject()`).
- Componente chamando service HTTP direto (passar pela facade).
- `*ngIf` / `*ngFor` (usar `@if` / `@for` com `track`).
- Facade injetada em dumb component.
- Titulo da pagina repetido no toolbar quando o layout ja mostra breadcrumb.
- `any` em retorno de chamada HTTP.
- Logica de formatacao duplicada em varios templates (extrair pipe em `utils/`).
- Estado derivado guardado em signal proprio em vez de `computed()`: dessincroniza.

## Checklist de dominio novo

- [ ] Pastas `data-access/`, `ui/`, `feature/` criadas
- [ ] Contrato publico vindo da lib compartilhada, nao redeclarado
- [ ] Facade com signals readonly e loading controlado
- [ ] Dumbs com `OnPush`, `input()`/`output()`, sem injecao
- [ ] Smart page orquestrando dialogs com `@if`
- [ ] Rota lazy com guard e breadcrumb
- [ ] Specs cobrindo estados vazio, carregando e erro
