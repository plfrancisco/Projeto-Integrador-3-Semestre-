# Índice de Tasks — Projeto Integrador

Este índice define qual documento controla a execução. A existência de uma task
histórica ou futura não autoriza implementação.

## Fonte canônica

Tasks são organizadas por especialidade — cada arquivo é a única fonte de
ordem, checkbox e status para seu domínio.

| Arquivo | Estado | Especialidade |
|---|---|---|
| *(nenhuma task ativa ainda)* | — | — |

## Documentos históricos

Arquivos concluídos ficam em `completed/`, fora do fluxo de execução ativo.

## Precedência

1. `.context/rules/ai_development_rules.md`
2. documentos normativos em `.context/plan/`
3. arquivo de task **ativo** da especialidade em questão
4. arquivos históricos somente para contexto

Conflito entre task e documento normativo deve ser apresentado ao usuário e
corrigido antes da implementação.
