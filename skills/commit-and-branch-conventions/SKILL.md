---
name: commit-and-branch-conventions
description: Conventional Commits, nomeacao de branch e fluxo de integracao ate producao. Use ao escrever mensagem de commit, criar branch, abrir PR ou definir politica de branch protegida.
---

# Convencoes de Commit e Branch

## Commit

Formato: `tipo(escopo): descricao`

- **Tipos**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
  (`perf`, `ci`, `build` quando o projeto usar).
- **Escopo**: o pacote afetado (nome do app ou da lib). Um escopo por commit.
- **Subject**: modo imperativo, ate 50 caracteres, sem ponto final,
  sem mencao a ferramenta de IA.
- **Body** (quando necessario): explica **o que** e **por que**, nunca **como**.
  O "como" ja esta no diff; o "por que" so existe na sua cabeca.
- Commits separados por pacote quando a mudanca cruza varios.
- Cada commit e uma unidade coerente e testavel: se um commit sozinho quebra o build,
  ele nao deveria existir sozinho.

```
feat(api): implementar endpoint de listagem de reunioes
fix(web): corrigir overlay travado ao cancelar dialog de edicao
refactor(shared): extrair envelope de resposta para tipo proprio
```

## Nome de branch

Formato: `tipo/descricao-curta-em-kebab-case`

| Tipo | Uso |
| --- | --- |
| `feature/` | funcionalidade nova |
| `fix/` | correcao de bug |
| `hotfix/` | correcao urgente direto sobre producao |
| `refactor/` | mudanca estrutural sem mudar comportamento |
| `chore/` | manutencao, dependencia, configuracao |
| `docs/` | documentacao |
| `test/` | teste |
| `ci/` | pipeline e automacao |

Descricao curta, sem numero de ticket sozinho (`feature/1234` nao diz nada;
`feature/1234-exportacao-pdf` diz).

## Fluxo ate producao

```
integracao (deploy automatico no ambiente de desenvolvimento)
  └─ branch de trabalho (cortada da integracao)
       └─ PR -> integracao (build da pipeline + 1 reviewer, bloqueantes)
            └─ validacao no ambiente de desenvolvimento
                 └─ PR integracao -> producao (build + reviewer)
                      └─ deploy em producao com aprovacao manual
```

- Branch de trabalho sai da base de integracao, nunca da branch de producao.
- Nenhuma branch de trabalho vai direto para producao. Excecao: `hotfix/`, que sobe para
  producao e **retorna** para a integracao no mesmo dia, senao a correcao some no proximo merge.
- Nomes de branch principais precisam bater literalmente com o que a pipeline espera:
  uma variacao (`develop` em vez de `dev`) faz o deploy automatico simplesmente nao disparar.

## Politica de branch protegida

- Minimo 1 reviewer, bloqueante.
- Build de validacao da pipeline, bloqueante.
- Force push e delete negados para o time nas branches principais. Push normal, PR e merge
  seguem liberados.

## Anti-patterns

- Commit "wip", "ajustes", "correcoes" sem escopo nem contexto.
- Commit gigante que mistura refactor, feature e correcao: impossivel de reverter em parte.
- Mensagem em imperativo no titulo e narrativa no corpo descrevendo linha a linha o diff.
- Merge de branch de trabalho direto em producao pulando a integracao.
