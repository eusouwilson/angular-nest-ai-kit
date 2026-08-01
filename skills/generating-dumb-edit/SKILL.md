---
name: generating-dumb-edit
description: Gera Dumb Component de edicao Angular + PrimeNG - p-dialog com Reactive Form pre-preenchido via effect() e padrao dialogVisible signal. Use quando o usuario pede "crie o componente de edicao", "dialog de editar", "dumb edit component" ou "generate-dumb-edit".
---

# Gerando Dumb Edit Component

## Inicio rapido

Pedir o nome da entidade em PascalCase e gerar 3 arquivos:

```
apps/<frontend-app>/src/app/{dominio}/ui/{kebab}-edit/
  {kebab}-edit.component.ts
  {kebab}-edit.component.html
  {kebab}-edit.component.scss   <- vazio
```

## Template TypeScript

```typescript
import { ChangeDetectionStrategy, Component, effect, inject, input, output, signal } from '@angular/core';
import { FormBuilder, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import { ButtonModule } from 'primeng/button';
import { DialogModule } from 'primeng/dialog';
import { InputTextModule } from 'primeng/inputtext';
import { TextareaModule } from 'primeng/textarea';
import { ProgressBarModule } from 'primeng/progressbar';
import { HasPermissionDirective } from '@/app/shared/directives/has-permission.directive';
import { I{Entity} } from '../../data-access/{kebab}.model';

@Component({
  selector: 'app-{kebab}-edit',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    DialogModule, ReactiveFormsModule, InputTextModule, TextareaModule,
    ButtonModule, ProgressBarModule, HasPermissionDirective,
  ],
  templateUrl: './{kebab}-edit.component.html',
  styleUrl: './{kebab}-edit.component.scss',
})
export class {Entity}EditComponent {
  private fb = inject(FormBuilder);

  isLoading = input<boolean>(false);
  item = input<I{Entity} | null>(null);

  save = output<Partial<I{Entity}>>();
  cancel = output<void>();

  dialogVisible = signal<boolean>(true);

  form: FormGroup = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(3)]],
    description: ['', [Validators.required, Validators.minLength(3)]],
  });

  constructor() {
    effect(() => {
      const current = this.item();
      if (current) {
        this.form.patchValue({
          name: current.name,
          description: current.description,
        });
      }
    });
  }

  closeDialog(): void {
    this.dialogVisible.set(false);
  }

  onDialogHide(): void {
    this.form.reset();
    this.cancel.emit();
  }

  submit(): void {
    if (this.form.valid) {
      this.save.emit({ id: this.item()?.id, ...this.form.value });
    }
  }
}
```

## Diferencas em relacao ao create

1. Input `item` com a entidade selecionada, vinda da smart page.
2. `effect()` no construtor faz `patchValue` sempre que a entidade muda.
3. Titulo "Editar" e permissao `'{kebab}:update'` no botao Salvar.
4. O payload de `save` inclui o `id`.

## Template HTML

Mesmo padrao do create: ver [DIALOG-PATTERN](DIALOG-PATTERN.md).

## Regras

- `effect()` no `constructor()` para o `patchValue`: garante reatividade quando a entidade muda.
- No `patchValue`, mapear apenas campos editaveis (fora `id`, `createdAt`, chaves de escopo).
- Mesmo padrao `dialogVisible` do create: nunca emitir `cancel` direto do X.
- Se o form tiver estado sujo, avaliar confirmacao antes de fechar (nao incluido no template).
