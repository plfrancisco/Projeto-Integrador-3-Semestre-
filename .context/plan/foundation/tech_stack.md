# Tech Stack

Decisões tecnológicas do projeto e justificativas.

## Visão geral

```
┌─────────────────────────────────────────────┐
│  Frontend (React + Vite + Tailwind + TS)    │
│  Upload de imagem → exibe resultado visual  │
└──────────────────┬──────────────────────────┘
                   │ REST API
┌──────────────────▼──────────────────────────┐
│  Backend (FastAPI + Python)                 │
│  Serve o modelo, processa imagem, retorna   │
│  máscara + % + status                      │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  Modelo de ML (PyTorch + U-Net)             │
│  Segmentação da armadilha                   │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  PostgreSQL (futuro)                        │
│  Histórico de análises, armadilhas, clientes│
└─────────────────────────────────────────────┘
```

## Backend

| Componente | Tecnologia | Justificativa |
|---|---|---|
| Linguagem | Python 3.11+ | Padrão de mercado para ML/CV |
| Framework web | FastAPI | Async, tipagem, docs automáticas (`/docs`), fácil de servir modelo via API REST. Mesma stack usável quando virar produto real |
| Servidor | Uvicorn | ASGI padrão para FastAPI |

## Frontend

| Componente | Tecnologia | Justificativa |
|---|---|---|
| Framework | React + TypeScript | Base sólida, 2+ pessoas no grupo já conhecem. Escalável pra PWA no futuro |
| Build tool | Vite | Rápido, zero config, HMR |
| Estilização | Tailwind CSS | Produtividade, sem escrever CSS manual |
| Componentes UI | (a definir, se necessário) | Para o MVP, HTML + Tailwind bastam. Radix/shadcn pode entrar depois se necessário |

> **Decisão:** frontend simples no MVP — sem TanStack Router/Query/Start, sem
> biblioteca de componentes pesada. O frontend faz upload de uma imagem, chama
> a API do FastAPI, e exibe o resultado (máscara + % + status). Se no futuro
> virar PWA/produto, adiciona roteamento e estado mais robusto.

## Modelo / ML

| Componente | Tecnologia | Justificativa |
|---|---|---|
| Framework de ML | PyTorch | Flexibilidade, comunidade ativa, padrão acadêmico |
| Biblioteca de segmentação | `segmentation_models_pytorch` (smp) | Fornece U-Net com encoder pré-treinado em poucas linhas. Trocar de arquitetura (U-Net → FPN → DeepLabV3) vira um parâmetro, permitindo comparação no relatório com esforço mínimo |
| Arquitetura | U-Net com encoder ResNet34 | Referência para segmentação semântica; ResNet34 equilibra capacidade e tamanho |
| Transfer learning | Encoder pré-treinado em ImageNet (`encoder_weights="imagenet"`) | **Decisão crítica** — o encoder já reconhece bordas, texturas e formas, o que viabiliza o treino com o dataset pequeno do projeto. Sem isso há risco real de não convergência |
| Training loop | PyTorch puro (sem Lightning) | Poucos experimentos previstos; escrever o loop dá controle e entendimento do processo, relevante para a defesa acadêmica. Lightning fica como opção futura se o treino virar repetitivo |
| Geração sintética | OpenCV + Pillow + Albumentations | Gerar imagens de armadilhas com máscaras automáticas |
| Augmentation | Albumentations | Leve, integra com PyTorch, augmenta imagem + máscara juntos |
| Métricas | torchmetrics | IoU/Dice, erro percentual, matriz de confusão |
| Baseline comparativo | OpenCV (threshold de cor) | Ver seção abaixo |

### Baseline clássico com OpenCV

Antes do modelo treinado, implementar uma solução clássica: a placa adesiva tem
cor conhecida e uniforme, os insetos destoam dela — contar os pixels que
destoam dá um percentual de área coberta em poucas linhas de OpenCV.

**Não é um substituto do modelo.** Serve como referência de comparação:

- Sem baseline, o resultado é *"a U-Net atingiu X% de IoU"* — o avaliador não
  tem parâmetro para julgar se X é bom.
- Com baseline, o resultado é *"o método clássico atingiu X% e degradou sob
  variação de iluminação; a U-Net atingiu Y% e manteve Z%"* — isso demonstra
  com evidência que o deep learning era necessário, em vez de apenas afirmar.

Benefício adicional: se o baseline funcionar bem demais, isso é descoberto no
início do projeto, não na véspera da entrega.

### Alternativas avaliadas e descartadas

| Alternativa | Motivo da rejeição |
|---|---|
| YOLOv8/v11-seg | Faz segmentação de **instâncias** (identifica cada inseto), contrariando a decisão de medir área em vez de contar. Além disso, licença AGPL-3.0 é um problema concreto dada a empresa parceira real |
| PyTorch Lightning | Abstração útil para muitos experimentos; com o volume previsto, o boilerplate manual é aceitável e traz mais entendimento |
| TensorFlow/Keras | Sem vantagem técnica no caso; ecossistema de segmentação menos rico que o do PyTorch (`smp`) |
| U-Net escrita manualmente | ~150 linhas para o mesmo resultado que `smp` entrega em 3, e sem pesos pré-treinados por padrão |

## Banco de dados

| Componente | Tecnologia | Justificativa |
|---|---|---|
| Banco | PostgreSQL | Robusto, gratuito, escala bem. No MVP pode não ser necessário (modelo roda stateless), mas quando persistir histórico de análises/clientes já está pronto |
| ORM | SQLAlchemy ou Tortoise (a definir) | Integra com FastAPI |

> **No MVP:** banco é opcional. O modelo recebe imagem e retorna resultado
> sem precisar persistir nada. Banco entra quando houver cadastro de
> armadilhas, histórico de análises, etc.

## Infra

| Componente | Tecnologia | Justificativa |
|---|---|---|
| Containerização | Docker + Docker Compose | Ambiente reproduzível, fácil de subir backend + frontend + banco juntos |
| Deploy (futuro) | A definir (Railway, Fly.io, VPS, ou cloud da faculdade) | |

## Ambiente de desenvolvimento

| Componente | Tecnologia |
|---|---|
| Gerenciador Python | uv (recomendado — rápido, substitui pip+venv) ou pip + venv |
| Gerenciador Node | npm ou pnpm |
| Linter/Formatter Python | Ruff (substitui flake8 + black + isort num só tool) |
| Linter/Formatter JS/TS | ESLint + Prettier |
| GPU (treino) | A definir (local, Google Colab, Kaggle) |
| Tracking de experimentos | A definir (W&B, MLflow, ou manual) |

## Estrutura de pastas do código (proposta)

```
backend/
├── src/
│   ├── api/            # rotas FastAPI (upload, resultado)
│   ├── inference/      # pipeline de inferência (imagem → status)
│   ├── models/         # definição da U-Net
│   └── config/         # configurações, constantes
├── training/           # scripts de treinamento (roda separado, não no server)
│   ├── train.py
│   ├── evaluate.py
│   └── generate_synthetic.py
├── requirements.txt
└── Dockerfile

frontend/
├── src/
│   ├── components/     # componentes React
│   ├── pages/          # telas (upload, resultado)
│   └── services/       # chamadas à API
├── package.json
└── Dockerfile

data/                   # NÃO versionado (.gitignore)
├── synthetic/
├── real/
└── masks/

models/                 # NÃO versionado (.gitignore)
└── checkpoints/

notebooks/              # exploração e prototipagem

docker-compose.yml
```

## Decisões pendentes

- [ ] Onde treinar (GPU local? Colab? Kaggle?)
- [ ] Tracking de experimentos (W&B, MLflow, ou manual?)
- [ ] ORM para PostgreSQL (SQLAlchemy vs Tortoise)
- [ ] Gerenciador Python (uv vs pip+venv)
- [ ] Deploy (Railway, Fly.io, VPS?)
