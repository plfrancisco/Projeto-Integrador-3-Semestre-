# PROJECT OVERVIEW — Projeto Integrador

**Versão:** 1.0
**Status:** ATIVO
**Escopo:** visão geral, problema, escopo do MVP e abordagem técnica

---

# 1. Problema

Empresas que distribuem armadilhas adesivas de mosquito para mercados,
açougues e estabelecimentos similares dependem de inspeção manual periódica
para saber quando cada armadilha está saturada e precisa de troca. Esse
processo é impreciso: a troca pode ocorrer cedo demais (desperdício) ou tarde
demais (armadilha saturada perde eficácia).

---

# 2. Público-alvo

- Empresas que vendem/distribuem armadilhas adesivas de mosquito.
- Donos de estabelecimentos (mercados, açougues, etc.) que usam as armadilhas.

**Empresa parceira real:** existe contato com empresa do ramo para validação
futura e fornecimento de dados reais. O equipamento de referência é o
StickFly K-45 — ver `hardware_armadilha.md`.

---

# 3. Objetivo geral

Usar visão computacional para, a partir de uma imagem da armadilha (capturada
por uma câmera fixa acoplada a ela), identificar automaticamente o nível de
ocupação da superfície adesiva e estimar quando está na hora de trocar.

---

# 4. Modelos de uso possíveis

Alternativas a validar mais adiante:

- **B2B para a empresa fabricante/vendedora** — usa o sistema para saber quais
  clientes precisam de visita de troca.
- **App para o dono do estabelecimento** — monitora suas armadilhas e decide
  quando pedir troca.
- Ambos, com painéis diferentes sobre a mesma base.

> Não é necessário decidir isso agora — o projeto integrador foca no MVP
> técnico (modelo de visão computacional).

---

# 5. Escopo do projeto integrador

Este projeto é um **MVP de validação técnica**, não uma solução completa.

## 5.1 Entregas do MVP

1. **Baseline clássico (OpenCV)** — solução por threshold de cor, sem deep
   learning. Serve como referência comparativa para demonstrar, com
   evidência, que o modelo treinado agrega valor real.
2. **Modelo de segmentação treinado** — U-Net com encoder pré-treinado
   (transfer learning). Recebe imagem de armadilha, gera máscara de
   segmentação, calcula % de área coberta, classifica como `ok`, `atenção`
   ou `trocar`.
3. **Relatório de métricas comparativo** — IoU/Dice, erro percentual, matriz
   de confusão, gráficos de loss, exemplos visuais de predição vs. gabarito,
   comparando baseline vs. modelo treinado, inclusive sob variação de
   condições (iluminação, ângulo).
4. **Interface web de demonstração** — upload de foto, exibe máscara de
   segmentação, percentual de área e status. Para apresentação do projeto.

## 5.2 Fora de escopo

- Hardware/câmera real funcionando em campo 24/7.
- Painel/app completo para empresa ou estabelecimento.
- Autenticação, múltiplos usuários, modelo de negócio, cobrança.
- Integração real com câmeras IoT em campo.

---

# 6. Tipo de armadilha

Armadilha adesiva (placa/superfície com cola) — os mosquitos ficam
visivelmente grudados na superfície, o que torna viável estimar ocupação por
análise de imagem.

A superfície de interesse é o **refil adesivo de 450 × 220 mm**, não a
armadilha inteira. Especificações completas do equipamento em
`hardware_armadilha.md`.

---

# 7. Abordagem técnica

## 7.1 Decisão: segmentação por área coberta

Medir percentual de área coberta da superfície adesiva, em vez de contar
insetos individualmente.

- Motivo: mosquitos colados podem se sobrepor/aglomerar, tornando contagem
  individual difícil e propensa a erro. Área coberta é um indicador mais
  direto e robusto de saturação.
- Trata-se de um problema de **segmentação semântica** — separar pixels de
  "superfície limpa" dos pixels "cobertos por insetos/sujeira".
- **Modelo:** U-Net via `segmentation_models_pytorch`, com encoder ResNet34
  pré-treinado em ImageNet (transfer learning) — decisão motivada pelo
  dataset pequeno. Detalhes e alternativas avaliadas em `tech_stack.md`.
- **Baseline:** solução clássica com OpenCV (threshold de cor) implementada
  antes do modelo, como referência comparativa.

## 7.2 Classificação do status

Com base no percentual de área coberta, classificar em três faixas. Os
limiares abaixo são sugestões iniciais e devem ser calibrados com dados
reais.

| Status | Significado | Limiar sugerido |
|---|---|---|
| `ok` | Armadilha funcional, sem necessidade de troca | < 40% coberta |
| `atenção` | Armadilha se aproximando da saturação | 40% – 70% coberta |
| `trocar` | Armadilha saturada, eficácia comprometida | > 70% coberta |

## 7.3 Pipeline do MVP

```text
Imagem da armadilha (input)
        ↓
Pré-processamento (resize, normalização)
        ↓
Modelo de segmentação (U-Net)
        ↓
Máscara binária (área coberta vs. limpa)
        ↓
Cálculo do percentual de área coberta
        ↓
Classificação do status (ok / atenção / trocar)
        ↓
Output: máscara visual + percentual + status
```

---

# 8. Estratégia de dataset

## 8.1 Fase 1 — Dataset sintético

Gerar imagens artificiais de armadilhas adesivas com diferentes níveis de
ocupação. A vantagem é que as máscaras de segmentação são geradas junto com a
imagem, eliminando o trabalho de rotulagem manual.

Abordagem prevista:

- Base: imagem de placa adesiva limpa (fotografada ou gerada), respeitando a
  proporção de 450 × 220 mm do refil real.
- Sobreposição: insetos recortados ou gerados, posicionados aleatoriamente
  sobre a placa, com variação de quantidade, posição, escala e rotação.
- Augmentation: variações de iluminação, ângulo e ruído de câmera,
  reproduzindo as condições descritas em `hardware_armadilha.md`.
- A máscara é derivada automaticamente das posições dos insetos.

**Pendente:** foto real do refil da empresa parceira como referência visual
para calibrar a geração sintética.

## 8.2 Fase 2 — Dados reais

Após obtenção de material da empresa parceira:

- Coletar fotos reais de armadilhas em diferentes estágios de ocupação.
- Rotular semi-automaticamente, usando o modelo da Fase 1 como ponto de
  partida e corrigindo manualmente.
- Realizar fine-tuning do modelo treinado com dados sintéticos.
- Validar exclusivamente com dados reais nunca usados no treino.

---

# 9. Restrições

- Tecnologias: definidas em `tech_stack.md`.
- Regras de desenvolvimento: `.context/rules/ai_development_rules.md`.
- Integrantes do grupo: a definir.
- Prazo de entrega: a definir.

---

# 10. Pendências

- [ ] Confirmar limiares de classificação com dados reais
- [ ] Obter foto real do refil para calibrar a geração sintética
- [ ] Definir integrantes do grupo e divisão de responsabilidades
- [ ] Definir prazo de entrega

---

# FIM DO PROJECT OVERVIEW
