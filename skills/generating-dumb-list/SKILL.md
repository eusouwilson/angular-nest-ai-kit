---
name: generating-dumb-list
description: Gera Dumb Component de lista Angular + PrimeNG - p-table com toolbar, busca global, sorting, tag de status e acoes com RBAC. Use quando o usuario pede "crie o componente de lista", "dumb list component", "gere a tela de listagem" ou "generate-dumb-list".
---

# Gerando Dumb List Component

## Inicio rapido

Pedir o nome da entidade em PascalCase (ex: `Product`, `Team`) e gerar 3 arquivos:

```
apps/<frontend-app>/src/app/{dominio}/ui/{kebab}-list/
  {kebab}-list.component.ts
  {kebab}-list.component.html
  {kebab}-list.component.scss   <- vazio
```

## Template TypeScript

```typescript
import { ChangeDetectionStrategy, Component, input, output } from '@angular/core';
import { Table, TableModule } from 'primeng/table';
import { ButtonModule } from 'primeng/button';
import { IconFieldModule } from 'primeng/iconfield';
import { InputIconModule } from 'primeng/inputicon';
import { InputTextModule } from 'primeng/inputtext';
import { TagModule } from 'primeng/tag';
import { ToolbarModule } from 'primeng/toolbar';
import { TooltipModule } from 'primeng/tooltip';
import { HasPermissionDirective } from '@/app/shared/directives/has-permission.directive';
import { I{Entity} } from '../../data-access/{kebab}.model';

@Component({
  selector: 'app-{kebab}-list',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    TableModule, ToolbarModule, ButtonModule, InputTextModule,
    IconFieldModule, InputIconModule, TagModule, TooltipModule, HasPermissionDirective,
  ],
  templateUrl: './{kebab}-list.component.html',
  styleUrl: './{kebab}-list.component.scss',
})
export class {Entity}ListComponent {
  items = input<I{Entity}[]>([]);
  isLoading = input<boolean>(false);

  create = output<void>();
  edit = output<I{Entity}>();
  toggleStatus = output<I{Entity}>();

  onFilterGlobal(table: Table, event: Event): void {
    table.filterGlobal((event.target as HTMLInputElement).value, 'contains');
  }
}
```

## Template HTML

Ver [TEMPLATE-HTML](TEMPLATE-HTML.md).

## Regras obrigatorias

- `ChangeDetectionStrategy.OnPush` sempre.
- `templateUrl` + `styleUrl`, nunca inline.
- `input()` / `output()`, nunca `@Input()` / `@Output()`.
- Nunca injetar Facade/Service em dumb component.
- `@for (item of items(); track item.id)`, nunca `*ngFor`.
- Diretiva de permissao nos botoes de acao que modificam dados.
- Nome do output descreve a intencao (`toggleStatus`), nao a implementacao (`delete`)
  quando o comportamento real e desativar.

## Depois de gerar, adaptar

1. Ajustar colunas para os campos reais de `I{Entity}`.
2. Ajustar `[globalFilterFields]` para os campos pesquisaveis.
3. Ajustar `colspan` do `emptymessage` para o numero real de colunas.
4. Remover a coluna `status` se a entidade nao tiver esse campo.
5. Para volumes altos, migrar para paginacao server-side (`[lazy]="true"` + `(onLazyLoad)`).
