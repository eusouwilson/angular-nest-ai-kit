# Dialog Close Pattern + HTML do Create Component

Parte da skill [generating-dumb-create](SKILL.md).

## Por que o signal interno

Quando o pai monta o dialog com `@if (facade.createDialogOpen())`, emitir `cancel`
diretamente destroi o componente ANTES do PrimeNG fechar o overlay. Resultado: o backdrop
fica preso na tela e a pagina trava.

```
Clique no X
  └─ closeDialog()            -> dialogVisible.set(false)
       └─ PrimeNG fecha o overlay (com animacao)
            └─ (onHide) dispara
                 └─ onDialogHide()
                      ├─ form.reset()
                      └─ cancel.emit()   -> agora sim o pai remove via @if
```

## Template HTML completo

```html
<form [formGroup]="form" (ngSubmit)="submit()">
  <p-dialog
    [visible]="dialogVisible()"
    [style]="{ width: '600px' }"
    [modal]="true"
    [closable]="false"
    [closeOnEscape]="true"
    (onHide)="onDialogHide()"
    class="p-fluid">

    @if (isLoading()) {
      <p-progressBar mode="indeterminate" [style]="{ height: '6px' }" />
    }

    <ng-template pTemplate="header">
      <div class="flex items-center w-full">
        <p class="text-2xl w-full m-0">Adicionar {EntityLabel}</p>
        <p-button
          icon="pi pi-times"
          severity="secondary"
          [text]="true"
          [rounded]="true"
          (onClick)="closeDialog()" />
      </div>
    </ng-template>

    <ng-template pTemplate="content">

      <div class="field flex flex-col gap-2 mb-4">
        <label for="name">Nome *</label>
        <input id="name" pInputText placeholder="Nome" formControlName="name" />
        @if (form.get('name')?.invalid && form.get('name')?.touched) {
          <small class="p-error">Nome e obrigatorio e precisa de no minimo 3 caracteres.</small>
        }
      </div>

      <div class="field flex flex-col gap-2 mb-4">
        <label for="description">Descricao *</label>
        <textarea
          id="description"
          pTextarea
          rows="3"
          placeholder="Descricao"
          formControlName="description"></textarea>
        @if (form.get('description')?.invalid && form.get('description')?.touched) {
          <small class="p-error">Descricao e obrigatoria e precisa de no minimo 3 caracteres.</small>
        }
      </div>

    </ng-template>

    <ng-template pTemplate="footer">
      <p-button
        label="Cancelar"
        icon="pi pi-times"
        severity="danger"
        [text]="true"
        type="button"
        (onClick)="closeDialog()" />
      <p-button
        *hasPermission="'{kebab}:create'"
        label="Salvar"
        icon="pi pi-check"
        severity="success"
        [disabled]="form.invalid || isLoading()"
        type="submit" />
    </ng-template>

  </p-dialog>
</form>
```

O `<form>` envolve o `p-dialog` (e nao o contrario) porque o PrimeNG move o conteudo do
dialog para fora do fluxo do DOM; sem isso o `type="submit"` do footer perde o formulario.

## Snippets de campos adicionais

### p-select

```html
<div class="field flex flex-col gap-2 mb-4">
  <label for="typeId">Tipo *</label>
  <p-select
    inputId="typeId"
    formControlName="typeId"
    [options]="types()"
    optionLabel="description"
    optionValue="id"
    placeholder="Selecione um tipo"
    appendTo="body"
    [showClear]="true" />
</div>
```

### p-multiselect

```html
<div class="field flex flex-col gap-2 mb-4">
  <label for="teamIds">Equipes</label>
  <p-multiselect
    inputId="teamIds"
    formControlName="teamIds"
    [options]="teams()"
    optionLabel="description"
    optionValue="id"
    placeholder="Selecione equipes"
    display="chip"
    appendTo="body"
    [filter]="true" />
</div>
```

### Checkbox binario

```html
<div class="field-checkbox flex items-center gap-2 mb-4">
  <p-checkbox inputId="status" formControlName="status" [binary]="true" />
  <label for="status">Ativo</label>
</div>
```

### Grupo de checkboxes

```html
<div class="field mb-4">
  <p-fieldset legend="Areas">
    <div class="flex flex-row flex-wrap gap-3">
      @for (segment of segments(); track segment.id) {
        <div class="flex items-center gap-1">
          <p-checkbox
            name="segment"
            [value]="segment.id"
            [inputId]="segment.id.toString()"
            (onChange)="onSegmentChange($event, segment.id)" />
          <label [for]="segment.id.toString()">{{ segment.description }}</label>
        </div>
      }
    </div>
  </p-fieldset>
</div>
```

`appendTo="body"` em todo overlay dentro do dialog: sem isso o painel do select e
cortado pelo `overflow` do dialog.
