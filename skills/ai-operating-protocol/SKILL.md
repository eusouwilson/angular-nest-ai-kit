---
name: ai-operating-protocol
description: Protocolo operacional para agentes de IA em monorepo - precedencia de regras, o que confirmar antes de agir, guardrails de arquivos sensiveis e disciplina de documentacao. Use ao configurar as regras de IA de um projeto novo ou ao definir o que um agente pode fazer sem perguntar.
---

# Protocolo Operacional para Agentes de IA

Modelo de regras de projeto, sem amarra a um repositorio especifico. Adaptar e colocar no
arquivo de instrucoes da ferramenta (`CLAUDE.md`, `AGENTS.md`, regras do editor).

## Precedencia de regras

1. **Canonico**: um unico documento e a fonte da verdade de arquitetura e padroes.
   Os arquivos de instrucao das ferramentas sao derivados dele, nao editados a mao.
2. **Guardrails**: regras sempre ativas (seguranca, dominio, build/CI, protocolo).
3. **Especializadas**: regras por escopo (frontend, backend, testes), ativadas por contexto.

Em caso de divergencia, prevalece o canonico, e o arquivo divergente e corrigido para convergir.
Duas fontes discordando e pior do que uma fonte incompleta.

## Antes de comecar

- Confirmar **qual app/pacote** e o alvo quando o workspace tem mais de um.
- Ler o documento canonico de padroes antes de criar ou alterar algo estrutural.
- Verificar se o contrato publico envolvido ja existe na lib compartilhada.
- Perguntar quando duas leituras razoaveis do pedido levariam a trabalhos diferentes.
  Nao perguntar o que da para decidir com o codigo na frente.

## Durante

- Usar as skills e agentes do projeto quando a tarefa cair no escopo deles, em vez de gerar
  tudo do zero a cada vez.
- Ferramenta de consulta de documentacao de biblioteca antes de assumir API de framework.
  Se a consulta nao estiver disponivel, dizer isso em vez de chutar.
- Escrever teste antes do codigo de producao em componente ou service novo.

## Guardrails: exigem confirmacao explicita

- Pipeline de CI, `Dockerfile`, compose, workflow de build ou etapa de publicacao de imagem.
- Comando de banco destrutivo: reintrospeccao que sobrescreve schema, push com perda de dados,
  reset de migrations.
- Alteracao de regra de RBAC: explicar > propor diff > aguardar autorizacao > aplicar > documentar.
- Alteracao de layout ou fluxo visivel ao usuario final, mesmo dentro de correcao urgente.
- Mudanca nas proprias regras globais de arquitetura.

Confirmacao dada para uma acao nao se estende para a proxima.

## Depois

- Atualizar a documentacao do que foi tocado (modulo, endpoint, componente, regra) na mesma
  entrega. Doc "depois" nao acontece.
- Padrao reutilizavel novo entra no documento canonico, nao so no codigo.
- Rodar o verificador de padroes e o lint do pacote afetado antes de declarar concluido.
- Relatar o resultado como ele foi: teste falhando e teste falhando; etapa pulada e etapa pulada.

## Enforcement automatico

Regra sem gate e sugestao. As que valem mesmo devem ter verificacao automatica:

- Lint com regra customizada para os anti-patterns do projeto.
- Script de verificacao de padroes com baseline: nao exige limpar o legado de uma vez,
  mas impede piorar.
- Hooks que bloqueiam comando destrutivo e edicao de arquivo sensivel.

Documentar quais regras tem gate real e quais sao so texto: misturar as duas categorias faz
o time (e a IA) tratar todas como opcionais.

## Planejamento de feature media ou grande

Para mudanca que atinge dois ou mais pacotes, exige migration ou altera contrato publico:
usar um fluxo de especificacao antes de implementar (requisitos, design, tarefas, execucao),
com artefatos versionados. Feature pequena nao precisa de cerimonia.
