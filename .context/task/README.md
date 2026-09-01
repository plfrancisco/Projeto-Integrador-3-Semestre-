# ÍNDICE DE TASKS — Projeto Integrador

**Versão:** 1.0
**Status:** ATIVO
**Escopo:** controle de execução e precedência entre tasks

---

# 1. Objetivo

Este índice define qual documento controla a execução. A existência de uma
task histórica ou futura não autoriza implementação.

---

# 2. Fonte canônica

Tasks são organizadas por especialidade — cada arquivo é a única fonte de
ordem, checkbox e status para seu domínio.

| Arquivo | Estado | Especialidade |
|---|---|---|
| *(nenhuma task ativa ainda)* | — | — |

---

# 3. Documentos históricos

Arquivos concluídos ficam em `completed/`, fora do fluxo de execução ativo.
Não podem ser alterados sem autorização explícita.

---

# 4. Precedência

1. `.context/rules/ai_development_rules.md`
2. documentos normativos em `.context/plan/`
3. arquivo de task **ativo** da especialidade em questão
4. arquivos históricos, somente para contexto

Conflito entre task e documento normativo deve ser apresentado ao usuário e
corrigido antes da implementação.

---

# FIM DO ÍNDICE DE TASKS
