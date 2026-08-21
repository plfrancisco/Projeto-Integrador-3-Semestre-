# Projeto Integrador — Visão Computacional para Armadilhas Adesivas

Sistema de visão computacional que analisa imagens de armadilhas adesivas de mosquito e estima automaticamente o nível de saturação (% de área coberta), indicando o momento ideal de troca.

Organizado com abordagem *spec-driven development*: a documentação de especificação vive em [`.context/`](.context/) e orienta o que será implementado.

## Estrutura

```
.context/
├── plan/                          ← especificações e planejamento
│   ├── README.md                  ← índice com ordem de leitura
│   ├── foundation/                ← visão geral, arquitetura e diretrizes
│   │   ├── project_overview.md
│   │   └── proposta_individual.md
│   └── data/                      ← modelo de dados
│       └── data_model.md
├── rules/
│   └── ai_development_rules.md    ← regras de desenvolvimento com IA
├── task/
│   ├── README.md                  ← índice de tasks (ativas e históricas)
│   └── completed/                 ← tasks concluídas
└── validation/                    ← relatórios de validação/homologação
```

O código-fonte da implementação será adicionado posteriormente, guiado pelo conteúdo dessas especificações.
