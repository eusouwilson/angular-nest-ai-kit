# Template de Facade com Signals (Angular)

Parte da skill [generating-data-layer](SKILL.md).

```typescript
import { computed, inject, Injectable, signal } from '@angular/core';
import { firstValueFrom } from 'rxjs';
import { NotificationService } from '@/app/shared/services/notification.service';
import { I{Entity}, I{Entity}CreatePayload } from './{kebab}.model';
import { {Entity}Service } from './{kebab}.service';

@Injectable({ providedIn: 'root' })
export class {Entity}Facade {
  private service = inject({Entity}Service);
  private notification = inject(NotificationService);

  // Signals privados gravaveis
  private readonly _items = signal<I{Entity}[]>([]);
  private readonly _selectedItem = signal<I{Entity} | null>(null);
  private readonly _isLoading = signal<boolean>(false);
  private readonly _createDialogOpen = signal<boolean>(false);
  private readonly _editDialogOpen = signal<boolean>(false);

  // Signals publicos readonly
  readonly items = this._items.asReadonly();
  readonly selectedItem = this._selectedItem.asReadonly();
  readonly isLoading = this._isLoading.asReadonly();
  readonly createDialogOpen = this._createDialogOpen.asReadonly();
  readonly editDialogOpen = this._editDialogOpen.asReadonly();

  // Computed
  readonly count = computed(() => this._items().length);

  toggleCreateDialog(): void { this._createDialogOpen.update((v) => !v); }
  toggleEditDialog(): void { this._editDialogOpen.update((v) => !v); }

  selectItem(item: I{Entity}): void { this._selectedItem.set(item); }

  async loadAll(tenantId?: number): Promise<void> {
    try {
      this._isLoading.set(true);
      const data = await firstValueFrom(this.service.getAll(tenantId));
      this._items.set(data);
    } catch (error) {
      this.notification.error('Nao foi possivel carregar os registros.');
      console.error('Erro ao carregar {entityLabel}:', error);
    } finally {
      this._isLoading.set(false);
    }
  }

  async create(payload: I{Entity}CreatePayload): Promise<void> {
    try {
      this._isLoading.set(true);
      const item = await firstValueFrom(this.service.create(payload));
      this._items.update((items) => [...items, item]);
      this.notification.success('{EntityLabel} criado com sucesso.');
    } catch (error) {
      this.notification.error('Nao foi possivel criar o registro.');
      console.error('Erro ao criar {entityLabel}:', error);
    } finally {
      this._isLoading.set(false);
    }
  }

  async update(id: number, payload: Partial<I{Entity}>): Promise<void> {
    try {
      this._isLoading.set(true);
      const updated = await firstValueFrom(this.service.update(id, payload));
      this._items.update((items) => items.map((i) => (i.id === id ? updated : i)));
      this.notification.success('{EntityLabel} atualizado com sucesso.');
    } catch (error) {
      this.notification.error('Nao foi possivel atualizar o registro.');
      console.error('Erro ao atualizar {entityLabel}:', error);
    } finally {
      this._isLoading.set(false);
    }
  }

  async toggleStatus(id: number, currentStatus: boolean): Promise<void> {
    try {
      this._isLoading.set(true);
      const updated = await firstValueFrom(this.service.toggleStatus(id, currentStatus));
      this._items.update((items) => items.map((i) => (i.id === id ? updated : i)));
      this.notification.success(
        currentStatus ? '{EntityLabel} desativado.' : '{EntityLabel} ativado.',
      );
    } catch (error) {
      this.notification.error('Nao foi possivel alterar o status.');
      console.error('Erro ao alterar status:', error);
    } finally {
      this._isLoading.set(false);
    }
  }
}
```

## Regras do estado

- Atualizacao otimista da lista local apos create/update: evita refetch completo.
  Se o backend calcula campos derivados, preferir recarregar o item alterado.
- Erro nunca fica silencioso: notificacao para o usuario + log para o dev.
- `_isLoading` sobe em toda operacao e desce no `finally`, inclusive no caminho de erro.
- Nada de `Subject`/`BehaviorSubject` para estado: signals cobrem o caso.
- Facade nao conhece PrimeNG nem componente: so estado e orquestracao.
