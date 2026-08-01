# PrimeNG Audit Checklist

Checklist da skill [angular-primeng-audit](../SKILL.md).

## Achados de risco alto

- Import de `DropdownModule` ou uso de `<p-dropdown>`. Corrigir com `SelectModule` e `<p-select>`.
- Dialog controlado por `@if` no pai emitindo `cancel`/`close` direto do X ou footer.
  Corrigir com `dialogVisible` interno e `(onHide)`. Sintoma: backdrop travado na tela.
- Dumb component injetando facade/service ou fazendo HTTP. Mover orquestracao para a smart page.
- `template:` ou `styles:` inline em componente novo/refatorado. Separar em `.html`/`.scss`.
- Acao de create/edit/delete/toggle sem diretiva de permissao no template.
- Select/multiselect dentro de dialog ou tabela sem `appendTo="body"` (overlay cortado).

## Achados de risco medio

- Falta de `ChangeDetectionStrategy.OnPush` em dumb component.
- `@Input()` / `@Output()` em dumb novo/refatorado no lugar de `input()` / `output()`.
- DI por construtor no lugar de `inject()`.
- `*ngIf` / `*ngFor` em template atualizado no lugar de `@if` / `@for`.
- Binding de signal sem chamada: `[loading]="isLoading"` em vez de `[loading]="isLoading()"`.
- Botao de submit escondido em vez de desabilitado para form invalido/carregando.
- Tabela sem `dataKey`, sem `trackBy`/`track`, ou com `colspan` do `emptymessage` errado.

## Mapa de imports PrimeNG

| Seletor | Modulo |
| --- | --- |
| `p-table` | `TableModule` |
| `p-dialog` | `DialogModule` |
| `p-button` | `ButtonModule` |
| `pInputText` | `InputTextModule` |
| `pTextarea` | `TextareaModule` |
| `p-select` | `SelectModule` |
| `p-multiselect` | `MultiSelectModule` |
| `p-checkbox` | `CheckboxModule` |
| `p-tag` | `TagModule` |
| `pTooltip` | `TooltipModule` |
| `p-progressBar` | `ProgressBarModule` |
| `p-toolbar` | `ToolbarModule` |
| `p-confirmDialog` | `ConfirmDialogModule` |
| `p-toast` | `ToastModule` |
| `p-iconfield` | `IconFieldModule` |
| `p-inputicon` | `InputIconModule` |
