# Workflow com Claude Code

4 modos de trabalho dependendo do contexto.

---

## Modo 1: Engenheiro Cuidadoso (default)

Para código de produção. Controle humano total, qualidade garantida.

### Filosofia
- **Sem auto-accept** — revisar cada passo
- **Plano em arquivo local** — fonte de verdade do progresso
- **Claude atualiza o plano** a cada task concluída antes de seguir
- **Sessões curtas e focadas** — uma tarefa por sessão

### Fluxo

```
1. PLANEJAR  →  Claude cria plano em .claude/task_plan.md
2. EXECUTAR  →  Task por task, revisando cada diff
3. ATUALIZAR →  Claude marca task done no plano, segue pra próxima
4. CONTINUAR →  Antes de compactar: pedir prompt de continuidade + /clear
5. REVIEW    →  git diff final antes de abrir PR
```

### Diff lado a lado na IDE

Rodar o Claude Code pelo terminal integrado da IDE facilita o review de cada alteração com diff visual. Extensões úteis:

- **VS Code** — a extensão oficial [Claude Code for VS Code](https://marketplace.visualstudio.com/items?itemName=anthropics.claude-code) mostra diff side-by-side automaticamente a cada edição proposta, direto no editor. Dá pra aceitar/rejeitar cada mudança individualmente.
- **JetBrains (GoLand, IntelliJ, etc)** — o plugin [Claude Code for JetBrains](https://plugins.jetbrains.com/plugin/claude-code) oferece a mesma experiência de diff integrado.
- **Git lens (VS Code)** — complementar ao anterior, mostra blame inline e histórico de alterações por linha, útil pra entender o contexto do que está sendo modificado.

O diff visual é o que torna viável revisar cada passo sem perder velocidade. Em vez de ler o output no terminal, você vê exatamente o que mudou, onde mudou, com syntax highlighting.

### Continuidade entre sessões
Quando o contexto está ficando grande:
1. Pedir ao Claude: *"me dê um prompt para continuar de uma nova janela com contexto completo, lendo o plano atualizado"*
2. Receber prompt detalhado com estado atual
3. `/clear` e colar o prompt na nova sessão
4. Claude lê o plano e continua de onde parou

### Exemplo de plano (.claude/task_plan.md)
```markdown
# Plano: Adicionar middlewares na API

## Tasks
- [x] 1. Criar middleware de logging (method, path, status, duração)
- [x] 2. Criar middleware de recovery (captura panics, retorna 500)
- [ ] 3. Adicionar validação no POST /users
- [ ] 4. Testes unitários para handlers
- [ ] 5. Testes para middlewares

## Contexto
- API em Go puro (net/http)
- Sem framework, sem dependências externas
```

---

## Modo 2: Com Agentes

Para tarefas que não são codificação direta: review de PR, geração de diagramas, documentação. O agente faz o trabalho pesado, você valida o resultado.

### Agentes disponíveis

#### D2 Diagram Architect
Arquivo: `.claude/agents/d2-diagram-architect.md`

Cria diagramas de arquitetura profissionais usando D2 language. Paleta de cores consistente, ícones AWS, labels descritivos em todas as conexões.

Como invocar:
```
Use o agente d2-diagram-architect para criar um diagrama de [descrição]
```

#### Code Reviewer
Arquivo: `.claude/agents/code-reviewer.md`

Um agente que lança 4 subagentes em paralelo na branch atual:
- **Performance** — complexidade, alocações, goroutine leaks, N+1
- **Segurança** — injection, auth bypass, secrets, CVEs
- **Infraestrutura** — graceful shutdown, timeouts, circuit breakers, observabilidade
- **Qualidade** — convenções da linguagem, testes, estrutura, error handling

Consolida num relatório classificado por severidade (CRITICAL/WARNING/SUGGESTION).
Roda como review automatizado antes de abrir PR.

Como invocar:
```
Use o agente code-reviewer para fazer review das mudanças na branch atual
```

### Criando seus próprios agentes
Qualquer `.md` em `.claude/agents/` vira um agente invocável. Estrutura:
```yaml
---
name: nome-do-agente
description: "Quando usar este agente"
model: opus  # ou sonnet, haiku
---
Instruções do agente aqui.
```

---

## Modo 3: Browser Copilot (--chrome)

Para tarefas que não são código: ler tickets no Jira, analisar logs no Grafana/Datadog, investigar dashboards, cruzar informações entre ferramentas. O Claude vira um copiloto de browser que lê, interpreta e age sobre o que está na tela.

### Setup
1. Instalar a extensão [Claude Code for Chrome](https://chromewebstore.google.com/detail/claude-code/claudecode) no navegador
2. Iniciar o Claude Code com a flag `--chrome`:
```bash
claude --chrome
```
3. O Claude agora tem acesso às abas do Chrome para navegar, ler conteúdo e interagir com páginas

> Antes isso exigia configurar MCP chrome-devtools manualmente. Agora é nativo no Claude Code.

### Casos de uso

#### Jira → Plano de implementação
```
Abre o ticket PROJ-1234 no Jira, lê os requisitos, critérios de aceitação e comentários.
Cria um plano de implementação em .claude/task_plan.md baseado no que encontrar.
```

#### Grafana → Diagnóstico de performance
```
Abre o dashboard de performance no Grafana (aba já aberta).
Analisa os gráficos de latência e throughput das últimas 24h.
Identifica anomalias e sugere possíveis causas no código.
```

#### Datadog → Debug de produção
```
Lê os logs de erro no Datadog filtrando por service:payments nos últimos 30 min.
Identifica o padrão de erros, correlaciona com o código fonte e sugere fix.
```

#### Jira + Logs → Investigação completa
```
1. Lê o bug report no Jira (PROJ-5678)
2. Abre o Datadog e busca logs relacionados ao erro descrito
3. Cruza as informações e identifica a causa raiz no código
4. Cria o plano de correção em .claude/task_plan.md
```

### Fluxo típico
```
1. Abrir as ferramentas no Chrome (Jira, Grafana, Datadog, etc)
2. claude --chrome
3. Pedir para ler/analisar o que está na tela
4. Claude navega, lê, interpreta e propõe ações
5. Você valida e decide o próximo passo
```

### Dicas
- Deixar as abas já abertas nas páginas certas antes de iniciar — economiza tempo de navegação
- Funciona com qualquer ferramenta web: Confluence, Notion, Sentry, CloudWatch, etc
- Combina bem com o Modo 1: Claude lê o Jira, cria o plano, e você executa task por task

---

## Modo 4: Go-Horse com Ralph Loops

Para POCs, protótipos e código descartável. Loop autônomo, sem review humano detalhado.

### O que é Ralph
Um padrão de loop autônomo (github.com/snarktank/ralph) onde cada iteração roda com **contexto fresco**. A memória entre iterações persiste apenas por:
- Commits no Git
- `progress.txt` (aprendizados acumulados)
- `prd.json` (status das stories)

Erros de uma iteração não contaminam a próxima.

### Fluxo
1. Escrever PRD com user stories pequenas e testáveis
2. Converter para `prd.json`
3. Ralph pega a story de maior prioridade com `passes: false`
4. Implementa, roda testes
5. Se passa: commit, marca `passes: true`, próxima
6. Se falha: registra aprendizado, tenta de novo
7. Repete até todas passarem

### Quando usar
- POCs e protótipos rápidos
- Exploração técnica
- Features pequenas e bem definidas
- Código que vai ser jogado fora

### Quando NÃO usar
- Código de produção
- Mudanças arquiteturais
- Qualquer coisa com segurança ou dados sensíveis

### Regras para stories funcionarem
- Completável em UMA janela de contexto
- Critérios de aceitação claros
- Feedback loop automático (testes, typecheck)

Story boa: "Adicionar endpoint GET /health que retorna 200"
Story ruim: "Construir sistema de autenticação completo"

### Executar
```bash
./scripts/ralph/ralph.sh --tool claude [max_iterations]
```

---

## Resumo

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
