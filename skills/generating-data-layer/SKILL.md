---
name: generating-data-layer
description: Gera a camada de dados frontend (Model, Service HTTP, Facade com signals) para uma feature Angular. Use quando o usuario pede "crie a camada de dados", "gere model service e facade", "data layer", "frontend data access" ou "generate-data-layer".
---

# Gerando Frontend Data Layer (Angular)

## Inicio rapido

Pedir o nome da entidade em PascalCase. Gera 3 arquivos em `data-access/`:

```
apps/<frontend-app>/src/app/{dominio}/data-access/
  {kebab}.model.ts
  {kebab}.service.ts
  {kebab}.facade.ts
```

## Model (`{kebab}.model.ts`)

```typescript
// Re-exportar da lib compartilhada quando a interface ja existe la
export type { I{Entity} } from '@org/shared';

// Interfaces locais: payloads de UI que o backend nao conhece
export interface I{Entity}CreatePayload {
  name: string;
  description: string;
  status: boolean;
}
```

**Regra:** `export type { }` e nao `export { }` para interfaces (exigencia de `isolatedModules`).

**Guardrail:** se o payload/resposta do endpoint ainda nao existe em `@org/shared`, criar a
interface compartilhada primeiro. O `*.model.ts` nunca duplica contrato publico do backend.

## Service e Facade

- Service HTTP: [SERVICE-TEMPLATE](SERVICE-TEMPLATE.md)
- Facade com signals: [FACADE-TEMPLATE](FACADE-TEMPLATE.md)

## Divisao de responsabilidade

| Camada     | Faz                                                    | Nao faz                           |
| ---------- | ------------------------------------------------------ | --------------------------------- |
| Service    | HTTP, desembrulhar o envelope de resposta, mapear erro | guardar estado, exibir toast      |
| Facade     | estado em signals, orquestrar chamadas, notificar      | montar URL, tocar em `HttpClient` |
| Componente | renderizar, emitir eventos                             | chamar Service direto             |

## Regras obrigatorias

- `inject()`, nunca injecao por construtor.
- `@Injectable({ providedIn: 'root' })` em service e facade.
- Signals privados gravaveis: `_items`, `_isLoading`, `_selectedItem`, `_createDialogOpen`,
  `_editDialogOpen`. Publicos sempre readonly via `.asReadonly()`.
- `firstValueFrom()` para consumir Observable dentro da facade.
- `try/finally` para garantir que o loading sempre baixa.
- Nunca duplicar tipos que ja existem em `@org/shared`.
- Componente nunca chama Service direto: sempre via Facade.

## Placeholders

`<frontend-app>`, `{Entity}`, `{kebab}`, `@org/shared`, `NotificationService`
(wrapper de toast do projeto), `ErrorHandlingService` (tratamento central de erro HTTP).
