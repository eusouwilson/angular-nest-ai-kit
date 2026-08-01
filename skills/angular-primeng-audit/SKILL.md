---
name: angular-primeng-audit
description: Audit Angular + PrimeNG components for compliance with smart/dumb boundaries and modern Angular APIs. Use when reviewing or fixing p-table, p-dialog, p-select, signal inputs/outputs, RBAC template visibility, PrimeNG imports, deprecated DropdownModule usage, inline templates/styles, or dialog overlay close bugs.
---

# Angular + PrimeNG Audit

Revisao focada de conformidade de UI PrimeNG em componentes existentes.

## Fontes obrigatorias

- `references/primeng-audit-checklist.md` (nesta skill).
- Doc de templates do projeto destino, se existir: prevalece sobre esta skill.

## Fluxo de auditoria

1. Identificar o tipo do componente: smart page, dumb list, dialog create/edit,
   dialog de detalhe ou UI compartilhada.
2. Comparar `.ts`, `.html` e `.scss` juntos: muita violacao so aparece no cruzamento.
3. Reportar achados com referencia concreta de arquivo e linha.
4. Se pedirem correcao, alterar o minimo de arquivos e preservar comportamento local.
5. Rodar o teste/build mais focado disponivel apos corrigir.

## Checagens

- Estrutura: standalone, `ChangeDetectionStrategy.OnPush` em dumbs, HTML/SCSS separados.
- API: `input()` / `output()` em dumbs; `@Input()` / `@Output()` so em legado fora de escopo.
- DI: `inject()` apenas; sem DI por construtor em codigo novo/refatorado.
- Fronteira Smart/Dumb: dumb nao injeta facade/service e nao faz HTTP.
- Templates: `@if` / `@for`, leitura de signal com `()`, sem `*ngIf` / `*ngFor`.
- PrimeNG: modulo exato por seletor; `SelectModule`/`p-select` no lugar de `DropdownModule`.
- Overlay: `appendTo="body"` em selects dentro de dialog ou tabela.
- Dialog: pai com `@if` exige `dialogVisible` interno; X e footer chamam `closeDialog()`,
  nunca `cancel.emit()` direto.
- RBAC: acoes que modificam dados protegidas por diretiva de permissao.
- Forms: submit desabilitado (nao escondido) em invalido/carregando; mensagem de validacao
  exibida apos `touched` + `invalid`.

## Formato do relatorio

Por achado: severidade (alta/media), arquivo:linha, regra violada, correcao proposta.
Agrupar por severidade, alta primeiro. Nao misturar sugestao de estilo com defeito real.

## Alinhamento de arquitetura

Estrutura assumida: frontend por dominio com `data-access/`, `feature/`, `ui/`; contratos
HTTP publicos na lib compartilhada. Se o projeto destino diverge, ajustar antes de auditar
para nao gerar achado falso positivo.
