# SECURITY SPEC — Projeto Integrador

**Versão:** 1.0
**Status:** ATIVO
**Escopo:** requisitos de segurança obrigatórios para o backend, frontend e
infraestrutura do projeto

---

# 1. Objetivo

Definir controles de segurança específicos para esta stack (FastAPI, React,
PostgreSQL, upload de imagem, modelo PyTorch, Docker) — não uma lista
genérica de boas práticas. Cada item aqui responde a um vetor de ataque real
para o que este sistema faz.

Este documento tem a mesma autoridade normativa que os demais em
`.context/rules/`. Uma alteração que toque entrada de usuário, upload,
autenticação, banco de dados ou dependências deve ser confrontada contra
este documento antes de ser aprovada.

---

# 2. Modelo de ameaça e escopo

## 2.1 Escopo do MVP

O MVP roda **localmente ou em rede fechada, para demonstração** — não é
implantado publicamente na internet. Isso não reduz a exigência dos
controles desta seção; reduz apenas a urgência de alguns itens marcados
explicitamente como "Fase 2" na seção 9.

## 2.2 Superfícies de ataque relevantes

| Superfície | Por quê |
|---|---|
| Upload de imagem (`POST /api/analises`) | Único endpoint que recebe arquivo binário de origem externa |
| Campos de texto livre (`identificador`, `localizacao`, `modelo`) | Entram no banco e são exibidos no frontend |
| Carregamento do modelo (`MODEL_WEIGHTS_PATH`) | Arquivo binário desserializado na inicialização do backend |
| CORS | Define quem pode chamar a API a partir de um navegador |
| Variáveis de ambiente | Credenciais de banco, caminhos de sistema |

## 2.3 Fora de escopo nesta versão

Autenticação de usuário e autorização por papel — o MVP não tem contas de
usuário (`project_overview.md`, seção 5.2). A ausência de autenticação é um
**risco aceito e documentado**, não um esquecimento — ver seção 3.

---

# 3. Autenticação — risco aceito

O MVP não implementa autenticação. Todos os endpoints de escrita
(`POST`/`PATCH`/`DELETE`) estão abertos a qualquer cliente que alcance a
API.

**Controle obrigatório de compensação:** o backend **nunca** deve ser
exposto a uma rede não confiável (internet pública) sem autenticação
adicionada antes. `CORS_ORIGENS` restrito a origens conhecidas (seção 6) é
mitigação parcial para navegador, não para chamada direta à API.

Adicionar autenticação é pré-requisito obrigatório antes de qualquer
implantação além de demonstração controlada — registrado como pendência de
Fase 2 na seção 9, não implementado agora por decisão de escopo, não por
descuido.

---

# 4. Validação de upload de imagem

Aplica-se a `POST /api/analises` (`../plan/api/api_spec.md`, seção 6.1).

## 4.1 Validação de conteúdo, não apenas de extensão

| Controle | Detalhe |
|---|---|
| Verificar o tipo real do arquivo pelos primeiros bytes (magic bytes), não pela extensão do nome nem pelo `Content-Type` declarado pelo cliente | Um arquivo `.jpg` pode conter qualquer coisa — o cliente controla ambos os campos |
| Rejeitar qualquer tipo fora de JPEG/PNG após essa verificação | Conforme `api_spec.md`, seção 2.4 |
| Limite de tamanho de 10 MB aplicado **antes** de carregar o arquivo inteiro em memória | Streaming ou verificação de `Content-Length`, não `try/except` após o carregamento completo |

## 4.2 Proteção contra decompression bomb

Uma imagem pequena em disco pode se expandir para gigabytes em memória ao
ser decodificada (ex.: PNG com dimensões absurdas). Isso é uma negação de
serviço, não uma falha de tamanho de arquivo.

| Controle | Detalhe |
|---|---|
| Definir limite explícito de dimensões decodificadas (ex.: `PIL.Image.MAX_IMAGE_PIXELS`) antes de processar qualquer upload | Nunca desabilitar esse limite globalmente por conveniência |
| Rejeitar imagem cuja resolução decodificada exceda um teto razoável (ordem de dezenas de milhões de pixels) | Nenhuma foto de armadilha legítima chega perto disso |

## 4.3 Nome e caminho do arquivo

| Controle | Detalhe |
|---|---|
| Nunca usar o nome de arquivo enviado pelo cliente para compor o caminho de gravação | Abre caminho para path traversal (`../../etc/passwd`) |
| Gerar o nome do arquivo no servidor (ex.: UUID da análise) | O nome original do cliente, se guardado, fica apenas como metadado, nunca como caminho |
| Gravar exclusivamente dentro de `UPLOADS_DIR` (seção 6), validando que o caminho resultante não escapa do diretório | Checagem explícita, não apenas confiança na geração do nome |

---

# 5. Banco de dados

## 5.1 Injeção de SQL

| Controle | Detalhe |
|---|---|
| Toda consulta passa pelo ORM (SQLAlchemy) ou por parâmetros vinculados | Nunca concatenar ou formatar string para montar SQL a partir de entrada de usuário |
| Proibido SQL cru construído a partir de campo de texto livre (`identificador`, `localizacao`) | Mesmo para filtros ou ordenação dinâmica — usar a API de expressões do ORM |

Esta regra já está implícita no uso do SQLAlchemy definido em
`../plan/foundation/tech_stack.md`, mas é reforçada aqui como controle de
segurança explícito, não apenas escolha de produtividade.

## 5.2 Credenciais

| Controle | Detalhe |
|---|---|
| `DATABASE_URL` só via variável de ambiente (`../task/current_task_ambiente.md`, AMB03) | Nunca hardcoded, nunca em exemplo com credencial real |
| Usuário do banco usado pela aplicação não é o superusuário do Postgres | Princípio de menor privilégio, mesmo em ambiente de desenvolvimento |

---

# 6. CORS e configuração

| Controle | Detalhe |
|---|---|
| `CORS_ORIGENS` é lista explícita de origens, nunca `*` | Conforme `../plan/api/api_spec.md`, seção 2.6 |
| Variáveis sensíveis nunca aparecem em log, mensagem de erro ou resposta da API | Ver seção 8 |

---

# 7. Carregamento do modelo

Esta é uma superfície de ataque específica de aplicações que carregam
modelos PyTorch, frequentemente ignorada.

## 7.1 Desserialização seria

`torch.load` usa `pickle` por padrão, que pode executar código arbitrário se
o arquivo carregado for malicioso — desserializar um checkpoint não é uma
operação neutra.

| Controle | Detalhe |
|---|---|
| Carregar pesos com `weights_only=True` (ou equivalente da versão do PyTorch em uso) | Restringe a desserialização a tensores, impedindo execução de código arbitrário embutido no arquivo |
| `MODEL_WEIGHTS_PATH` aponta **sempre** para um caminho local controlado pelo time, nunca para um upload de usuário ou URL externa não verificada | O modelo é um artefato de build/deploy, não um dado de entrada da aplicação |

## 7.2 Falha de inicialização

Já especificado em `../plan/model/modelo_spec.md`, seção 8: pesos ausentes
ou inválidos impedem a aplicação de subir. Reforço de segurança: essa falha
**não deve** expor caminho interno de arquivo nem stack trace ao cliente —
apenas ao log do servidor.

---

# 8. Tratamento de erro e logs

| Controle | Detalhe |
|---|---|
| O corpo de erro retornado ao cliente (`api_spec.md`, seção 2.2) nunca inclui stack trace, caminho de arquivo do servidor, ou detalhe de configuração interna | `detalhes` no corpo de erro é para contexto do domínio (ex.: `armadilha_id`), não para depuração interna |
| Logs do servidor podem conter detalhe técnico completo | Log não é resposta pública |
| Nunca logar o conteúdo de `DATABASE_URL` ou outras variáveis sensíveis | Nem em nível de debug |

---

# 9. Infraestrutura e dependências

## 9.1 Docker

| Controle | Detalhe |
|---|---|
| Containers rodam com usuário não-root | Evita escalonamento de privilégio caso haja vulnerabilidade na aplicação |
| Imagens base mínimas (`-slim`/`-alpine` quando compatível) | Reduz superfície de vulnerabilidades herdadas da imagem |
| Nenhum segredo (senha, chave) copiado para dentro da imagem em tempo de build | Segredos entram só via variável de ambiente em tempo de execução |

## 9.2 Dependências

| Controle | Detalhe |
|---|---|
| Lockfile commitado (`uv.lock`, `package-lock.json`) | Build reprodutível, conforme `tech_stack.md`, seção 7 |
| Atualização de dependência é uma mudança revisada, não automática | Alinhado com a seção 4.2 de `ai_development_rules.md` — mudar dependência exige nova confirmação |

## 9.3 Pendente para além do MVP (Fase 2)

- [ ] Autenticação e autorização (seção 3)
- [ ] Rate limiting no endpoint de análise, se exposto além de demonstração
      controlada — inferência é operação computacionalmente cara e pode ser
      usada para negação de serviço
- [ ] HTTPS obrigatório em qualquer implantação além de localhost
- [ ] Rotina de verificação de vulnerabilidades conhecidas nas dependências
      (ex.: `pip-audit`, `npm audit`) antes de cada entrega relevante

---

# 10. Frontend

| Controle | Detalhe |
|---|---|
| Nunca usar `dangerouslySetInnerHTML` com dado vindo da API sem sanitização | Campos de texto livre (`identificador`, `localizacao`) são exibidos na interface e podem conter qualquer conteúdo |
| React escapa texto por padrão em renderização normal (`{variavel}`) | Suficiente na maioria dos casos — o risco é justamente contornar esse comportamento padrão sem necessidade |

---

# 11. Pendências

- [ ] Confirmar a biblioteca de detecção de tipo real de arquivo a usar
      (ex.: `python-magic`) e integrá-la em `current_task_api.md`, API07
- [ ] Definir usuário não-root nas imagens Docker de `current_task_infra.md`,
      INFRA03
- [ ] Revisar este documento antes de qualquer implantação além de
      demonstração local/controlada

---

# FIM DO SECURITY SPEC
