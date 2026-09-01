# CURRENT TASK — CONFIGURAÇÃO DE AMBIENTE — Projeto Integrador

**Versão:** 1.0
**Status:** ATIVA
**Escopo:** o que cada integrante do grupo precisa preparar na própria
máquina antes de qualquer outra task começar

---

# 1. Objetivo

Garantir que qualquer pessoa do grupo consiga clonar o repositório e ter o
projeto rodando localmente, sem depender de conhecimento não documentado.
Esta task é **pré-requisito de todas as outras** — nenhuma task de
`current_task_infra.md` em diante deve ser iniciada antes desta estar
concluída.

Diferença em relação a `current_task_infra.md`: aquela cria a estrutura do
repositório (uma vez, por quem inicia o projeto); esta é o que **cada
integrante** repete na própria máquina para conseguir trabalhar.

---

# 2. Escopo

Cobre pré-requisitos de software, variáveis de ambiente e verificação de que
o ambiente sobe. Não cobre a criação da estrutura de pastas em si
(`current_task_infra.md`, INFRA02) nem a escrita de código de aplicação.

---

# 3. Tarefas

## AMB01 — Pré-requisitos de software

- [ ] Git instalado
- [ ] Docker Desktop instalado e em execução
- [ ] Node.js LTS instalado (para rodar o frontend fora do Docker durante o
      desenvolvimento, se necessário)
- [ ] Python 3.11+ instalado
- [ ] `uv` instalado (`pip install uv` ou instalador nativo)

**Especificação:** `../plan/foundation/tech_stack.md`, seções 2, 3 e 7.
**Dependências:** nenhuma.

## AMB02 — Clonar e configurar o repositório

- [ ] Clonar `https://github.com/plfrancisco/Projeto-Integrador-3-Semestre-`
- [ ] Copiar `backend/.env.example` para `backend/.env` e preencher os
      valores de desenvolvimento (ver AMB03)
- [ ] Copiar `frontend/.env.example` para `frontend/.env`

**Especificação:** `../plan/api/api_spec.md`, seção 2.6.
**Dependências:** AMB01.

## AMB03 — Arquivos de exemplo de variáveis de ambiente

- [ ] Criar `backend/.env.example` com as variáveis da tabela abaixo
- [ ] Criar `frontend/.env.example` com `VITE_API_URL`
- [ ] Confirmar que `.env` (sem `.example`) está no `.gitignore` — nunca
      versionar valores reais

**Conteúdo de referência para `backend/.env.example`:**

```text
DATABASE_URL=postgresql://user:senha@db:5432/armadilhas
MODEL_WEIGHTS_PATH=/app/models/checkpoints/latest.pt
MODEL_VERSION=v0.1.0-dev
UPLOADS_DIR=/app/data/uploads
CORS_ORIGENS=http://localhost:5173
LIMIAR_ATENCAO=40
LIMIAR_TROCAR=70
```

**Conteúdo de referência para `frontend/.env.example`:**

```text
VITE_API_URL=http://localhost:8000/api
```

**Especificação:** `../plan/api/api_spec.md`, seção 2.6 (tabela completa de
variáveis, obrigatoriedade e valores padrão).
**Dependências:** nenhuma — pode ser feita antes de AMB02 por quem primeiro
monta a estrutura, e reaproveitada pelos demais integrantes.

**Nota:** `MODEL_WEIGHTS_PATH` aponta para um arquivo que só existirá após
`current_task_modelo.md` (ML04). Até lá, o backend sobe com o stub de
inferência (`current_task_api.md`, API04), que não depende deste arquivo.

## AMB04 — Verificação

- [ ] `docker compose up` sobe sem erro (Postgres, backend, frontend)
- [ ] Backend responde em `/docs`
- [ ] Frontend carrega em `http://localhost:5173`
- [ ] `uv run ruff check .` roda sem erro de configuração (mesmo com poucos
      arquivos ainda)

**Dependências:** AMB01, AMB02, AMB03, e a estrutura de
`current_task_infra.md` já criada.

---

# 4. Ordem de execução

AMB01 → AMB02 e AMB03 (podem ser paralelas, mas AMB03 idealmente é feita
uma vez e commitada por quem monta a estrutura inicial) → AMB04.

---

# 5. Critério de conclusão

Todo integrante do grupo tem o projeto rodando localmente via
`docker compose up`, com os três serviços respondendo. Esta task é concluída
por pessoa, não uma vez só pelo grupo — cada novo integrante repete AMB01 a
AMB04.

---

# FIM DA TASK — CONFIGURAÇÃO DE AMBIENTE
