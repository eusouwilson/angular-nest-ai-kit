---
name: nestjs-api-standards
description: Padroes de API NestJS - envelope de resposta, checklist de controller e endpoint, tratamento de erro, conversao snake_case para camelCase e RBAC por endpoint. Use ao criar/revisar controller ou service, ou ao definir contrato de endpoint novo.
---

# Padroes de API NestJS

Regras transversais do backend. Complementa `generating-backend-module`, que traz os templates.

## Envelope de resposta

Todo endpoint retorna `ApiResponse<T>`:

```typescript
interface ApiResponse<T> {
  message: string;      // mensagem legivel, pronta para exibir
  statusCode: number;
  type: 'success' | 'error';
  data: T;
}
```

Motivo: o frontend tem um unico ponto de desembrulho e uma unica forma de exibir feedback.
Endpoint que devolve o payload cru quebra esse contrato e obriga tratamento especial no cliente.

O tipo do envelope vive na lib compartilhada, nao em cada app.

## Checklist do controller

1. `@ApiTags('nome-do-modulo')`
2. `@ApiBearerAuth('JWT-auth')` (mais `@ApiSecurity` se houver outro esquema)
3. `@UseGuards(JwtAuthGuard)` (e o guard de permissao quando houver RBAC)
4. `@Controller('rota')` com nome no plural, kebab-case

## Checklist do endpoint

1. `@ApiOperation({ summary, description })`
2. `@ApiResponse({ status, type })` para cada status possivel
3. `@ApiParam(...)` quando a rota tem `:id`
4. `@ApiBody({ type })` em POST/PUT/PATCH
5. Decorator de permissao quando aplicavel
6. `ParseIntPipe` (ou equivalente) em parametro numerico

## Tratamento de erro

- Proibido `throw new Error(...)`: vira 500 generico e some com a causa.
- Usar `HttpException` com status correto, ou o helper central do `BaseService`.
- Mapear erro do Prisma para status HTTP:

| Codigo | Significado | Status |
| --- | --- | --- |
| `P2002` | violacao de unique | 409 |
| `P2025` | registro nao encontrado | 404 |
| `P2003` | violacao de FK | 400 ou 409 |

- Mensagem de erro nunca expoe stack, SQL ou caminho de arquivo.

## snake_case no banco, camelCase na API

A conversao acontece no service (mapeamento explicito) ou no response DTO (`@Expose` +
`@Transform`). Nunca vazar `union_code` ou `created_at` para o consumidor da API.

Serializacao com `excludeExtraneousValues: true`: sem isso, campo novo do banco vaza para
a resposta sem ninguem decidir.

## Paginacao

- Sempre no banco (`skip`/`take`), nunca em memoria.
- Resposta paginada carrega metadados: total, pagina atual, tamanho da pagina.
- Definir tamanho maximo de pagina e recusar valores acima.

## RBAC

Formato `recurso:acao`. Mapeamento: `POST` -> `create`, `GET` -> `read`,
`PUT`/`PATCH` -> `update`, `DELETE` -> `delete`. Sem acao guarda-chuva.
Ver [rbac-security-auditor](../rbac-security-auditor/SKILL.md) para o protocolo completo.

## Anti-patterns

- Regra de autorizacao hardcoded dentro do service (`if (!isAdmin) throw`): autorizacao
  pertence ao guard. Service filtra visibilidade de dados, nao nega acesso por papel.
- `select` implicito no Prisma (retornar a linha inteira).
- DTO de resposta declarado no app quando o contrato e publico: pertence a lib compartilhada.
- Endpoint sem `@ApiOperation` util ("cria" nao explica nada).
- Logar payload inteiro em endpoint que recebe dado pessoal.
