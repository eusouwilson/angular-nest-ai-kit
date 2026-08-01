# Template HTML do Dumb List Component

Parte da skill [generating-dumb-list](SKILL.md).

```html
<div class="col-12">
  <div class="card px-6 py-6">
    <p-table
      #dt
      [value]="items()"
      [paginator]="true"
      [rows]="10"
      [rowsPerPageOptions]="[10, 25, 50]"
      [loading]="isLoading()"
      [showCurrentPageReport]="true"
      currentPageReportTemplate="{first} - {last} de {totalRecords} registros"
      [globalFilterFields]="['name', 'description']"
      [rowHover]="true"
      responsiveLayout="scroll"
      dataKey="id">

      <ng-template pTemplate="caption">
        <p-toolbar>
          <ng-template pTemplate="start">
            <p-iconfield iconPosition="left">
              <p-inputicon styleClass="pi pi-search"></p-inputicon>
              <input
                type="text"
                pInputText
                (input)="onFilterGlobal(dt, $event)"
                placeholder="Pesquisar..." />
            </p-iconfield>
          </ng-template>
          <ng-template pTemplate="end">
            <p-button
              *hasPermission="'{kebab}:create'"
              icon="pi pi-plus"
              label="Novo {EntityLabel}"
              severity="primary"
              (onClick)="create.emit()" />
          </ng-template>
        </p-toolbar>
      </ng-template>

      <ng-template pTemplate="header">
        <tr>
          <th pSortableColumn="name">Nome <p-sortIcon field="name" /></th>
          <th pSortableColumn="description">Descricao <p-sortIcon field="description" /></th>
          <th pSortableColumn="status">Status <p-sortIcon field="status" /></th>
          <th style="width: 10rem">Acoes</th>
        </tr>
      </ng-template>

      <ng-template pTemplate="body" let-item>
        <tr>
          <td>{{ item.name }}</td>
          <td>{{ item.description }}</td>
          <td>
            <p-tag
              [value]="item.status ? 'Ativo' : 'Inativo'"
              [severity]="item.status ? 'success' : 'danger'" />
          </td>
          <td>
            <div class="flex gap-2">
              <p-button
                *hasPermission="'{kebab}:update'"
                icon="pi pi-pencil"
                label="Editar"
                severity="info"
                size="small"
                (onClick)="edit.emit(item)" />
              <p-button
                *hasPermission="'{kebab}:update'"
                [icon]="item.status ? 'pi pi-ban' : 'pi pi-check'"
                [label]="item.status ? 'Desativar' : 'Ativar'"
                [severity]="item.status ? 'danger' : 'success'"
                size="small"
                (onClick)="toggleStatus.emit(item)" />
            </div>
          </td>
        </tr>
      </ng-template>

      <ng-template pTemplate="emptymessage">
        <tr>
          <td colspan="4" class="text-center py-8 text-color-secondary">
            <i class="pi pi-inbox text-4xl mb-3 block"></i>
            Nenhum registro encontrado.
          </td>
        </tr>
      </ng-template>

    </p-table>
  </div>
</div>
```

## Variante server-side

Para listas grandes, trocar a paginacao client-side por lazy:

```html
<p-table
  [value]="items()"
  [lazy]="true"
  (onLazyLoad)="pageChange.emit($event)"
  [totalRecords]="totalRecords()"
  [paginator]="true"
  [rows]="10"
  [loading]="isLoading()">
```

Nesse caso o filtro global tambem vai para o servidor: o `filterGlobal` local deixa de valer
e o componente passa a emitir o termo de busca (com debounce) para a smart page.
