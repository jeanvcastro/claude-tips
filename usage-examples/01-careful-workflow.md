# Prompt: Modo Engenheiro Cuidadoso

## Contexto
Demonstra o workflow padrão: criar plano em arquivo local, executar task por task revisando cada diff, Claude atualiza o plano a cada conclusão.

O código Go de exemplo será gerado pelo próprio Claude no output/.

## Prompt 1 — Criar projeto e plano

```
Crie um projeto Go simples em output/01-workflow-basico/ com uma API HTTP (net/http puro):
- GET /health → {"status": "ok"}
- POST /users → cria user (id, name, email) em memória
- GET /users → lista users
- GET /users/{id} → busca user por id
- DELETE /users/{id} → remove user

Inicialize go mod e crie só o main.go básico funcional.

Depois, crie um plano em output/01-workflow-basico/.claude/task_plan.md com as seguintes melhorias a implementar:

1. Middleware de logging (method, path, status code, duração)
2. Middleware de recovery (captura panics, retorna 500)
3. Validação no POST /users (name obrigatório, email com formato válido)
4. Testes unitários para handlers
5. Testes para middlewares

NÃO execute as tasks ainda — só crie o código base e o plano.
```

## Prompt 2 — Executar task por task

```
Leia o plano em output/01-workflow-basico/.claude/task_plan.md.
Execute a próxima task pendente.
Após concluir, atualize o plano marcando como done e indique a próxima.
Aguarde meu review antes de seguir.
```

## Prompt 3 — Continuidade (antes de compactar)

```
O contexto está ficando grande. Me dê um prompt completo para eu continuar de uma nova sessão.
O prompt deve:
- Descrever o que já foi feito
- Apontar para o plano em output/01-workflow-basico/.claude/task_plan.md
- Indicar a próxima task a executar
- Incluir qualquer decisão ou contexto que seria perdido
```
