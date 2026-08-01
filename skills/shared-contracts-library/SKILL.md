---
name: shared-contracts-library
description: Regras da lib de contratos compartilhados entre frontend e backend - o que entra, o que nunca entra, barrel, export type e nomenclatura. Use ao criar interface publica, decidir onde um tipo mora ou resolver duplicacao de contrato entre app e API.
---

# Lib de Contratos Compartilhados

Uma lib de TypeScript puro onde vivem os contratos HTTP publicos, consumida pelos dois lados.

## Regra central

**Contrato HTTP publico nasce na lib, antes de qualquer model ou DTO local.**

Duplicar o shape nos dois lados parece inofensivo no dia 1 e vira divergencia silenciosa no
dia 30: o backend adiciona um campo, o frontend nao sabe, e o bug aparece em producao como
`undefined`.

## O que entra

- Interfaces de request e response de endpoints publicos.
- Enums usados pelos dois lados.
- Tipos utilitarios de resposta: envelope, paginacao, metadados.
- Constantes de contrato: tamanho de pagina default e maximo.
- Tipos de autenticacao e permissao (podem viver numa segunda lib dedicada).

## O que nunca entra

- Qualquer import de framework: `@angular/*`, `@nestjs/*`, decorators de framework.
  A lib precisa compilar e rodar nos dois contextos.
- Import de `apps/`: a dependencia e unidirecional; o inverso cria ciclo.
- Tipo usado por um lado so. Se so o backend usa, e do backend.
- Logica de negocio. E lib de contrato, nao de dominio.

A excecao tolerada e `class-validator` / `class-transformer` quando o DTO validado e mesmo
compartilhado. Fora isso, TypeScript puro.

## Convencoes

- Interface com prefixo `I`: `IProduct`, `IPaginatedResponse`.
- Toda export publica passa pelo barrel (`src/index.ts`). Import por caminho profundo
  amarra o consumidor a estrutura interna da lib.
- Re-export de tipo usa `export type { }` (exigencia de `isolatedModules`).
  Enum e valor em runtime: re-exportar com `export { }` normal.

```typescript
// tipos
export type { IProduct } from './interfaces/product.interface';

// enum (valor, nao so tipo)
export { OrderStatus } from './enums/order-status.enum';
```

## Envelope e paginacao

Manter os wrappers na lib para que os dois lados falem a mesma lingua:

```typescript
export interface ApiResponse<T> {
  message: string;
  statusCode: number;
  type: 'success' | 'error';
  data: T;
}

export interface PaginatedResponse<T> {
  items: T[];
  meta: IPaginationMeta;
}
```

## Checklist ao mexer na lib

- [ ] Export nova adicionada ao barrel
- [ ] `export type` usado para tipo, `export` normal para enum
- [ ] Nenhum import de framework introduzido
- [ ] Os dois apps ainda compilam
- [ ] O DTO do backend implementa a interface publica em vez de redeclarar campos
- [ ] O model do frontend re-exporta da lib em vez de copiar
