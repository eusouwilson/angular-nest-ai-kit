# Template de Service HTTP (Angular)

Parte da skill [generating-data-layer](SKILL.md).

```typescript
import { inject, Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { catchError, map } from 'rxjs/operators';
import { environment } from '@/environments/environment';
import { ErrorHandlingService } from '@/app/shared/services/error-handling.service';
import { I{Entity}, I{Entity}CreatePayload } from './{kebab}.model';

@Injectable({ providedIn: 'root' })
export class {Entity}Service {
  private http = inject(HttpClient);
  private errorHandling = inject(ErrorHandlingService);

  private readonly baseUrl = `${environment.apiUrl}/{kebab-plural}`;

  getAll(tenantId?: number): Observable<I{Entity}[]> {
    const url = tenantId ? `${this.baseUrl}?tenantId=${tenantId}` : this.baseUrl;
    return this.http.get<ApiEnvelope<I{Entity}[]>>(url).pipe(
      map((res) => res.data),
      catchError((err) => this.errorHandling.handleError(err, 'buscar {entityLabel}')),
    );
  }

  getById(id: number): Observable<I{Entity}> {
    return this.http.get<ApiEnvelope<I{Entity}>>(`${this.baseUrl}/${id}`).pipe(
      map((res) => res.data),
      catchError((err) => this.errorHandling.handleError(err, 'buscar {entityLabel}')),
    );
  }

  create(payload: I{Entity}CreatePayload): Observable<I{Entity}> {
    return this.http.post<ApiEnvelope<I{Entity}>>(this.baseUrl, payload).pipe(
      map((res) => res.data),
      catchError((err) => this.errorHandling.handleError(err, 'criar {entityLabel}')),
    );
  }

  update(id: number, payload: Partial<I{Entity}>): Observable<I{Entity}> {
    return this.http.put<ApiEnvelope<I{Entity}>>(`${this.baseUrl}/${id}`, payload).pipe(
      map((res) => res.data),
      catchError((err) => this.errorHandling.handleError(err, 'atualizar {entityLabel}')),
    );
  }

  toggleStatus(id: number, currentStatus: boolean): Observable<I{Entity}> {
    return this.http
      .patch<ApiEnvelope<I{Entity}>>(`${this.baseUrl}/${id}/status`, { status: !currentStatus })
      .pipe(
        map((res) => res.data),
        catchError((err) => this.errorHandling.handleError(err, 'alterar status')),
      );
  }
}
```

## Envelope de resposta

O template assume que o backend responde com um envelope:

```typescript
interface ApiEnvelope<T> {
  message: string;
  statusCode: number;
  type: 'success' | 'error';
  data: T;
}
```

Importar esse tipo de `@org/shared` em vez de redeclarar. Se o backend devolve o payload
cru, remover o `map((res) => res.data)` e tipar direto: `this.http.get<I{Entity}[]>(url)`.

Nunca usar `any` no tipo do `http.get`: e o ponto onde o contrato deixa de ser verificado.
