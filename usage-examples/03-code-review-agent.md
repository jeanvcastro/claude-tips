# Prompt: Agente Code Reviewer

## Contexto
Demonstra o review multi-disciplinar antes de abrir PR.
O agente go-code-reviewer lança 4 subagentes em paralelo (performance, segurança, infraestrutura, qualidade) e consolida num relatório.

## Pré-requisito
- Ter rodado o cenário 01 com as mudanças commitadas numa branch
- Ter uma branch main/master como base de comparação

## Setup

```bash
cd output/01-workflow-basico
git init && git add -A && git commit -m "initial"
git checkout -b feature/add-middlewares
# (as mudanças do cenário 01 já estarão aqui)
```

## Prompt

```
Use o agente go-code-reviewer para fazer review completo das mudanças na branch atual comparando com main.

Rode os 4 subagentes em paralelo:
- Performance: complexidade, alocações, goroutine leaks, N+1
- Segurança: injection, auth bypass, secrets, input validation
- Infraestrutura: graceful shutdown, timeouts, circuit breakers, observabilidade
- Qualidade: convenções Go, presença de testes, error handling, estrutura

Consolide tudo num relatório único classificado por severidade.
Salve em output/03-agente-review/review-report.md
```

## O que esperar
A API do cenário 01 propositalmente tem gaps para o reviewer encontrar:
- Sem graceful shutdown
- Sem timeouts no HTTP server
- Health check básico demais
- Possível path traversal no /users/{id}
- Config hardcoded (porta 8080)
- Sem rate limiting
- Handlers no main.go (sem separação de pacotes)
