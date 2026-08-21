# DATA MODEL — Projeto Integrador

**Versão:** 0.1
**Status:** RASCUNHO
**Escopo:** modelo de dados, entidades e regras de persistência

---

# 1. Objetivo

Especificar o modelo de dados do projeto: entidades, atributos,
relacionamentos e regras de integridade.

---

# 2. Situação atual

O MVP roda de forma **stateless** — o modelo recebe uma imagem e retorna o
resultado sem persistir nada. O banco de dados entra em cena quando houver
necessidade de cadastro de armadilhas, estabelecimentos e histórico de
análises.

Ver `.context/plan/foundation/tech_stack.md`, seção 5.

---

# 3. Entidades

A definir. Candidatas identificadas até o momento:

- **Estabelecimento** — cliente onde as armadilhas estão instaladas
- **Armadilha** — unidade física instalada (modelo, localização, data de
  instalação)
- **Análise** — resultado de uma inferência (data, percentual de área
  coberta, status, imagem de origem)
- **Refil** — histórico de trocas de refil por armadilha

---

# 4. Tecnologia

PostgreSQL, conforme `tech_stack.md`. ORM a definir (SQLAlchemy ou Tortoise).

---

# 5. Diagrama

A definir. Usar Mermaid para manter o diagrama versionado junto ao markdown.

---

# 6. Pendências

- [ ] Confirmar se o banco entra no escopo do MVP ou fica para fase posterior
- [ ] Definir entidades e atributos
- [ ] Definir relacionamentos e regras de integridade
- [ ] Escolher ORM
- [ ] Produzir diagrama ER

---

# FIM DO DATA MODEL
