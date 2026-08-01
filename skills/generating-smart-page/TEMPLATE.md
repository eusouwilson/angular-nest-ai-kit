# Template completo da Smart Page

Parte da skill [generating-smart-page](SKILL.md).

## TypeScript

```typescript
import { Component, effect, inject } from '@angular/core';
import { ConfirmationService, MessageService } from 'primeng/api';
import { ConfirmDialogModule } from 'primeng/confirmdialog';
import { ToastModule } from 'primeng/toast';
import { {Entity}ListComponent } from '../../ui/{kebab}-list/{kebab}-list.component';
import { {Entity}CreateComponent } from '../../ui/{kebab}-create/{kebab}-create.component';
import { {Entity}EditComponent } from '../../ui/{kebab}-edit/{kebab}-edit.component';
import { {Entity}Facade } from '../../data-access/{kebab}.facade';
import { TenantFacade } from '@/app/shared/data-access/tenant.facade';
import { I{Entity}, I{Entity}CreatePayload } from '../../data-access/{kebab}.model';

@Component({
  selector: 'app-{kebab}-page',
  standalone: true,
  imports: [
    {Entity}ListComponent,
    {Entity}CreateComponent,
    {Entity}EditComponent,
    ToastModule,
    ConfirmDialogModule,
  ],
  providers: [ConfirmationService, MessageService],
  templateUrl: './{kebab}-page.component.html',
  styleUrl: './{kebab}-page.component.scss',
})
export class {Entity}PageComponent {
  protected facade = inject({Entity}Facade);
  private tenantFacade = inject(TenantFacade);
  private confirmationService = inject(ConfirmationService);

  constructor() {
    // Recarrega sempre que o escopo ativo muda.
    // Remover se a entidade nao depende de tenant: usar ngOnInit com loadAll().
    effect(() => {
      const tenantId = this.tenantFacade.activeTenantId();
      if (tenantId) void this.facade.loadAll(tenantId);
    });
  }

  onCreate(): void {
    this.facade.toggleCreateDialog();
  }

  onEdit(item: I{Entity}): void {
    this.facade.selectItem(item);
    this.facade.toggleEditDialog();
  }

  onToggleStatus(item: I{Entity}): void {
    this.confirmationService.confirm({
      message: item.status
        ? `"${item.name}" esta ativo. Deseja desativar?`
        : `"${item.name}" esta inativo. Deseja ativar?`,
      header: 'Confirmacao',
      icon: 'pi pi-exclamation-triangle',
      acceptLabel: 'Sim',
      rejectLabel: 'Nao',
      accept: () => void this.facade.toggleStatus(item.id, item.status),
    });
  }

  onSaveCreate(payload: I{Entity}CreatePayload): void {
    const tenantId = this.tenantFacade.activeTenantId();
    void this.facade.create({ ...payload, tenantId });
    this.facade.toggleCreateDialog();
  }

  onSaveEdit(payload: Partial<I{Entity}>): void {
    if (!payload.id) return;
    void this.facade.update(payload.id, payload);
    this.facade.toggleEditDialog();
  }
}
```

## HTML

```html
<p-toast />

<app-{kebab}-list
  [items]="facade.items()"
  [isLoading]="facade.isLoading()"
  (create)="onCreate()"
  (edit)="onEdit($event)"
  (toggleStatus)="onToggleStatus($event)" />

@if (facade.createDialogOpen()) {
  <app-{kebab}-create
    [isLoading]="facade.isLoading()"
    (save)="onSaveCreate($event)"
    (cancel)="facade.toggleCreateDialog()" />
}

@if (facade.editDialogOpen()) {
  <app-{kebab}-edit
    [isLoading]="facade.isLoading()"
    [item]="facade.selectedItem()"
    (save)="onSaveEdit($event)"
    (cancel)="facade.toggleEditDialog()" />
}

<p-confirmDialog />
```

## Rota

```typescript
{
  path: '{kebab}',
  loadComponent: () =>
    import('./feature/{kebab}-page/{kebab}-page.component').then((m) => m.{Entity}PageComponent),
  canActivate: [authGuard],
  data: { breadcrumb: '{EntityLabel}' },
}
```

Manter o metadado de breadcrumb em toda rota com `loadComponent`: e o que evita ter titulo
duplicado dentro da pagina.
