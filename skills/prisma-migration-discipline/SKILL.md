---
name: prisma-migration-discipline
description: Disciplina de schema e migrations com Prisma - nomenclatura, comandos proibidos, seguranca de dados e performance de query. Use antes de alterar schema.prisma, gerar migration ou revisar query lenta.
---

# Disciplina de Schema e Migrations (Prisma)

## Nomenclatura

- Tabela: `snake_case` com `@@map("table_name")`.
- Coluna: `snake_case` com `@map("column_name")`.
- Model e campo no codigo: `PascalCase` / `camelCase`.

O banco fala `snake_case`; a API fala `camelCase`. A traducao e responsabilidade do backend,
nunca do consumidor.

## Migrations

- **Toda alteracao de schema gera migration na mesma entrega.** Schema commitado sem migration
  quebra o proximo deploy de quem der pull.
- Nome descritivo do efeito: `add_cnpj_to_schools`, `create_certificate_table`,
  `drop_legacy_status_column`.
- Ler o SQL gerado antes de aplicar. `DROP COLUMN` e `ALTER COLUMN` em tabela com dados
  exigem plano de migracao de dados, nao so de schema.
- Campo novo em tabela existente: nullable ou com default. `NOT NULL` sem default falha
  na hora de aplicar em base populada.

## Nome de migration sempre explicito

Nunca rodar `migrate dev` sem `--name`. Migration sem nome recebe sufixo generico e
inutiliza auditoria, review e rollback: o nome e o contrato publico da migration para o time.

```bash
prisma migrate dev --name add_cnpj_to_schools     # certo
prisma migrate dev                                 # proibido
```

## Comandos proibidos

| Comando | Por que |
| --- | --- |
| `prisma db pull` | reintrospecta e sobrescreve o schema, apagando models, relacoes, validacoes e comentarios escritos a mao |
| `prisma db push --accept-data-loss` | descarta dados sem migration e sem rastro |
| `prisma db push --force-reset` | recria o banco do zero |
| `prisma migrate reset` fora de ambiente local descartavel | apaga tudo |

Se o banco divergiu do schema, corrigir com migration explicita.

## Performance

- `select` explicito em toda query. Nunca trazer a linha inteira por comodidade:
  qualquer coluna nova passa a trafegar sem ninguem decidir.
- N+1: query dentro de loop e o caso mais comum. Resolver com `include` adequado,
  uma query com `in`, ou `Promise.all` quando as chamadas sao mesmo independentes.
- `include` aninhado profundo gera join caro: preferir consultas separadas e montar em memoria
  quando o volume permitir.
- Indice em coluna usada em `where` de query quente e em toda FK.
- Paginacao com `skip`/`take` no banco.
- Transacao (`$transaction`) quando duas escritas precisam ser atomicas. Sem isso o sistema
  fica com estado meio-gravado no primeiro erro.

## Shadow database

- Configurar `shadowDatabaseUrl` apontando para um banco separado. Nunca reutilizar o banco
  de desenvolvimento como shadow.
- Migrations precisam ser reproduziveis do zero no shadow. Se uma migration antiga falha no
  replay, corrigir aquela migration: nenhuma migration nova e criada enquanto o replay estiver
  quebrado.
- Quando o shadow estiver inutilizavel e a migration precisar sair, o fallback e escrever o
  SQL a mao no diretorio de migrations e aplicar com `prisma db execute`. Isso **nao** registra
  a migration na tabela de historico: documentar no PR e sincronizar o historico manualmente,
  senao os outros ambientes divergem em silencio.

## Cuidados por banco

**SQL Server**: quando a mesma migration adiciona uma coluna e depois a referencia
(`UPDATE`, `CHECK`, indice filtrado), o batch pode ser compilado antes do `ALTER TABLE` e
falhar com "nome de coluna invalido". Executar os comandos dependentes via SQL dinamico
(`EXEC sp_executesql`). Envolver a migration em transacao explicita com rollback no erro.

**Postgres**: `ALTER TABLE` que reescreve tabela grande trava escrita; para tabela quente,
preferir adicionar coluna nullable, backfill em lotes e so entao aplicar a constraint.

## Checklist antes de aplicar

- [ ] Migration gerada com nome descritivo
- [ ] SQL revisado quanto a perda de dados
- [ ] Campo novo nullable ou com default
- [ ] Cliente Prisma regenerado
- [ ] Interface publica correspondente atualizada na lib compartilhada
- [ ] Services com `select` explicito atualizados com o campo novo
- [ ] Seed ainda funciona
