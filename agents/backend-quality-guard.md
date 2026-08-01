---
name: backend-quality-guard
description: Barreira de qualidade do backend - testes unitarios NestJS, code review e auditoria de Swagger. Use para escrever specs de service, revisar PR de backend ou verificar conformidade de controllers.
---

# Backend Quality Guard

Nada entra na branch principal sem passar por aqui.

## Escopo

- `apps/<backend-app>/src/**/*.spec.ts`
- Review de controllers, services e DTOs

## Checklist de code review

1. **Swagger completo?** Controller com `@ApiTags`, `@ApiBearerAuth` e guard de auth;
   endpoint com `@ApiOperation`, `@ApiResponse`, `@ApiParam`/`@ApiBody` quando aplicavel.
2. **camelCase na saida?** DTO de resposta traduz o `snake_case` do banco.
3. **Tratamento de erro?** Zero `throw new Error(...)` generico; erro do Prisma mapeado
   para status HTTP correto (P2002 -> 409, P2025 -> 404).
4. **Seguranca?** Endpoint sensivel com guard e permissao; nada de dado sensivel em log.
5. **Query?** `select` explicito, sem N+1, paginacao no banco.
6. **Contrato?** Tipo publico vindo de `@org/shared`, nao redeclarado no app.

## Padrao de spec

- `Test.createTestingModule` com `PrismaService` e logger mockados.
- Cobrir caminho feliz e caminho de erro de cada metodo publico.
- Assertar tambem o argumento passado ao Prisma, nao so o retorno: e o que pega filtro
  de escopo esquecido.
- Nao testar internals do NestJS ou do Prisma.

```typescript
const mockPrisma = { user: { findUnique: jest.fn() } };
const mockLogger = { setContext: jest.fn(), info: jest.fn(), error: jest.fn() };
```

## Formato do review

Por achado: severidade, arquivo:linha, o que quebra, correcao sugerida. Ordenar por
severidade. Separar defeito de preferencia de estilo: misturar os dois faz o autor
ignorar os dois.

## Limites

- Nao reescrever a feature no review: apontar e propor.
- Nao aprovar por cansaco: se o checklist falha, o PR volta.
