# Projeto Integrador — Visão Computacional para Armadilhas Adesivas

Sistema de visão computacional que analisa imagens de armadilhas adesivas de mosquito e estima automaticamente o nível de saturação (% de área coberta), indicando o momento ideal de troca.

Organizado com abordagem *spec-driven development*: a documentação de especificação vive em [`.context/`](.context/) e orienta o que será implementado.

## Estrutura

```
.context/
├── plan/                             ← especificações e planejamento
│   ├── README.md                     ← índice com ordem de leitura
│   ├── foundation/                   ← visão geral, arquitetura e diretrizes
│   │   ├── project_overview.md
│   │   ├── proposta_individual.md
│   │   ├── tech_stack.md
│   │   ├── hardware_armadilha.md
│   │   └── stakeholder_evidencias.md
│   ├── data/                         ← banco de dados e datasets
│   │   ├── data_model.md
│   │   ├── seed_demo.md
│   │   └── dataset_sintetico.md
│   ├── model/                        ← modelo de visão computacional
│   │   ├── modelo_spec.md
│   │   └── avaliacao_spec.md
│   ├── frontend/                     ← telas da interface
│   │   └── telas_spec.md
│   └── api/                          ← contrato REST
│       └── api_spec.md
├── rules/
│   ├── ai_development_rules.md       ← regras de desenvolvimento com IA
│   └── security_spec.md              ← requisitos de segurança
├── task/                             ← quebra do trabalho por especialidade
│   ├── README.md                     ← índice de tasks (ativas e históricas)
│   ├── current_task_ambiente.md
│   ├── current_task_infra.md
│   ├── current_task_banco.md
│   ├── current_task_api.md
│   ├── current_task_frontend.md
│   ├── current_task_modelo.md
│   ├── current_task_seed_demo.md
│   ├── current_task_externo.md
│   └── completed/                    ← tasks concluídas
└── validation/                       ← relatórios de validação/homologação
```

O código-fonte da implementação será adicionado posteriormente, guiado pelo conteúdo dessas especificações.
