---
name: generating-dumb-create
description: Gera Dumb Component de criacao Angular + PrimeNG - p-dialog modal com Reactive Forms e padrao dialogVisible signal para evitar bug de overlay travado. Use quando o usuario pede "crie o componente de criacao", "dialog de criar", "dumb create component" ou "generate-dumb-create".
---

# Gerando Dumb Create Component

## Inicio rapido

Pedir o nome da entidade em PascalCase e gerar 3 arquivos:

```
apps/<frontend-app>/src/app/{dominio}/ui/{kebab}-create/
  {kebab}-create.component.ts
  {kebab}-create.component.html
  {kebab}-create.component.scss   <- vazio
```

## Template TypeScript

```typescript
import { ChangeDetectionStrategy, Component, inject, input, output, signal } from '@angular/core';
import { FormBuilder, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import { ButtonModule } from 'primeng/button';
import { DialogModule } from 'primeng/dialog';
import { InputTextModule } from 'primeng/inputtext';
import { TextareaModule } from 'primeng/textarea';
import { ProgressBarModule } from 'primeng/progressbar';
import { HasPermissionDirective } from '@/app/shared/directives/has-permission.directive';
import { I{Entity}CreatePayload } from '../../data-access/{kebab}.model';

@Component({
  selector: 'app-{kebab}-create',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    DialogModule, ReactiveFormsModule, InputTextModule, TextareaModule,
    ButtonModule, ProgressBarModule, HasPermissionDirective,
  ],
  templateUrl: './{kebab}-create.component.html',
  styleUrl: './{kebab}-create.component.scss',
})
export class {Entity}CreateComponent {
  private fb = inject(FormBuilder);

  isLoading = input<boolean>(false);

  save = output<I{Entity}CreatePayload>();
  cancel = output<void>();

  // Signal interno: evita o bug de overlay travado quando o pai usa @if
  dialogVisible = signal<boolean>(true);

  form: FormGroup = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(3)]],
    description: ['', [Validators.required, Validators.minLength(3)]],
  });

  closeDialog(): void {
    this.dialogVisible.set(false);
  }

  onDialogHide(): void {
    this.form.reset();
    this.cancel.emit();
  }

  submit(): void {
    if (this.form.valid) {
      this.save.emit(this.form.value as I{Entity}CreatePayload);
    }
  }
}
```

## Template HTML e padrao de fechamento

Ver [DIALOG-PATTERN](DIALOG-PATTERN.md).

## Regras criticas

- `dialogVisible = signal(true)` interno; nao ligar `[visible]` a um input do pai.
- Nunca chamar `cancel.emit()` direto do botao X ou Cancelar.
- Fluxo unico: X > `closeDialog()` > `dialogVisible.set(false)` > `(onHide)` >
  `onDialogHide()` > `cancel.emit()`.
- `ChangeDetectionStrategy.OnPush` sempre.
- `templateUrl` + `styleUrl`, nunca inline.
- Botao de salvar desabilitado (nao escondido) em form invalido ou carregando.

## Depois de gerar, adaptar

1. Campos do form conforme `I{Entity}CreatePayload`.
2. Importar `SelectModule`, `CheckboxModule`, `MultiSelectModule` conforme os campos.
3. Criar `I{Entity}CreatePayload` no model se ainda nao existir.
4. Ajustar largura do dialog (padrao `600px`; formularios maiores `700px` a `900px`).
