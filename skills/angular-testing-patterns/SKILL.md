---
name: angular-testing-patterns
description: Padroes de teste para Angular standalone com Signals (Jasmine/Karma) e disciplina de TDD. Use ao escrever spec de componente ou service, mockar facade com signals, ou definir o que cobrir antes de implementar.
---

# Padroes de Teste Angular

## Ciclo TDD (obrigatorio para componente ou service novo)

1. **Red**: escrever o teste antes do codigo de producao e conferir que ele falha
   **pelo motivo certo** (a implementacao nao existe, nao um erro de sintaxe).
2. **Green**: escrever o minimo para o teste passar. Passou, para de adicionar logica.
3. **Refactor**: remover duplicacao e melhorar nomes, mantendo os testes verdes.

Repetir por comportamento, nao por arquivo. Nunca escrever codigo de producao sem um teste
falhando antes.

Backend: rodar a suite antes de subir o servidor local. Frontend: nao e obrigatorio antes do
serve, mas a suite precisa estar verde antes de abrir PR.

## Estrutura da spec

- `TestBed.configureTestingModule` com o componente standalone em `imports`.
- `NoopAnimationsModule` em todo teste que renderiza componente de UI: sem isso, animacao
  gera flake intermitente.
- Padrao AAA (Arrange, Act, Assert), sem misturar etapas dentro do mesmo bloco.
- `jasmine.createSpyObj` para service; facade mockada com signals reais.
- `fixture.detectChanges()` antes de qualquer assert sobre o template.
- Ler signal com chamada: `component.items()`.

```typescript
const facadeMock = {
  items: signal<IProduct[]>([]),
  isLoading: signal(false),
  loadAll: jasmine.createSpy('loadAll'),
  toggleCreateDialog: jasmine.createSpy('toggleCreateDialog'),
};
```

Mock com signal real (e nao array estatico) e o que permite testar reatividade:
`facadeMock.items.set([...])` seguido de `detectChanges()` verifica o render de verdade.

## Assincronia

- `fakeAsync` + `tick()` para fluxo baseado em Observable/timer.
- `TestScheduler` quando o teste e sobre a forma do stream, nao sobre o resultado.
- `await fixture.whenStable()` para promise encadeada.

## O que cobrir

- Estados da tela: vazio, carregando, com dados, erro.
- Emissao de output ao interagir: clicar em editar emite exatamente o item da linha.
- Render condicional nos dois ramos do `@if`.
- Formulario: submit bloqueado enquanto invalido; payload emitido no formato certo.
- Componente com `effect()`: mudanca do `input()` reflete no formulario.

## O que nao cobrir

- Comportamento interno da biblioteca de UI (paginacao da tabela, animacao do dialog).
  Testar o uso, nao a lib.
- Metodo privado: se renomear quebra o teste, o teste esta acoplado demais.
- CSS e detalhe visual.

## Armadilha de ambiente

Runner headless costuma exigir o caminho do Chrome em variavel de ambiente (`CHROME_BIN`).
Sem isso o comando de teste trava sem mensagem util. Conferir isso antes de suspeitar do teste.
