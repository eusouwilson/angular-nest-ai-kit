---
name: generating-backend-module
description: Gera modulo NestJS completo - DTOs com class-validator, Service estendendo BaseService, Controller com Swagger completo, Module e testes unitarios com TDD. Use quando o usuario pede "crie um modulo backend", "novo endpoint", "novo recurso no backend" ou "generate-module".
---

# Gerando Backend Module (NestJS)

## Inicio rapido

Pedir o nome do modulo em kebab-case (ex: `products`, `teams`, `reports`). Gera:

```
apps/<backend-app>/src/<modulo>/
  dto/
    create-{module}.dto.ts
    update-{module}.dto.ts
    {module}-response.dto.ts
  {module}.service.ts
  {module}.controller.ts
  {module}.module.ts
  {module}.service.spec.ts   <- TDD: escrever ANTES do service
```

## Ordem de geracao (obrigatoria)

1. **Interface compartilhada**: verificar/criar em `@org/shared` se ainda nao existe.
2. **DTOs**: create, update, response.
3. **Testes** (`spec.ts`): escrever ANTES do service (TDD Red).
4. **Service**: implementar ate os testes passarem (TDD Green).
5. **Controller**: conectar DTOs + Swagger.
6. **Module**: registrar controller + service e importar no modulo raiz.

## Templates

- DTOs: [DTOS](DTOS.md)
- Service: [SERVICE](SERVICE.md)
- Controller e Module: [CONTROLLER](CONTROLLER.md)
- Testes: [TESTS](TESTS.md)

## Checklist Swagger (por endpoint)

- [ ] `@ApiOperation({ summary, description })`
- [ ] `@ApiResponse({ status: 200, type: ResponseDto })`
- [ ] `@ApiParam` se a rota tem `:id`
- [ ] `@ApiBody({ type: CreateDto })` em POST/PUT/PATCH
- [ ] Controller com `@ApiTags`, `@ApiBearerAuth`, `@UseGuards(JwtAuthGuard)`

## Regras criticas

- Nunca `throw new Error()`: usar `HttpException` ou o helper de erro do `BaseService`.
- Nunca `export default`.
- Sempre `excludeExtraneousValues: true` na serializacao dos response DTOs.
- Importar tipos de `@org/shared`; nunca duplicar contrato publico.
- TDD: spec antes da implementacao.
- Banco em `snake_case`, JSON em `camelCase` (mapear com `@Expose` / `@Transform` ou no service).
- `select` explicito no Prisma: nunca retornar a linha inteira por padrao.

## Placeholders

| Placeholder | Troca por |
| --- | --- |
| `<backend-app>` | nome do app NestJS |
| `{Entity}` / `{module}` / `{model}` | PascalCase / kebab-case / nome no Prisma |
| `@org/shared` | alias da lib de contratos |
| `JwtAuthGuard` | guard de auth do projeto |
| `tenantId` / `tenant_id` | campo de escopo multi-tenant, ou remover se nao aplicavel |

## Pre-requisito: BaseService

Os templates assumem um `BaseService` com:

```typescript
protected validateId(id: number): void;
protected validateExists<T>(entity: T | null, label: string, id: number): asserts entity is T;
protected handleServiceError(error: unknown, context: string): never;
```

Se o projeto destino nao tiver, criar antes ou trocar as chamadas por
`throw new HttpException(...)` direto no service.

## Alinhamento de arquitetura

Contratos HTTP publicos nascem na lib compartilhada, nunca duplicados entre front e back.
Nao introduzir novos limites de rede (microservicos, BFF extra) sem aprovacao explicita.
