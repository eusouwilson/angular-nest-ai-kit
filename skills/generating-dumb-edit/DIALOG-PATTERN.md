# Template HTML do Edit Component

Parte da skill [generating-dumb-edit](SKILL.md). Mesmo padrao do create; diferencas marcadas
com comentario.

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
        <!-- diferenca: titulo "Editar" -->
        <p class="text-2xl w-full m-0">Editar {EntityLabel}</p>
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
      <!-- diferenca: permissao :update -->
      <p-button
        *hasPermission="'{kebab}:update'"
        label="Salvar"
        icon="pi pi-check"
        severity="success"
        [disabled]="form.invalid || isLoading()"
        type="submit" />
    </ng-template>

  </p-dialog>
</form>
```

Os snippets de campos adicionais (select, multiselect, checkbox) valem igual:
ver [create/DIALOG-PATTERN](../generating-dumb-create/DIALOG-PATTERN.md).
