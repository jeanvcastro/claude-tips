# Prompt: Ralph Loop para POC

## Contexto
Demonstra o modo go-horse: loop autônomo com contexto fresco a cada iteração.
Cria uma CLI em Go de forma totalmente autônoma usando Ralph.

## Pré-requisito
- Ralph instalado: https://github.com/snarktank/ralph

## Passo 1 — Setup do projeto

```bash
mkdir -p output/04-ralph-loop
cd output/04-ralph-loop
git init

# Baixa os 3 arquivos do Ralph
mkdir -p scripts/ralph
curl -sO --output-dir scripts/ralph https://raw.githubusercontent.com/snarktank/ralph/main/ralph.sh
curl -sO --output-dir scripts/ralph https://raw.githubusercontent.com/snarktank/ralph/main/CLAUDE.md
curl -sO --output-dir scripts/ralph https://raw.githubusercontent.com/snarktank/ralph/main/prompt.md
chmod +x scripts/ralph/ralph.sh
```

## Passo 2 — Setup do container descartável

O Ralph precisa de autonomia total — escrever arquivos, rodar build/test, commitar. Para isso usa `--dangerously-skip-permissions`, que pula **todas** as verificações de permissão do Claude Code.

**Rode sempre dentro de um container.** Assim nenhum arquivo da sua máquina é afetado. Se algo der errado, descarta o container e recomeça.

### Opção recomendada: Dev Container oficial da Anthropic

A Anthropic mantém um [devcontainer de referência](https://github.com/anthropics/claude-code/tree/main/.devcontainer) com firewall restritivo (whitelist de domínios), isolamento de rede e Claude Code pré-instalado. É a forma recomendada de rodar `--dangerously-skip-permissions` com segurança.

O setup consiste em 3 arquivos:
- **devcontainer.json** — config do container, extensões VS Code, volumes para persistir histórico e config do Claude
- **Dockerfile** — imagem Node.js 20 com git, zsh, fzf, gh, Claude Code já instalado
- **init-firewall.sh** — firewall que só permite acesso a npm, GitHub, API da Anthropic e bloqueia todo o resto

Baixe o Dockerfile e o firewall, e crie o devcontainer.json manualmente (precisa customizar):

```bash
mkdir -p .devcontainer
curl -sO --output-dir .devcontainer https://raw.githubusercontent.com/anthropics/claude-code/main/.devcontainer/Dockerfile
curl -sO --output-dir .devcontainer https://raw.githubusercontent.com/anthropics/claude-code/main/.devcontainer/init-firewall.sh

# Fix: DNS retorna IPs duplicados e ipset add falha (bug no script oficial).
sed -i 's/ipset add/ipset -exist add/g' .devcontainer/init-firewall.sh
```

Crie `.devcontainer/devcontainer.json`:

```jsonc
{
  "name": "Ralph Loop Sandbox",
  "build": {
    "dockerfile": "Dockerfile",
    "args": {
      "TZ": "${localEnv:TZ:America/Sao_Paulo}",
      "CLAUDE_CODE_VERSION": "latest"
    }
  },
  "runArgs": ["--cap-add=NET_ADMIN", "--cap-add=NET_RAW"],
  "remoteUser": "node",
  "mounts": [
    "source=ralph-bashhistory-${devcontainerId},target=/commandhistory,type=volume",
    "source=ralph-claude-config-${devcontainerId},target=/home/node/.claude,type=volume"
  ],
  "containerEnv": {
    "NODE_OPTIONS": "--max-old-space-size=4096",
    "CLAUDE_CONFIG_DIR": "/home/node/.claude"
  },
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind,consistency=delegated",
  "workspaceFolder": "/workspace",
  "postStartCommand": "sudo /usr/local/bin/init-firewall.sh",
  "waitFor": "postStartCommand"
}
```

Se precisar de Go, adicione no Dockerfile:

```dockerfile
# Após as instalações base, adicione Go
RUN wget -qO- https://go.dev/dl/go1.22.0.linux-amd64.tar.gz | tar -C /usr/local -xzf -
ENV PATH=$PATH:/usr/local/go/bin
```

Abra no VS Code com a extensão [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) → "Reopen in Container". Autentique com `claude login`.

> **Segurança:** o firewall bloqueia acesso a domínios não-autorizados, mas não impede exfiltração de credenciais do Claude Code para domínios permitidos. Use apenas com repositórios confiáveis.

### Opção simples: Docker direto

Se não quiser o setup completo, um container simples funciona — mas sem o firewall restritivo:

```bash
docker run --rm -it -v $(pwd):/app -w /app golang:1.22 bash

# Dentro do container
npm install -g @anthropic-ai/claude-code
claude login
```

## Passo 3 — Gerar PRD (dentro do container)

Cole no Claude Code:

```
Crie um PRD em prd.md para uma CLI Go que converte Markdown para HTML:

User stories (cada uma pequena e testável):
1. Setup: go mod init, main que lê path do .md como argumento CLI, exit 1 se arquivo não existe
2. Parser básico: converter headings, paragraphs, bold, italic, links para HTML
3. Code blocks: detectar blocos com linguagem (```go, ```js) e aplicar classes CSS
4. Output: flag --output/-o para arquivo, sem flag vai pra stdout, CSS inline
5. Template HTML5: head, meta charset, title, tema dark para code blocks, responsivo

Cada story deve ter critérios de aceitação claros e um teste automatizado.
```

## Passo 4 — Converter para prd.json

Cole no Claude Code:

```
Converta prd.md para prd.json no formato Ralph:

{
  "branchName": "poc-md2html",
  "userStories": [
    {
      "id": "US-001",
      "title": "...",
      "acceptanceCriteria": ["..."],
      "passes": false,
      "priority": 1
    }
  ]
}
```

## Passo 5 — Commitar e rodar o loop

```bash
git add -A && git commit -m "initial prd"
./scripts/ralph/ralph.sh --tool claude 10
```

## O que vai acontecer
1. Ralph cria branch `poc-md2html`
2. Pega US-001 (maior prioridade, passes: false)
3. Implementa, roda `go build` + `go test`
4. Passa → commit automático, marca passes: true, pega US-002
5. Falha → registra em progress.txt, tenta de novo com contexto fresco
6. Todas passes: true → loop encerra

## Resultado esperado
- Código Go funcional da CLI
- prd.json com todas as stories passes: true
- progress.txt com aprendizados
- Git log com commits incrementais por story
