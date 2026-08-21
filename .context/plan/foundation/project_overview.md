# Overview

## Problema

Empresas que distribuem armadilhas adesivas de mosquito para mercados, açougues
e estabelecimentos similares dependem de inspeção manual periódica para saber
quando cada armadilha está saturada e precisa de troca. Esse processo é
impreciso: a troca pode ocorrer cedo demais (desperdício) ou tarde demais
(armadilha saturada perde eficácia).

## Público-alvo

- Empresas que vendem/distribuem armadilhas adesivas de mosquito.
- Donos de estabelecimentos (mercados, açougues, etc.) que usam as armadilhas.

**Empresa parceira real:** existe contato com empresa do ramo para validação
futura e fornecimento de dados reais.

## Objetivo geral

Usar visão computacional para, a partir de uma imagem da armadilha (capturada
por uma câmera fixa acoplada a ela), identificar automaticamente o nível de
ocupação da superfície adesiva e estimar quando está na hora de trocar.

## Possíveis modelos de uso (a definir/validar mais adiante)

- **B2B para a empresa fabricante/vendedora** — usa o sistema para saber quais
  clientes precisam de visita de troca.
- **App para o dono do estabelecimento** — monitora suas armadilhas e decide
  quando pedir troca.
- Ambos, com painéis diferentes sobre a mesma base.

> Não é necessário decidir isso agora — o projeto integrador foca no MVP
> técnico (modelo de visão computacional).

## Escopo do projeto integrador (MVP)

Este projeto é um **MVP de validação técnica**, não uma solução completa.

### Entregas do MVP

1. **Baseline clássico (OpenCV)** — solução por threshold de cor, sem deep
   learning. Serve como referência comparativa para demonstrar, com evidência,
   que o modelo treinado agrega valor real.
2. **Modelo de segmentação treinado** — U-Net com encoder pré-treinado
   (transfer learning). Recebe imagem de armadilha, gera máscara de
   segmentação, calcula % de área coberta, classifica como `ok`, `atenção`
   ou `trocar`.
3. **Relatório de métricas comparativo** — IoU/Dice, erro percentual, matriz de
   confusão, gráficos de loss, exemplos visuais de predição vs. gabarito,
   **comparando baseline vs. modelo treinado**, inclusive sob variação de
   condições (iluminação, ângulo).
4. **Interface web simples de demonstração** — upload de foto → exibe máscara
   de segmentação + % de área + status. Para apresentação do projeto.

### Fora de escopo por ora

- Hardware/câmera real funcionando em campo 24/7.
- Painel/app completo para empresa ou estabelecimento.
- Autenticação, múltiplos usuários, modelo de negócio, cobrança.
- Integração real com câmeras IoT em campo.

## Tipo de armadilha

Armadilha adesiva (placa/superfície com cola) — os mosquitos ficam visivelmente
grudados na superfície, o que torna viável estimar ocupação por análise de
imagem. Referência visual da armadilha real da empresa parceira: **pendente**
(atualizar quando disponível).

## Abordagem técnica

### Decisão: segmentação por área coberta

Medir % de área coberta da superfície adesiva, em vez de contar insetos
individualmente.

- Motivo: mosquitos colados podem se sobrepor/aglomerar, tornando contagem
  individual difícil e propensa a erro. Área coberta é um indicador mais direto
  e robusto de saturação.
- Abordagem é um problema de **segmentação semântica** — separar pixels de
  "superfície limpa" dos pixels "cobertos por insetos/sujeira".
- **Modelo:** U-Net via `segmentation_models_pytorch`, com encoder ResNet34
  pré-treinado em ImageNet (transfer learning) — decisão motivada pelo dataset
  pequeno. Detalhes e alternativas avaliadas em
  [`tech_stack.md`](tech_stack.md).
- **Baseline:** solução clássica com OpenCV (threshold de cor) implementada
  antes do modelo, como referência comparativa.

### Classificação do status

Com base no % de área coberta, classificar em 3 faixas (limiares a calibrar):

| Status | Significado | Limiar sugerido (a validar) |
|---|---|---|
| `ok` | Armadilha funcional, sem necessidade de troca | < 40% coberta |
| `atenção` | Armadilha se aproximando da saturação | 40% – 70% coberta |
| `trocar` | Armadilha saturada, eficácia comprometida | > 70% coberta |

### Pipeline completo do MVP

```
Imagem da armadilha (input)
        ↓
Pré-processamento (resize, normalização)
        ↓
Modelo de segmentação (U-Net)
        ↓
Máscara binária (área coberta vs. limpa)
        ↓
Cálculo do % de área coberta
        ↓
Classificação do status (ok / atenção / trocar)
        ↓
Output: máscara visual + % + status
```

## Estratégia de dataset

### Fase 1 — Dataset sintético (MVP)

Gerar imagens artificiais de armadilhas adesivas com diferentes níveis de
ocupação. Vantagem: as máscaras de segmentação vêm "de graça" (são geradas
junto com a imagem), eliminando o trabalho de rotulagem manual.

Abordagem sugerida:
- Base: imagem de placa adesiva limpa (fotografada ou gerada).
- Sobreposição: insetos recortados/gerados posicionados aleatoriamente sobre
  a placa, com variação de quantidade, posição, escala e rotação.
- Augmentation: variações de iluminação, ângulo, ruído de câmera.
- A máscara é gerada automaticamente a partir das posições dos insetos.

**Pendente:** foto real da armadilha da empresa parceira como referência visual
para calibrar a geração sintética.

### Fase 2 — Dados reais (refinamento)

Após contato com a empresa parceira:
- Coletar fotos reais de armadilhas em diferentes estágios de ocupação.
- Rotular semi-automaticamente usando o modelo da Fase 1 como ponto de partida
  + correção manual.
- Fine-tuning do modelo treinado com dados sintéticos usando os dados reais.
- Validar com dados reais exclusivamente (nunca usados no treino).

## Restrições

- Tecnologias: Python, PyTorch, U-Net (ver `.context/rules/ai_development_rules.md`)
- Integrantes do grupo: a definir
- Prazo de entrega: a definir
