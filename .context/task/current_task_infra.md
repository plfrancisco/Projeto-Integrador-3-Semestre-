# CURRENT TASK — INFRAESTRUTURA — Projeto Integrador

**Versão:** 1.0
**Status:** ATIVA
**Escopo:** ambiente de desenvolvimento, estrutura de pastas e orquestração
Docker

---

# 1. Objetivo

Preparar o ambiente e a estrutura de repositório sobre os quais backend,
frontend e modelo serão construídos. Nenhuma outra task depende de código
funcional aqui — apenas da estrutura existir.

---

# 2. Escopo

Cobre ambiente Python, estrutura de pastas e Docker Compose. Não cobre
código de aplicação (backend, frontend, modelo) nem o pipeline de treino,
que roda fora do Docker por decisão registrada em `tech_stack.md`, seção 6.1.

---

# 3. Tarefas

## INFRA01 — Ambiente Python

- [ ] Inicializar projeto com `uv` e `pyproject.toml`
- [ ] Gerar lockfile e exportar `requirements.txt` para o build Docker
- [ ] Configurar Ruff como linter/formatter

**Especificação:** `../plan/foundation/tech_stack.md`, seção 7.
**Dependências:** nenhuma.

## INFRA02 — Estrutura de pastas

- [ ] Criar `backend/src/{api,entities,repositories,inference,networks,config}`
- [ ] Criar `backend/training/`, `backend/migrations/`
- [ ] Criar `frontend/src/{components,pages,services}`
- [ ] Criar `data/` e `models/` (fora do versionamento, conforme `.gitignore`)
- [ ] Criar `notebooks/`

**Especificação:** `../plan/foundation/tech_stack.md`, seção 8.
**Dependências:** nenhuma.

**Nota:** a rede neural vai em `src/networks/`, não em `src/models/` —
entidades ORM ficam em `src/entities/`. A ambiguidade entre os dois sentidos
de "models" (ORM vs. rede) está resolvida em `tech_stack.md`, seção 8, e não
deve ser reaberta.

## INFRA03 — Docker Compose

- [ ] `docker-compose.yml` orquestrando PostgreSQL, backend (FastAPI) e
      frontend (React)
- [ ] Volume Docker para `data/uploads/`
- [ ] Volume Docker para `models/checkpoints/` (pesos do modelo)

**Especificação:** `../plan/foundation/tech_stack.md`, seção 6.1.
**Dependências:** INFRA01, INFRA02.

**Nota:** o treino do modelo **não** roda em container — GPU passthrough via
WSL2 não compensa o esforço de configuração no escopo deste projeto.

---

# 4. Ordem de execução

INFRA01 e INFRA02 podem rodar em paralelo. INFRA03 depende de ambos.

---

# 5. Critério de conclusão

Ambiente sobe com `docker compose up`, backend e frontend respondem (mesmo
vazios), Postgres aceita conexão, e a estrutura de pastas está criada
conforme especificado.

---

# FIM DA TASK — INFRAESTRUTURA
