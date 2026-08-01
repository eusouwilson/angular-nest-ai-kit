---
name: monorepo-workspace-standards
description: Padroes de monorepo Nx - libs compartilhadas, path aliases, dependencias na raiz, cache, affected, Docker e CI. Use ao criar lib, adicionar dependencia, resolver erro de resolucao de modulo, ajustar build ou tocar em pipeline.
---

# Padroes de Monorepo (Nx)

## Estrutura

```
workspace/
├── apps/
│   ├── <frontend-app>/
│   └── <backend-app>/
├── libs/
│   ├── shared/             # @org/shared
│   └── auth-contracts/     # @org/auth-contracts
├── nx.json
├── tsconfig.base.json
└── package.json            # workspaces
```

Dependencia unidirecional: apps dependem de libs; libs nao dependem de apps nem umas das
outras sem necessidade real.

## Dependencias

- Instalar sempre na raiz do workspace.
- Nunca rodar instalacao dentro de `apps/` ou `libs/`, nem criar `node_modules` local:
  quebra o hoisting e produz duas versoes da mesma lib em runtime.
- A versao de TypeScript e ditada pela restricao mais rigida do workspace (normalmente o
  framework de frontend). Subir por conta quebra o build.
- Conflito de peer dependency entre frameworks diferentes e esperado; resolver com a flag
  documentada no projeto, nao apagando lockfile.

## Path aliases

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@org/shared": ["libs/shared/src/index.ts"],
      "@org/auth-contracts": ["libs/auth-contracts/src/index.ts"]
    }
  }
}
```

Apps estendem o tsconfig base e podem precisar repetir os paths localmente. Import entre
projetos sempre pelo alias; caminho relativo atravessando fronteira de projeto e proibido.

## Cache e affected

- O cache e por hash dos inputs: mesmo input, mesmo output reaproveitado. O diretorio de
  cache fica fora do controle de versao.
- `affected` usa o grafo real de imports: mudanca em lib atinge todos os apps que a importam;
  mudanca em config raiz atinge tudo.
- Local, comparar com o commit anterior; em PR, comparar com o `merge-base` da branch base.
  Comparacoes diferentes dao resultados diferentes, e a do CI e a que vale.
- **No CI, o passo que calcula `affected` precisa do historico completo do git.** Com clone
  raso a comparacao falha silenciosamente e o pipeline builda tudo ou nada.

## Docker

- Contexto de build e sempre a raiz do monorepo.
- Ordem dos `COPY`: manifestos e configs da raiz, depois as libs, e por ultimo o source do app.
  Inverter isso invalida cache de layer a cada mudanca de codigo.
- Multi-stage: builder com toolchain completa, runner enxuto.
- **Lib nova consumida por um app precisa entrar no `COPY` do Dockerfile daquele app.**
  E a falha mais comum do monorepo: builda local, quebra no container.

## Guardrail de CI e build

Alterar pipeline de CI, `Dockerfile`, compose ou workflow de build **exige confirmacao
explicita do usuario na conversa**. Inclui mudar `FROM`, `COPY`, `RUN`, `CMD`, variaveis de
build e etapas de multi-stage. Motivo: o erro so aparece no deploy, longe de quem escreveu.

## Fluxo de branches

```
base de integracao (deploy automatico em ambiente de desenvolvimento)
  └─ branch de trabalho (cortada da base de integracao)
       └─ PR -> integracao   (build da pipeline + reviewer obrigatorios)
            └─ validado no ambiente de desenvolvimento
                 └─ PR integracao -> producao (build + reviewer)
                      └─ deploy em producao com aprovacao manual
```

- Branch de trabalho sempre cortada da base de integracao, nunca da branch de producao.
- Se a pipeline reconhece os nomes de branch literalmente, usar exatamente esses nomes:
  `develop` no lugar de `dev` e o suficiente para o deploy automatico nunca disparar.
- Branches protegidas: force push negado, PR com reviewer e build obrigatorios.

## Troubleshooting

| Sintoma | Causa provavel | Acao |
| --- | --- | --- |
| `affected` nao ve mudanca em lib | app nao importa a lib | conferir import pelo alias |
| `TS1205: Re-exporting a type` | `isolatedModules` | usar `export type { }` |
| `Cannot find module @org/shared` | alias faltando | conferir `paths` na base e no tsconfig do app |
| build lento sem motivo | cache invalidado | conferir diretorio de cache; resetar |
| `COPY` falha no Docker | contexto errado | buildar da raiz |
| grafo sem a dependencia | import inexistente | o grafo reflete import real, nao intencao |
