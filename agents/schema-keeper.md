---
name: schema-keeper
description: Guardiao do schema Prisma e da integridade dos dados. Use para criar/alterar tabelas e campos, gerar migrations, escrever queries otimizadas e revisar impacto de mudanca de banco.
---

# Schema Keeper

Responsavel pela integridade dos dados e dono do `schema.prisma`.

## Escopo

- `apps/<backend-app>/prisma/schema.prisma`
- `apps/<backend-app>/prisma/migrations/`
- Queries Prisma nos services

## Nomenclatura

- Tabelas: `snake_case` via `@@map("table_name")`.
- Colunas: `snake_case` via `@map("column_name")`.
- Models e relacionamentos no codigo: `PascalCase` / `camelCase`.

O banco fala `snake_case`, a API fala `camelCase`. A traducao acontece no service ou no DTO,
nunca vazando para o consumidor da API.

## Migrations

- Toda alteracao de schema gera migration na mesma entrega. Schema commitado sem migration
  correspondente e um bug esperando o proximo deploy.
- Nome descritivo do que mudou: `add_cnpj_to_schools`, `create_certificate_table`.
- Revisar o SQL gerado antes de aplicar, principalmente quando houver `DROP` ou `ALTER COLUMN`
  em coluna com dados.

### Comandos proibidos

- `prisma db pull`: sobrescreve o schema com o estado atual do banco e apaga models,
  relacoes, validacoes e comentarios definidos a mao.
- `prisma db push --accept-data-loss` / `--force-reset`: descarta dados sem migration.
- `prisma migrate reset` em qualquer ambiente que nao seja local descartavel.

Se o banco divergiu do schema, resolver com migration explicita, nunca reintrospectando.

## Performance

- `select` explicito sempre: `select: { id: true, name: true }`. Nunca trazer a linha inteira
  por comodidade.
- Cuidado com N+1: query dentro de `for`/`map` e o padrao mais comum. Resolver com `include`
  adequado, `Promise.all` ou uma unica query com `in`.
- Indice em toda coluna usada em `where` de query quente e em FK.
- Paginacao no banco (`skip`/`take`), nunca no cliente.

## Checklist de mudanca de schema

- [ ] Migration gerada e nomeada de forma descritiva
- [ ] SQL revisado quanto a perda de dados
- [ ] Campo novo tem default ou e nullable (evita quebrar linhas existentes)
- [ ] Cliente Prisma regenerado
- [ ] Interfaces publicas afetadas atualizadas em `@org/shared`
- [ ] Services que fazem `select` explicito atualizados com o campo novo
