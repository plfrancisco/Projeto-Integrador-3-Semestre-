# AI DEVELOPMENT RULES — Projeto Integrador

**Versão:** 1.0
**Status:** ATIVO
**Escopo:** desenvolvimento e manutenção com assistência de IA

---

# 1. Objetivo

Definir regras para o uso de IAs no desenvolvimento deste projeto, mantendo o
trabalho dentro do escopo solicitado e da estrutura documentada.

---

# 2. Princípio geral

> A IA pode analisar, explicar e sugerir livremente, mas somente pode executar
> alterações autorizadas pelo usuário e dentro do escopo solicitado.

Um pedido explícito para criar, corrigir, mover ou atualizar arquivos já
constitui autorização para executar aquela mudança.

---

# 3. Leitura antes de alterar

Antes de uma mudança relevante, a IA deve consultar:

1. a task aplicável em `.context/task/`
2. `.context/plan/foundation/project_overview.md`
3. a especificação do domínio afetado em `.context/plan/`
4. estas regras
5. os arquivos reais relacionados à solicitação

A IA não deve considerar uma estrutura planejada como implementada.

---

# 4. Escopo da autorização

## 4.1 Autorizado

Após pedido explícito, a IA pode:

- alterar os arquivos necessários ao objetivo solicitado
- criar arquivos em diretórios autorizados
- executar validações relacionadas à mudança
- realizar pequenos ajustes indispensáveis no mesmo domínio
- atualizar referências quebradas pela alteração aprovada

## 4.2 Exige nova confirmação

A IA deve solicitar nova confirmação quando precisar:

- ampliar o objetivo original
- modificar arquivos de outro domínio
- mudar arquitetura ou contrato
- criar uma dependência
- alterar schema ou dados do banco
- acessar ou modificar credenciais
- excluir arquivos ou dados
- executar comandos destrutivos
- substituir uma decisão humana registrada

---

# 5. Apresentação prévia

Em tarefas estruturais ou que atinjam vários arquivos, a IA deve apresentar:

- objetivo
- arquivos que pretende criar, alterar, mover ou remover
- resumo das mudanças
- validações previstas
- riscos ou decisões abertas

Correções pequenas e claramente solicitadas não exigem plano formal.

---

# 6. Organização

A IA deve:

- usar `snake_case` nos documentos da `.context`
- respeitar a estrutura de pastas documentada
- evitar arquivos genéricos na raiz
- não criar estruturas concorrentes
- atualizar referências quando um arquivo autorizado for movido ou renomeado

Se a localização necessária não estiver documentada, a IA deve pedir decisão
ao usuário.

## 6.1 Padrão de formatação dos documentos

Todo documento da `.context` segue o mesmo formato, para que a leitura seja
previsível e a navegação entre documentos não exija reaprender a estrutura:

1. **Título em caixa alta**, seguido de travessão e do contexto:
   `# NOME DO DOCUMENTO — Projeto Integrador`
2. **Bloco de metadados** logo abaixo do título, sem linha em branco entre
   os campos:
   ```text
   **Versão:** 1.0
   **Status:** ATIVO | RASCUNHO | CONCLUÍDO
   **Escopo:** uma linha descrevendo o que o documento cobre
   ```
3. **Separador `---`** entre todas as seções de primeiro nível.
4. **Seções numeradas em H1**: `# 1. Objetivo`, `# 2. Escopo`.
5. **Subseções numeradas em H2**: `## 4.1 Autorizado`.
6. **Quebra de linha em ~78 colunas.** Tabelas são exceção — quebrá-las
   inviabiliza a renderização.
7. **Blocos de código sempre com linguagem declarada** (` ```text `,
   ` ```python `), para evitar realce incorreto.
8. **Marcador de fim**: `# FIM DO NOME DO DOCUMENTO`.

Documentos com pendências devem encerrar com uma seção de pendências em
formato de checklist, antes do marcador de fim.

---

# 7. Proteção de documentos

## 7.1 Documentos normativos

A IA não pode alterar silenciosamente:

- regras de engenharia
- contratos e especificações técnicas
- regras de banco
- estrutura oficial do projeto

Mudanças devem estar diretamente relacionadas ao pedido do usuário.

## 7.2 Documentos históricos

Tasks concluídas em `.context/task/completed/` são histórico consultável.
Não podem ser alteradas sem autorização explícita.

---

# 8. Regras para código

A IA deve:

- limitar o código ao objetivo aprovado
- seguir os contratos documentados
- evitar duplicação de lógica
- preservar alterações existentes de outras pessoas
- usar tipagem quando suportada
- tratar erros de forma explícita
- validar entradas nas fronteiras do sistema
- preferir soluções simples a abstrações prematuras
- seguir formatadores, linters e convenções já adotados no repositório

A IA não pode:

- inventar requisitos
- implementar módulos planejados sem solicitação
- adicionar credenciais ao código
- corrigir problemas não relacionados sem informar o usuário

---

# 9. Regras específicas de visão computacional e ML

## 9.1 Reprodutibilidade

- Toda conclusão sobre o modelo deve ser baseada em métricas reproduzíveis.
- Conjuntos de treino, validação e teste devem permanecer separados — sem
  imagens de teste usadas no treinamento.
- Seeds, configurações de treinamento e versões relevantes devem ser
  registradas.

## 9.2 Arquitetura do modelo

- O MVP usa Python e segmentação baseada em U-Net ou similar, preferencialmente
  com PyTorch.
- Dados, treinamento, inferência, avaliação e configurações devem permanecer
  em módulos separados.
- Notebooks podem ser usados para exploração, mas código reutilizável deve
  ficar em módulos Python.

## 9.3 Testes e métricas

- Testes unitários para pré-processamento, cálculo da área coberta e
  classificação do status.
- Teste de integração para o fluxo completo: imagem → segmentação →
  percentual → status.
- Avaliação do modelo deve incluir, quando aplicável: IoU/Dice, erro
  percentual e matriz de confusão.

## 9.4 Dados e artefatos

- Datasets grandes, modelos treinados e artefatos gerados devem ficar fora
  do versionamento Git (conforme `.gitignore`).
- Dados e máscaras devem ter origem e divisão documentadas.
- A avaliação deve usar conjunto de teste com gabarito conhecido, sem depender
  de inspeção manual não registrada.

## 9.5 Escopo do MVP

O MVP limita-se a estimar a porcentagem de área ocupada da armadilha e
classificar o resultado como `ok`, `atencao` ou `trocar`. Hardware real,
autenticação, painel completo, modelo de negócio e integração IoT permanecem
fora do escopo inicial, salvo aprovação explícita.

---

# 10. Validação

Após uma alteração, a IA deve executar validações proporcionais ao risco e
informar:

- o que foi validado
- o resultado
- o que não pôde ser validado
- riscos ou pendências

Uma tarefa não pode ser declarada pronta quando uma validação obrigatória
falhar.

---

# 11. Conclusão e atualização da task

A IA não deve declarar unilateralmente que uma task foi concluída.

```text
alteração executada
        ↓
validações realizadas
        ↓
resultado apresentado
        ↓
usuário confirma a conclusão
        ↓
task aplicável é atualizada
```

---

# 12. Comunicação final

Ao terminar uma alteração, a IA deve apresentar:

- resultado alcançado
- arquivos alterados
- validações executadas
- falhas ou limitações
- próxima ação, somente quando necessária

---

# 13. Conflitos e ausência de regra

Quando encontrar conflito entre documentação, código e task:

1. identificar a divergência
2. não escolher silenciosamente uma versão
3. informar o usuário
4. solicitar decisão quando a escolha alterar o resultado

Quando uma situação não estiver coberta, adotar a alternativa mais
conservadora que permita avançar sem sair do escopo.

---

# FIM DAS AI DEVELOPMENT RULES
