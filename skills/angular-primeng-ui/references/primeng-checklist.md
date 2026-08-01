# PrimeNG UI Checklist

Checklist rapido da skill [angular-primeng-ui](../SKILL.md).

## Regras de componente

- Dumb: `standalone: true`, `ChangeDetectionStrategy.OnPush`, `input()`/`output()`,
  zero injecao de facade/service.
- Smart page: injeta facades/services, coordena visibilidade de create/edit,
  confirmacoes e eventos dos filhos.
- Sempre `templateUrl` + `styleUrl` com `.html` e `.scss` separados.
- Estrutura por dominio: `data-access/`, `feature/`, `ui/`.

## Padroes PrimeNG

- Tabela: `TableModule`, `p-table`, paginator, `dataKey`, `responsiveLayout="scroll"`,
  `pSortableColumn`, `p-sortIcon`.
- Tabela server-side: `[lazy]="true"`, `(onLazyLoad)="onPageChange($event)"`,
  `[totalRecords]` vindo do backend.
- Dialog: `DialogModule`, `p-dialog`, `dialogVisible = signal(true)` interno,
  `closeDialog()`, `(onHide)="onDialogHide()"`.
- Select: `SelectModule` + `p-select` com `appendTo="body"`. Nunca `DropdownModule`.
- Multiselect: `MultiSelectModule` + `p-multiselect` com `appendTo="body"`.
- Botoes: `ButtonModule` + `p-button`; icone + severity + tooltip em acoes icon-only.
- RBAC: `*hasPermission="'resource:action'"` nas acoes que modificam dados.

## Acoes por linha na tabela

Botao de acao em linha usa `size="small"` com `label` **e** `icon`. Icone sozinho depende de
tooltip para ser compreendido e reduz a area de clique.

| Acao | severity | icon |
| --- | --- | --- |
| Editar | `info` | `pi pi-pencil` |
| Detalhes / Ver | `secondary` | `pi pi-eye` |
| Ativar | `success` | `pi pi-check` |
| Desativar | `danger` | `pi pi-ban` |
| Cancelar acao | `danger` | `pi pi-ban` |
| Testar / diagnostico | `info` | icone do dominio |

Proibido em acao de tabela: `[text]="true"` (ghost button), ausencia de `size="small"`,
ausencia da diretiva de permissao quando a acao modifica dados.

**Excecao:** `[text]="true"` e permitido no botao X do header customizado de dialog e em
acoes de topbar; a proibicao vale para acoes dentro de tabela.

Botao "Novo/Adicionar" na toolbar tambem exige diretiva de permissao.

## Visibilidade de dialog: modelo de duas camadas

As duas camadas nao se contradizem, cada uma resolve um problema:

1. **Pai**: monta e desmonta o componente com `@if (dialogAberto())`. Controla existencia no DOM
   e garante formulario limpo a cada abertura.
2. **Filho**: o `p-dialog` liga `[visible]` a um signal **interno** (`dialogVisible`, inicia
   `true` ao montar). Nunca `[visible]="true"` fixo em modal interativo, nem ligado ao input do pai.

## Dialog Close Pattern

Quando o pai controla o dialog com `@if`, nunca emitir cancel/close direto do X
ou do botao Cancelar: o Angular destroi o componente antes do PrimeNG fechar o
overlay e o backdrop trava na tela.

```typescript
dialogVisible = signal<boolean>(true);

closeDialog(): void {
  this.dialogVisible.set(false);
}

onDialogHide(): void {
  this.form.reset();
  this.cancel.emit();
}
```

```html
<p-dialog [visible]="dialogVisible()" [closable]="false" (onHide)="onDialogHide()">
  <ng-template pTemplate="header">
    <p-button icon="pi pi-times" [text]="true" (onClick)="closeDialog()" />
  </ng-template>
</p-dialog>
```
