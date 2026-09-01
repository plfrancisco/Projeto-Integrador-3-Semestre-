# CURRENT TASK — API — Projeto Integrador

**Versão:** 1.0
**Status:** ATIVA
**Escopo:** backend FastAPI — esqueleto, contrato de erro, stub de
inferência e os 8 endpoints definidos em `api_spec.md`

---

# 1. Objetivo

Implementar a API REST completa consumida pelo frontend, incluindo um stub
de inferência que permite construir e testar todo o sistema antes do modelo
treinado existir.

---

# 2. Escopo

Cobre os 8 endpoints de `api_spec.md`. O endpoint de análise usa
inicialmente o **stub de inferência** (API04) — a substituição pela
inferência real é responsabilidade de `current_task_modelo.md` (ML07), não
desta task.

**Leitura obrigatória adicional:** `../rules/security_spec.md` — em
especial as seções 3 (nenhum segredo em código), 5 (upload), 6.1 (SQL) e 9
(tratamento de erro), que se aplicam diretamente aos endpoints desta task.

---

# 3. Tarefas

## API01 — Esqueleto FastAPI

- [ ] App inicializa e expõe `/docs`
- [ ] CORS lido de `CORS_ORIGENS`
- [ ] Variáveis de ambiente lidas na inicialização: `DATABASE_URL`,
      `MODEL_WEIGHTS_PATH`, `MODEL_VERSION`, `UPLOADS_DIR`, `CORS_ORIGENS`,
      `LIMIAR_ATENCAO`, `LIMIAR_TROCAR`

**Especificação:** `../plan/api/api_spec.md`, seção 2.6.
**Dependências:** `current_task_infra.md` (INFRA01, INFRA02).

## API02 — Formato padronizado de erro

- [ ] Corpo de erro único: `{erro: {codigo, mensagem, detalhes}}`
- [ ] Todos os códigos da seção 2.3 implementados

**Especificação:** `../plan/api/api_spec.md`, seções 2.2 e 2.3.
**Dependências:** API01.

**Nota:** o frontend reage ao `codigo`, nunca ao texto da `mensagem`.

## API03 — Endpoints de armadilha

- [ ] `GET /api/armadilhas` — lista com `percentual_atual`, `status`,
      `dias_ate_saturar` e `em_alerta_desde` calculados no backend
- [ ] `POST /api/armadilhas` — com `criar_refil_inicial` opcional
- [ ] `GET /api/armadilhas/{id}` — detalhe com refil ativo e refis anteriores
- [ ] `PATCH /api/armadilhas/{id}` — `data_instalacao` não editável

**Especificação:** `../plan/api/api_spec.md`, seção 4.
**Dependências:** `current_task_banco.md` (DB02), API01, API02.

## API04 — Stub de inferência

- [ ] Função que recebe imagem e devolve `percentual_coberto`, `status`,
      `modelo_versao`, `mascara` no mesmo formato do contrato real
- [ ] Documentado como stub temporário, substituível sem alterar chamadores

**Especificação:** `../plan/model/modelo_spec.md`, seção 10.
**Dependências:** nenhuma.

## API05 — Cálculo de dias até saturar

- [ ] Regressão linear sobre **todas** as análises do refil ativo — nunca
      apenas a primeira e a última
- [ ] Casos sem cálculo (menos de 2 análises, inclinação ≤ 0) retornam
      nulo/zero conforme a tabela da especificação

**Especificação:** `../plan/frontend/telas_spec.md`, seção 5.
**Dependências:** API03.

**Nota:** o método de dois pontos foi descartado por sensibilidade a erro de
medição — a justificativa completa está na seção 5.1 da especificação e não
deve ser reaberta sem motivo novo.

## API06 — Endpoint de troca de refil

- [ ] `POST /api/armadilhas/{id}/refis` — encerra o refil ativo e cria o
      novo na mesma transação (atômico)
- [ ] Se não houver refil ativo, cria apenas o novo

**Especificação:** `../plan/api/api_spec.md`, seção 5.1.
**Dependências:** `current_task_banco.md` (DB03).

## API07 — Endpoint de análise

- [ ] `POST /api/analises` — valida arquivo, localiza refil ativo, persiste
      imagem, executa inferência (via stub, API04), persiste análise
- [ ] Falha na inferência não persiste registro parcial e remove a imagem
      gravada
- [ ] Validação de upload conforme `../rules/security_spec.md`, seção 5 —
      tipo real do arquivo (não extensão), limite de dimensões decodificadas,
      nome de arquivo gerado no servidor (nunca o do cliente)

**Especificação:** `../plan/api/api_spec.md`, seção 6.1.
**Dependências:** API03, API04.

## API08 — Histórico e imagens

- [ ] `GET /api/armadilhas/{id}/analises` — parâmetros `escopo`,
      `refil_id`, `limite`; `refil_id` em cada item retornado
- [ ] `GET /api/analises/{id}/imagem` e `/mascara` — binário com
      `Content-Type`, não base64 no JSON

**Especificação:** `../plan/api/api_spec.md`, seções 6.2 e 6.4.
**Dependências:** API07.

---

# 4. Ordem de execução

API01 → API02 → (API03 e API04 em paralelo) → API05, API06, API07 → API08.

---

# 5. Critério de conclusão

Os 8 endpoints respondem conforme `api_spec.md`, com o stub de inferência
no lugar do modelo real. Erros seguem o formato padronizado. Suite de testes
cobrindo os casos de exceção documentados (refil ausente, arquivo inválido,
etc.).

---

# FIM DA TASK — API
