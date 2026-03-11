# Guia de Otimização Global do Claude Code

---

## 1. settings.json (~/.claude/settings.json)

Configurações globais que valem para TODOS os projetos.

### Permissões Amplas

Cada vez que o Claude para e pede permissão, gasta tokens na ida e volta.
Configurar permissões amplas para ferramentas do dia a dia e restringir só ações destrutivas.

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)",
      "Bash(git add*)",
      "Bash(git commit*)",
      "Bash(go build*)",
      "Bash(go test*)",
      "Bash(go run*)",
      "Bash(go vet*)",
      "Bash(go fmt*)",
      "Bash(go mod*)",
      "Bash(ls*)",
      "Bash(mkdir*)",
      "Bash(cat*)",
      "Bash(which*)",
      "Bash(echo*)",
      "Agent"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(git push --force*)",
      "Bash(git reset --hard*)",
      "Bash(drop table*)",
      "Bash(sudo*)"
    ]
  }
}
```

### Por que isso importa
- Cada interrupção de permissão = ~500-1000 tokens desperdiçados
- O fluxo do agente quebra, ele precisa re-analisar contexto após cada pausa
- Deny list protege contra ações destrutivas

---

## 2. settings.local.json (~/.claude/settings.local.json)

Preferências pessoais que NÃO vão para repositórios (gitignored).

```json
{
  "preferences": {
    "outputStyle": "concise",
    "language": "pt-BR"
  }
}
```

### Limpeza periódica
- O Claude Code acumula permissões one-off de sessões passadas
- Revisar periodicamente e remover regras de projetos antigos
- Manter só permissões genéricas reutilizáveis

---

## 3. Estratégia de Modelos para Subagentes

O Claude Code lança subagentes para tarefas paralelas. Usar modelos conforme complexidade:

| Modelo | Quando usar | Exemplos |
|--------|------------|----------|
| **Haiku** | Buscas triviais | grep, find file, glob |
| **Sonnet** | Análise e planejamento | code review, refactoring plan, exploração |
| **Opus** | Implementação complexa | contexto principal, arquitetura |

Salvar essa diretriz no memory do projeto para o agente seguir automaticamente.

---

## 4. CLAUDE.md Global (~/.claude/CLAUDE.md)

Instruções que valem para TODOS os projetos. Manter enxuto (<100 linhas).

Deve conter:
- Idioma preferido (pt-BR)
- Estilo de resposta (conciso, direto)
- Convenções gerais de código
- Seção "NÃO FAÇA" (tão importante quanto instruções positivas)
- Referência à estratégia de modelos

---

## 5. Estrutura de Projeto (por repositório)

Para cada projeto, a estrutura recomendada:

```
projeto/
├── CLAUDE.md              # Contexto e memória DO PROJETO (enxuto)
├── CLAUDE.local.md        # Config pessoal do projeto (gitignored)
├── .claude/
│   ├── settings.json
│   ├── rules/             # Regras por domínio
│   │   ├── coding.md
│   │   └── testing.md
│   ├── skills/            # Workflows reutilizáveis
│   │   ├── review/
│   │   │   └── SKILL.md
│   │   └── refactor/
│   │       └── SKILL.md
│   ├── output-styles/     # Estilos de resposta
│   └── hooks/             # Automações
│       ├── session_start/
│       └── post_tool_use/
├── docs/
│   ├── architecture.md
│   └── decisions/
└── .mcp.json              # Ferramentas externas
```

---

## 6. Dicas de Economia de Tokens

1. **CLAUDE.md enxuto** — o Claude consulta o código quando precisa de detalhes, não precisa carregar tudo no contexto
2. **Sessões curtas e focadas** — em vez de conversas longas, iniciar nova sessão por tarefa
3. **outputStyle: "concise"** — respostas mais curtas
4. **Permissões amplas** — menos interrupções = menos tokens na ida/volta
5. **Modelos certos para tarefas certas** — Haiku para buscas, Sonnet para análise, Opus para implementação

---

## 7. Seção "NÃO FAÇA" (essencial no CLAUDE.md)

Exemplos do que incluir:
- Não crie arquivos sem necessidade
- Não faça commits sem pedir
- Não use force push
- Não adicione dependências sem discutir
- Não altere configs de CI/CD sem aprovação
- Não ignore erros de compilação
- Não crie abstrações prematuras
