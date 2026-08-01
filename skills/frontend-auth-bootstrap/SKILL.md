---
name: frontend-auth-bootstrap
description: Padroes de autenticacao no frontend - SSO centralizado entre varios apps, inicializacao sem race condition e carregamento de permissoes antes do primeiro render. Use ao integrar um app novo ao provedor de identidade, corrigir bug de token ausente na primeira request ou tela que renderiza sem permissao.
---

# Autenticacao e Bootstrap no Frontend

Dois problemas distintos, resolvidos juntos: **de onde vem a sessao** (SSO) e
**o que precisa estar pronto antes do primeiro render** (bootstrap).

## SSO centralizado (plataforma com varios frontends)

Um app e o provedor de identidade; os demais nao tem tela de login.

```
App sem sessao -> redireciona para o provedor (com returnUrl e id do app)
  -> provedor autentica
  -> provedor emite token de troca, validando que o usuario tem acesso AQUELE app
  -> volta para returnUrl com o token
  -> app troca o token por sessao (com retry e backup em sessionStorage)
  -> app revalida o acesso ao proprio app
  -> 403 -> signOut + tela de acesso negado; OK -> renderiza
```

### Regras para app consumidor

1. **Sem login proprio**: nada de popup de login ou tela de credenciais. Rota `/login`
   redireciona para a raiz.
2. **Cliente de SSO compartilhado**: uma lib com o cliente, o interceptor e a config.
   Nao reimplementar por app: o comportamento precisa ser identico em todos.
3. **Config por ambiente**: URL do provedor, id do app e URL da API. Mais a chave do app,
   que o interceptor envia em toda request autenticada.
4. **Guard local** por app, mas injetando o cliente compartilhado: `async`, aguarda o
   `authReady`, checa `accessDenied()` antes de `isAuthenticated()`, e ao falhar redireciona
   para o provedor (nao para `/login`).
5. **Interceptor compartilhado**: injeta `Authorization` e o header de identificacao do app;
   trata 401 de forma unica.
6. **Expiracao**: quando a sessao cai depois de valida, redirecionar para o provedor.
7. **Loop de redirect**: o provedor marca a tentativa com um parametro; o cliente nao
   redireciona de novo se ele estiver presente.
8. **Token copiado**: revalidar que o usuario tem acesso ao app **atual**, nao apenas que o
   token e valido.

### Anti-patterns

- Login proprio em app consumidor.
- Validar acesso so no fluxo do token e nao na sessao ja existente.
- `catch` que retorna `true` na verificacao de acesso: fail-open. Verificacao de acesso e
  fail-closed, sempre.
- Excluir a conta do usuario para negar acesso em vez de encerrar a sessao.

## Bootstrap sem race condition

O inicializador da aplicacao precisa deixar **tres coisas** prontas antes do primeiro render:
sessao, usuario e permissoes.

```typescript
export function initializeAuthFactory(
  ssoAuth: SsoAuthClient,
  userPermissions: UserPermissionsService,
): () => Promise<void> {
  return async () => {
    await ssoAuth.initializeAuthState();
    if (ssoAuth.isAuthenticated()) {
      await userPermissions.ensureLoaded(); // fail-open: nunca rejeita e nao trava o boot
    }
  };
}
```

### Regras

- A chamada que carrega o usuario e sempre `await`. Fire-and-forget aqui produz bug
  intermitente que so aparece em rede lenta.
- No fluxo de login, popular o signal de usuario **antes** da chamada a API: a subscription
  do estado de auth resolve em microtask e chega tarde demais para o interceptor montar o
  header de autorizacao.
- Nao disparar o carregamento do usuario dentro da subscription do estado de auth no
  construtor: gera chamada duplicada e corrida com o token.
- Permissoes resolvidas por chamada separada precisam ser aguardadas no inicializador.
  Sem isso, um componente que decide o que carregar por `hasPermission(...)` no construtor
  roda com permissao vazia e carrega a coisa errada.
- Deduplicar buscas concorrentes da mesma permissao (reusar a Promise em voo): o efeito de
  runtime e o bootstrap disparam a mesma busca quase ao mesmo tempo.
- `ensureLoaded()` nunca lanca: falha de rede deixa a lista vazia e o usuario so nao ve
  telas restritas ate recarregar. Travar o boot inteiro por isso e pior.

### Consequencia util

Com o bootstrap garantido, componente **nao** precisa reimplementar espera reativa
(polling de RxJS, flags manuais, `waitForUser`). Esses paliativos costumam aparecer em
guard, topbar e paginas de dashboard; sao sintoma de bootstrap incompleto, nao solucao.

## Ao investigar "usuario sem permissao"

Antes de assumir problema de timing no frontend, conferir se o backend nao esta checando
nome de permissao antigo que ninguem migrou. Permissao renomeada e nao propagada faz todo
usuario cair no fallback restritivo, e o sintoma e identico ao de race condition.
