---
name: angular-primeng-ui
description: Create or refactor Angular + PrimeNG UI following smart/dumb boundaries. Use when working on templates, smart pages, dumb list/create/edit components, p-table, p-dialog, p-select, p-multiselect, p-button, PrimeNG module imports, Angular signal input()/output(), or the dialogVisible close pattern.
---

# Angular + PrimeNG UI

Use esta skill antes de implementar ou refatorar UI Angular com PrimeNG.

## Fontes obrigatorias

- `references/primeng-checklist.md` (nesta skill) para o checklist rapido.
- Se o projeto tiver um doc proprio de templates de UI, ler primeiro e tratar como fonte
  de verdade; esta skill vale como default quando nao houver.

## Fluxo

1. Confirmar escopo: lista, lista server-side, dialog create/edit, grid de campos,
   select/multiselect, botoes de acao ou smart page.
2. Ler a secao correspondente do checklist.
3. Manter a fronteira Smart/Dumb:
   - Smart pages injetam facades, coordenam dialogs, confirmacoes e loading.
   - Dumb components usam `input()`/`output()` e nunca injetam facade/service.
4. Arquivos separados: `.component.ts`, `.component.html`, `.component.scss`.
5. Importar os modulos PrimeNG exatos dos seletores usados.
6. Visibilidade RBAC via diretiva estrutural (`*hasPermission="'resource:action'"`).
7. Dialogs controlados por `@if` no pai: usar `dialogVisible = signal(true)` interno e
   emitir cancel/close apenas no `(onHide)`.
8. Validar rodando o target de teste/build mais focado que existir.

## Regras rigidas

- `ChangeDetectionStrategy.OnPush` em dumb components.
- `input()` / `output()`, nunca `@Input()` / `@Output()`.
- `inject()`, nunca DI por construtor.
- `@if` / `@for`, nunca `*ngIf` / `*ngFor`.
- `SelectModule` + `p-select`; `DropdownModule` esta deprecado desde PrimeNG 18.
- `templateUrl` e `styleUrl`; evitar `template:` / `styles:` inline.
- Nunca chamar `cancel.emit()` ou `close.emit()` direto do X/footer do dialog.
- Nao esconder o botao de submit em form invalido/carregando: desabilitar com `[disabled]`.
- Nao injetar service/facade em dumb component.
- Todo overlay (`p-select`, `p-multiselect`, `p-datepicker`) dentro de dialog ou tabela:
  `appendTo="body"` para evitar clipping.

## Infra de UI que a skill assume

Se o projeto destino tiver equivalentes, usar os dele; se nao tiver, vale criar antes de
espalhar solucao ad-hoc:

- `NotificationService`: wrapper unico de toast. Componente e facade nunca chamam o
  `MessageService` do PrimeNG direto; centralizar e o que permite trocar o mecanismo depois.
- `BaseFacade<T>`: signals `items`, `selectedItem`, `isLoading`, `error` e CRUD comum.
- `BasePaginatedListComponent<T>`: base para lista server-side com `p-table` lazy.
- Servico de layout unico como fonte da verdade do tema (claro/escuro). Nao criar alternador
  de tema paralelo por pagina.
- Servico de loading global com `suppress()` / `restore()`: telas que ja tem skeleton proprio
  suprimem o overlay global no `ngOnInit` e restauram no destroy (`DestroyRef.onDestroy`),
  senao o usuario ve dois indicadores de carregamento ao mesmo tempo.
- Interceptor de erro que trata API fora do ar (`status === 0`) redirecionando para uma rota
  publica dedicada, sem facade nem service de dominio.

## Padrao de topbar

Quando a plataforma tem varios frontends, a topbar segue o mesmo desenho em todos:
titulo/breadcrumb a esquerda; a direita, na ordem, seletores de escopo (visiveis conforme
permissao), acoes especificas do app, notificacoes e avatar. Botao de topbar sempre e o
componente de botao da lib de UI, nunca `<button>` com CSS proprio: icone-only usa
`text rounded` com `aria-label`; acao com label usa `text` e severidade refletindo estado.

Sidebar pode variar por app; a topbar nao.

## Mapa de referencia

- Lista: `p-table`, `TableModule`, toolbar com busca, `p-tag`, botoes de acao.
- Lista server-side: paginacao lazy com `[lazy]="true"` e `(onLazyLoad)`.
- Create/Edit: `p-dialog`, Reactive Forms, `dialogVisible` interno, `ProgressBarModule`.
- Selects: `SelectModule`, `MultiSelectModule`, `appendTo="body"`, `optionLabel`, `optionValue`.
- Smart page: orquestracao de facade, `p-toast`, `p-confirmDialog`, `ConfirmationService`.

## Alinhamento de arquitetura

Estrutura assumida (ajustar se o projeto destino usar outra):

- Frontend: `<dominio>/` com `data-access/`, `feature/`, `ui/`.
- Backend: modulo com `dto/` e services.
- Contratos HTTP publicos vivem na lib compartilhada, nunca duplicados nos dois lados.

Nao introduzir novas camadas de rede ou contratos duplicados sem aprovacao explicita.
