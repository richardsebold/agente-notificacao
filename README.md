# Agente de Comunicado de Liberação — Twygo

Gera automaticamente o comunicado de liberações para produção a partir de links do Artia, usando o Claude Code com Playwright.

## Pré-requisitos

- [Claude Code](https://claude.ai/code) instalado (`npm install -g @anthropic-ai/claude-code`)
- [Node.js](https://nodejs.org/) 18+ (para o Playwright MCP)
- Conta no Artia com acesso aos projetos

## Setup

**1. Clone o repositório**

```bash
git clone https://github.com/richardsebold/agente-notificacao.git
cd agente-notificacao
```

**2. Crie o arquivo `.env` com suas credenciais do Artia**

```bash
cp .env.example .env
```

Edite o `.env` e preencha com seu e-mail e senha do Artia:

```
ARTIA_EMAIL=seu-email@empresa.com
ARTIA_SENHA=sua-senha
```

**3. Abra o projeto no Claude Code**

```bash
claude
```

Na primeira vez, o Claude pode pedir permissão para usar o Playwright — confirme.

## Como usar

Cole no chat do Claude Code a mensagem de liberações que você recebe (com os links do Artia). Exemplo:

```
Boa tarde pessoal, segue algumas liberações para hoje:

Competências - Gerar versão para deploy da Twygo - 23/05/2026
https://app.artia.com/a/4874953/f/6257028/activities/32856832

Kit de marca - Gerar versão para deploy da Twygo - 23/05/2026
https://app2.artia.com/a/4874953/f/6414891/activities/32866954
```

O agente vai:
1. Abrir o Artia no navegador e fazer login automaticamente
2. Ler as atividades concluídas e validadas de cada projeto
3. Montar o comunicado formatado pronto para copiar e enviar

Os comunicados gerados são salvos na pasta `outputs/`.

## Estrutura do projeto

```
.
├── .claude/
│   ├── skills/
│   │   └── comunicado-liberacao-twygo/  # skill do agente
│   └── settings.local.json              # permissões do Claude Code
├── .mcp.json                            # configuração do Playwright MCP
├── .env                                 # suas credenciais (não commitado)
├── .env.example                         # modelo do .env
└── outputs/                             # comunicados gerados
```
