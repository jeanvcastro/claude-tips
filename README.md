# Claude Code Tips

Guia prático e replicável de como tirar o máximo do Claude Code como ferramenta de desenvolvimento.

Este repositório documenta workflows, configurações globais, agentes especializados e exemplos de uso prontos para rodar. A ideia é que qualquer pessoa possa clonar, aplicar as configs e rodar os exemplos — cada um terá resultados diferentes, porque o output não é versionado.

---

## Estrutura do Repositório

```
claude-tips/
├── README.md                          # Este arquivo
├── .gitignore                         # Ignora output/
│
├── workflow-guide.md                  # Os 3 modos de trabalho detalhados
├── global-config-guide.md             # Configurações globais para performance
│
├── .claude/
│   └── agents/                        # Agentes prontos para copiar em qualquer projeto
│       ├── d2-diagram-architect.md    # Gera diagramas D2 de arquitetura
│       └── code-reviewer.md           # Review multi-disciplinar antes da PR
│
├── usage-examples/                    # Exemplos de uso de cada workflow
│   ├── 01-careful-workflow.md         # Engenheiro cuidadoso: plano + task-by-task
│   ├── 02-d2-diagram-agent.md        # Agente: diagrama de arquitetura
│   ├── 03-code-review-agent.md       # Agente: review com 4 subagentes
│   ├── 04-ralph-loop-poc.md          # Go-horse: POC autônoma com Ralph loop
│   └── 05-browser-copilot.md         # Browser: Jira, Grafana, Datadog
│
└── output/                            # Resultados dos exemplos (NÃO versionado)
    ├── 01-careful-workflow/
    ├── 02-d2-diagram-agent/
    ├── 03-code-review-agent/
    ├── 04-ralph-loop-poc/
    └── 05-browser-copilot/
```

---

## Os 4 Modos de Trabalho

O Claude Code é poderoso, mas não é auto-pilot. O melhor resultado vem de escolher o modo certo para o contexto.

### Modo 1: Engenheiro Cuidadoso (default)

Para código de produção. Controle humano total.

- **Sem auto-accept** — revisar cada diff antes de aceitar
- **Diff lado a lado na IDE** — usar as extensões do Claude Code para VS Code ou JetBrains que mostram diff visual a cada edição, permitindo aceitar/rejeitar mudanças individualmente
- **Plano em arquivo local** (`.claude/task_plan.md`) — fonte de verdade do progresso
- **Claude atualiza o plano** a cada task concluída antes de seguir para a próxima
- **Sessões curtas** — antes do contexto estourar, pedir prompt de continuidade, `/clear`, e colar na nova sessão
- O plano sempre reflete o estado real do progresso

Fluxo:
```
PLANEJAR → EXECUTAR (task por task) → ATUALIZAR plano → CONTINUAR (nova sessão) → REVIEW → PR
```

Exemplo de uso: `usage-examples/01-careful-workflow.md`

### Modo 2: Com Agentes

Para tarefas que não são codificação direta: review de PR, geração de diagramas, documentação técnica. O agente faz o trabalho pesado, você valida o resultado.

**Agentes incluídos:**

| Agente | O que faz |
|--------|-----------|
| `d2-diagram-architect` | Cria diagramas D2 profissionais com paleta de cores, ícones AWS e labels descritivos |
| `code-reviewer` | Lança 4 subagentes em paralelo (performance, segurança, infraestrutura, qualidade) e consolida num relatório classificado por severidade |

Os agentes vivem em `.claude/agents/` e podem ser copiados para qualquer projeto. Qualquer `.md` nessa pasta vira um agente invocável pelo Claude Code.

Exemplos de uso: `usage-examples/02-d2-diagram-agent.md` e `usage-examples/03-code-review-agent.md`

### Modo 3: Browser Copilot (--chrome)

Para tarefas operacionais que não são código: ler tickets no Jira, analisar logs no Grafana/Datadog, investigar dashboards, cruzar informações entre ferramentas web.

Requer a extensão [Claude Code for Chrome](https://chromewebstore.google.com/detail/claude-code/claudecode) instalada no navegador. Ao iniciar com `claude --chrome`, o Claude ganha acesso às abas do Chrome para navegar, ler e interagir com qualquer ferramenta web.

Funciona com: Jira, Grafana, Datadog, Sentry, Confluence, Notion, CloudWatch, etc.

Combina bem com o Modo 1: Claude lê o ticket no Jira, cria o plano, e você executa task por task.

Exemplo de uso: `usage-examples/05-browser-copilot.md`

### Modo 4: Go-Horse com Ralph Loops

Para POCs, protótipos e código descartável. Loop autônomo sem review humano.

Usa o [Ralph](https://github.com/snarktank/ralph) — um padrão de loop onde cada iteração roda com **contexto fresco**. A memória entre iterações persiste apenas por commits no Git, `progress.txt` e `prd.json`. Erros de uma iteração não contaminam a próxima.

Fluxo:
```
PRD com stories pequenas → prd.json → Ralph loop → implementa + testa → commit → próxima story → repete
```

Quando usar: POCs, exploração técnica, código descartável.
Quando NÃO usar: produção, mudanças arquiteturais, segurança.

Exemplo de uso: `usage-examples/04-ralph-loop-poc.md`

---

## Configurações Globais

Antes de usar os exemplos, aplique as configurações globais documentadas em `global-config-guide.md`. Resumo:

- **`~/.claude/settings.json`** — permissões amplas para ferramentas do dia a dia, deny list para ações destrutivas
- **`~/.claude/CLAUDE.md`** — instruções globais: idioma, estilo conciso, estratégia de modelos, seção "NÃO FAÇA"
- **`~/.claude/settings.local.json`** — limpar periodicamente (acumula permissões de sessões antigas)
- **Estratégia de modelos** — Haiku para buscas triviais, Sonnet para análise, Opus para implementação complexa

---

## Como Usar

### 1. Aplicar configs globais
```bash
# Ler o guia e aplicar
cat global-config-guide.md
```
Copie os JSONs para `~/.claude/settings.json` e crie o `~/.claude/CLAUDE.md`.

### 2. Copiar agentes para seu projeto
```bash
cp -r .claude/agents/ /seu/projeto/.claude/agents/
```

### 3. Escolher um exemplo e rodar
Cada arquivo em `usage-examples/` é independente e tem os prompts prontos para colar no Claude Code. Abra o arquivo, leia o contexto e cole os prompts na ordem.

### 4. Resultados em output/
A pasta `output/` é gitignored. Cada pessoa que rodar os exemplos terá resultados diferentes — esse é o ponto. Mesmo prompt, mesmo código base, soluções distintas.

---

## Resumo Rápido

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  PRODUÇÃO    →  Modo 1: Engenheiro cuidadoso         │
│                 Plano + task-by-task + review humano  │
│                                                      │
│  SUPORTE     →  Modo 2: Agentes                      │
│                 Review de PR, diagramas, docs         │
│                                                      │
│  OPERACIONAL →  Modo 3: Browser copilot (--chrome)   │
│                 Jira, Grafana, Datadog, Sentry        │
│                                                      │
│  DESCARTÁVEL →  Modo 4: Ralph loops                  │
│                 POCs, protótipos, código experimental │
│                                                      │
└──────────────────────────────────────────────────────┘
```
